# =============================================================================
# Production image for https://github.com/pulp/pulp-ui
#
# Self-contained: fetches the pulp-ui source at a pinned release tag, builds
# the bundle and serves it with nginx. No external build context required.
#
#   docker build -t pulp-ui src/pulp-ui
#   # pin a different pulp-ui release (tag or commit sha):
#   docker build --build-arg PULP_UI_REF=v0.1.28 -t pulp-ui src/pulp-ui
#
# Ports:
#   - container listen port: env LISTEN_PORT (default 8002, must be >= 1024)
#   - host port:             docker -p <host>:<LISTEN_PORT> mapping
#   - upstream pulp API:     env PULP_API_URL (API paths are proxied to it)
#
#   docker run -d --name pulp-ui -p 8080:8002 \
#     -e PULP_API_URL=http://pulp:8080 pulp-ui
#
# Multi-arch: both base images publish linux/amd64 and linux/arm64, so this
# Dockerfile builds unchanged with `docker buildx --platform linux/amd64,linux/arm64`.
# =============================================================================

# -----------------------------------------------------------------------------
# Stage 1: fetch the pulp-ui source at a pinned release
# -----------------------------------------------------------------------------
FROM alpine:3 AS source
ARG PULP_UI_REF=v0.1.28
RUN apk add --no-cache git \
    && git clone --depth 1 --branch "${PULP_UI_REF}" https://github.com/pulp/pulp-ui.git /pulp-ui \
    && git -C /pulp-ui rev-parse HEAD > /pulp-ui/.ui-commit \
    && rm -rf /pulp-ui/.git

# -----------------------------------------------------------------------------
# Stage 2: build the production bundle into dist/
# -----------------------------------------------------------------------------
FROM node:24-alpine AS build
WORKDIR /pulp-ui

# install dependencies first so the layer stays cached across source changes
# (--legacy-peer-deps: v0.1.28 has a test-only peer conflict vitest@5.0.5 vs
#  @vitest/coverage-v8@5.0.1, irrelevant to the webpack production build)
COPY --from=source /pulp-ui/package.json /pulp-ui/package-lock.json ./
RUN npm ci --legacy-peer-deps

COPY --from=source /pulp-ui/ ./
# PULP_UI_COMMIT (from .ui-commit) spares the build from requiring git;
# it is shown in the UI's about dialog
RUN PULP_UI_COMMIT="$(cat .ui-commit)" npm run build

# -----------------------------------------------------------------------------
# Stage 3: serve the bundle with nginx (non-root, unprivileged port)
# -----------------------------------------------------------------------------
FROM nginxinc/nginx-unprivileged:1.28-alpine

LABEL org.opencontainers.image.title="pulp-ui" \
      org.opencontainers.image.source="https://github.com/pulp/pulp-ui"

COPY --from=build /pulp-ui/dist/ /usr/share/nginx/html/

# rendered by envsubst at container start (see nginx.conf.template)
COPY nginx.conf.template /etc/nginx/templates/default.conf.template

# LISTEN_PORT:  port nginx listens on inside the container (>= 1024, non-root)
# PULP_API_URL: pulp server that /api, /pulp, /v2, ... are proxied to
# EXPOSE documents the default; the real port is whatever LISTEN_PORT sets
ENV LISTEN_PORT=8002 \
    PULP_API_URL=http://127.0.0.1:8080

EXPOSE 8002
STOPSIGNAL SIGQUIT
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD wget -q --spider http://127.0.0.1:${LISTEN_PORT}/ || exit 1
