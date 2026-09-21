# =============================================================================
# Base Image: Clean CUDA 12.8 Development Environment (Ubuntu 24.04)
# =============================================================================
FROM nvidia/cuda:12.8.0-devel-ubuntu24.04

# Multi-stage copy for uv & uvx (fastest, cleanest official binary method)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /usr/local/bin/

ENV DEBIAN_FRONTEND=noninteractive \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    HOSTNAME=develop \
    SHELL=/bin/zsh \
    UV_INDEX_URL=https://mirrors.aliyun.com/pypi/simple

# =============================================================================
# System Dependencies & Tools
# =============================================================================
RUN if [ -f /etc/apt/sources.list.d/ubuntu.sources ]; then \
        sed -i 's|http://archive.ubuntu.com/ubuntu/|https://mirrors.aliyun.com/ubuntu/|g' /etc/apt/sources.list.d/ubuntu.sources && \
        sed -i 's|http://security.ubuntu.com/ubuntu/|https://mirrors.aliyun.com/ubuntu/|g' /etc/apt/sources.list.d/ubuntu.sources; \
    elif [ -f /etc/apt/sources.list ]; then \
        sed -i 's|http://archive.ubuntu.com/ubuntu/|https://mirrors.aliyun.com/ubuntu/|g' /etc/apt/sources.list && \
        sed -i 's|http://security.ubuntu.com/ubuntu/|https://mirrors.aliyun.com/ubuntu/|g' /etc/apt/sources.list; \
    fi \
    && (mkdir -p -m 755 /etc/apt/keyrings \
        && curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg -o /etc/apt/keyrings/githubcli-archive-keyring.gpg \
        && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
        && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" > /etc/apt/sources.list.d/github-cli.list \
        || true) \
    && apt-get update -y && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        cmake \
        curl \
        gh \
        git \
        git-lfs \
        python3 \
        python3-dev \
        python3-pip \
        python-is-python3 \
        sudo \
        wget \
        zsh \
    # ─── Remove externally-managed restriction for system pip ────────────────
    && rm -rf /usr/lib/python3*/EXTERNALLY-MANAGED 2>/dev/null || true \
    # ─── Install Node.js & Set China Mirror ──────────────────────────────────
    && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && npm config set registry https://registry.npmmirror.com --global \
    # ─── Install Neovim (Latest Official Release) ────────────────────────────
    && curl -fsSL https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.tar.gz \
        | tar -xz --strip-components=1 -C /usr/local \
    # ─── Install eza (Modern ls Replacement) ─────────────────────────────────
    && curl -fsSL https://github.com/eza-community/eza/releases/latest/download/eza_x86_64-unknown-linux-musl.tar.gz \
        | tar -xz -C /usr/local/bin \
    && chmod +x /usr/local/bin/eza \
    # ─── Install Starship Prompt ─────────────────────────────────────────────
    && curl -fsSL https://starship.rs/install.sh | sh -s -- -y \
    # ─── Build & Install tatr (Task Tracker) ──────────────────────────────────
    && git clone --depth 1 https://github.com/tsoding/tatr.git /tmp/tatr \
    && cd /tmp/tatr \
    && cc -o nob nob.c \
    && ./nob \
    && cp build/tatr /usr/local/bin/tatr \
    && chmod +x /usr/local/bin/tatr \
    && rm -rf /tmp/tatr \
    # ─── Clean apt cache ─────────────────────────────────────────────────────
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# =============================================================================
# Global Mirrors & Environment Configuration
# =============================================================================
RUN mkdir -p /etc/uv \
    && printf '[[index]]\nurl = "https://mirrors.aliyun.com/pypi/simple"\ndefault = true\n' > /etc/uv/uv.toml \
    && mkdir -p /etc/pip \
    && printf '[global]\nindex-url = https://mirrors.aliyun.com/pypi/simple\ntrusted-host = mirrors.aliyun.com\n' > /etc/pip.conf

# =============================================================================
# User & Hostname Setup
# =============================================================================
ARG USERNAME=user
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN echo "develop" > /etc/hostname \
    && existing_user=$(getent passwd $USER_UID | cut -d: -f1); \
    if [ -n "$existing_user" ]; then \
        home_dir=$(getent passwd $USER_UID | cut -d: -f6); \
        userdel "$existing_user"; \
        rm -rf "$home_dir"; \
    fi \
    && existing_group=$(getent group $USER_GID | cut -d: -f1); \
    if [ -n "$existing_group" ]; then \
        groupdel "$existing_group"; \
    fi \
    && groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m -s /bin/zsh $USERNAME \
    && echo "$USERNAME ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# =============================================================================
# User Developer Environment (Zsh, Starship, Neovim, Themes & Plugins)
# =============================================================================
USER $USERNAME
WORKDIR /home/$USERNAME

RUN mkdir -p ~/.config/uv ~/.config/nvim ~/.config/pip ~/.npm \
    # ─── User level mirrors ──────────────────────────────────────────────────
    && printf '[[index]]\nurl = "https://mirrors.aliyun.com/pypi/simple"\ndefault = true\n' > ~/.config/uv/uv.toml \
    && printf '[global]\nindex-url = https://mirrors.aliyun.com/pypi/simple\ntrusted-host = mirrors.aliyun.com\n' > ~/.config/pip/pip.conf \
    && echo "registry=https://registry.npmmirror.com" > ~/.npmrc \
    # ─── Oh My Zsh & Recommended Plugins ────────────────────────────────────
    && sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)" "" --unattended \
    && git clone --depth=1 https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions \
    && git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
    && git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting \
    # ─── Configure Starship with Catppuccin Powerline Preset ─────────────────
    && (starship preset catppuccin-powerline -o ~/.config/starship.toml 2>/dev/null \
        || curl -fsSL https://raw.githubusercontent.com/starship/starship/master/docs/public/presets/toml/catppuccin-powerline.toml -o ~/.config/starship.toml) \
    # ─── Configure .zshrc ────────────────────────────────────────────────────
    && sed -i 's/plugins=(git)/plugins=(git sudo gh zsh-autosuggestions zsh-syntax-highlighting zsh-completions)/g' ~/.zshrc \
    && echo '' >> ~/.zshrc \
    && echo '# Modern CLI aliases' >> ~/.zshrc \
    && echo 'alias ls="eza"' >> ~/.zshrc \
    && echo 'alias ll="eza -lh --icons=auto --git"' >> ~/.zshrc \
    && echo 'alias la="eza -lha --icons=auto --git"' >> ~/.zshrc \
    && echo 'alias lt="eza --tree --level=2 --icons=auto"' >> ~/.zshrc \
    && echo 'setopt no_nomatch # disable wildcard error if no match' >> ~/.zshrc \
    && echo 'eval "$(starship init zsh)"' >> ~/.zshrc \
    # ─── Neovim Catppuccin Theme & Configuration ─────────────────────────────
    && git clone --depth=1 https://github.com/catppuccin/nvim.git ~/.local/share/nvim/site/pack/themes/start/catppuccin \
    && printf '%s\n' \
        '-- Catppuccin Theme Configuration' \
        'vim.cmd.colorscheme "catppuccin-mocha"' \
        '' \
        '-- Basic Editor Options' \
        'vim.g.mapleader = " "' \
        'vim.opt.number = true' \
        'vim.opt.relativenumber = true' \
        'vim.opt.cursorline = true' \
        'vim.opt.termguicolors = true' \
        'vim.opt.tabstop = 4' \
        'vim.opt.shiftwidth = 4' \
        'vim.opt.expandtab = true' \
        'vim.opt.autoindent = true' \
        'vim.opt.smartindent = true' \
        'vim.opt.ignorecase = true' \
        'vim.opt.smartcase = true' \
        'vim.opt.wrap = false' \
        'vim.opt.scrolloff = 8' \
        'vim.opt.signcolumn = "yes"' \
        'vim.opt.mouse = "a"' \
        'vim.opt.clipboard = "unnamedplus"' > ~/.config/nvim/init.lua \
    # ─── Clean user cache ────────────────────────────────────────────────────
    && rm -rf ~/.cache

# =============================================================================
# Final Stage Cleanup & Entrypoint
# =============================================================================
USER root
RUN rm -rf /root/.cache /tmp/* /var/tmp/*

USER $USERNAME
WORKDIR /home/$USERNAME
CMD ["/bin/zsh"]
