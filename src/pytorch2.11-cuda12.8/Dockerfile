# =============================================================================
# Base Image
# =============================================================================

FROM pytorch/pytorch:2.11.0-cuda12.8-cudnn9-devel
ENV DEBIAN_FRONTEND=noninteractive

# =============================================================================
# System Dependencies
# =============================================================================

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
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

# ─── Remove pip externally-managed restriction (if present) ──────────────────

RUN rm -rf /usr/lib/python3*/EXTERNALLY-MANAGED 2>/dev/null || true

# ─── Print all pre-installed pip packages in base image during build ─────────

RUN echo "=================================================================" \
    && echo "             BASE IMAGE PRE-INSTALLED PIP PACKAGES               " \
    && echo "=================================================================" \
    && pip list --format=columns \
    && echo "================================================================="

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
    && echo 'setopt no_nomatch # disable * match' >> ~/.zshrc \
    && echo '' >> ~/.zshrc \
    && sed -i 's/#force_color_prompt=yes/force_color_prompt=yes/g' ~/.bashrc \
    && mkdir -p ~/.vscode-server/data/Machine

# =============================================================================
# Cleanup
# =============================================================================

USER root
RUN rm -rf /root/.cache/pip /home/$USERNAME/.cache/pip
