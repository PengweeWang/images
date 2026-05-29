# =============================================================================
# Base Image
# =============================================================================

FROM nvcr.io/nvidia/pytorch:25.10-py3
ENV DEBIAN_FRONTEND=noninteractive

# =============================================================================
# System Dependencies
# =============================================================================

RUN apt update -y && \
    apt install -y --no-install-recommends \
    vim \
    ffmpeg \
    wget \
    curl \
    build-essential \
    git \
    git-lfs \
    sudo \
    zsh \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# ─── Remove pip externally-managed restriction ────────────────────────────────

RUN rm -rf /usr/lib/python3.12/EXTERNALLY-MANAGED

# =============================================================================
# Python Packages
# =============================================================================

COPY requirements.txt /tmp/requirements.txt
RUN pip3 install -r /tmp/requirements.txt \
    && pip cache purge \
    && rm -f /tmp/requirements.txt


# =============================================================================
# User Setup
# =============================================================================

ARG USERNAME=rose
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# ─── Create user ─────────────────────────────────────────────────────────────

RUN existing_user=$(getent passwd $USER_UID | cut -d: -f1); \
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
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# ─── Set default shell to Zsh ────────────────────────────────────────────────

RUN sudo chsh -s /bin/zsh $USERNAME

# =============================================================================
# Developer Experience (ohmyzsh + vimrc)
# =============================================================================

USER $USERNAME
RUN sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)" "" --unattended \
    && git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions \
    && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
    && git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting \
    && sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="ys"/g' ~/.zshrc \
    && sed -i 's/plugins=(git)/plugins=(git zsh-syntax-highlighting zsh-autosuggestions zsh-completions)/g' ~/.zshrc \
    && git clone --depth=1 https://github.com/amix/vimrc.git ~/.vim_runtime \
    && sh ~/.vim_runtime/install_awesome_vimrc.sh \
    && echo 'set mouse-=a' >> ~/.vim_runtime/my_configs.vim \
    && echo '' >> ~/.zshrc \
    && echo 'setopt no_nomatch # disable * match' >> ~/.zshrc\
    && echo '' >> ~/.zshrc\
    && sed -i 's/#force_color_prompt=yes/force_color_prompt=yes/g' ~/.bashrc \
    && mkdir -p ~/.vscode-server/data/Machine

# =============================================================================
# Code Server (Browser-based VS Code)
# =============================================================================

# ─── Install system-wide ──────────────────────────────────────────────────────

USER root
RUN curl -fsSL https://code-server.dev/install.sh | sh

# ─── Configure ────────────────────────────────────────────────────────────────

USER $USERNAME
RUN mkdir -p ~/.config/code-server \
    && echo 'bind-addr: 0.0.0.0:7568' > ~/.config/code-server/config.yaml \
    && echo 'auth: none' >> ~/.config/code-server/config.yaml \
    && echo 'cert: false' >> ~/.config/code-server/config.yaml

# ─── Install extensions ───────────────────────────────────────────────────────

RUN code-server --install-extension ms-python.python \
    && code-server --install-extension aaron-bond.better-comments \
    && code-server --install-extension MS-CEINTL.vscode-language-pack-zh-hans

# =============================================================================
# Cleanup
# =============================================================================

USER root
RUN rm -rf /root/.cache/pip /home/$USERNAME/.cache/pip