# syntax=docker/dockerfile:1
# =============================================================================
# Production image for https://github.com/pulp/pulp-ui
#
# Usage: copy Dockerfile, nginx.conf.template and .dockerignore into the root
# of a pulp-ui checkout, then build from there:
#
#   docker build --build-arg PULP_UI_COMMIT=$(git rev-parse --short HEAD) -t pulp-ui .
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
# Stage 1: build the production bundle into dist/
# -----------------------------------------------------------------------------
FROM node:24-alpine AS build
WORKDIR /pulp-ui

# install dependencies first so the layer stays cached across source changes
COPY package.json package-lock.json ./
RUN npm ci

# PULP_UI_COMMIT spares the build from requiring git (it is shown in the UI's
# about dialog); pass --build-arg to embed the real commit
COPY . .
ARG PULP_UI_COMMIT=unknown
ENV PULP_UI_COMMIT=${PULP_UI_COMMIT}
RUN npm run build

# -----------------------------------------------------------------------------
# Stage 2: serve the bundle with nginx (non-root, unprivileged port)
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
