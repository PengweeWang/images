FROM nvidia/cuda:13.0.2-cudnn-devel-ubuntu24.04
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get clean && \
    apt update -y && \
    apt upgrade -y && \
    apt install -y --no-install-recommends \
    vim \
    ffmpeg \
    wget \
    build-essential \
    python3 \
    python3-dev\
    git \
    git-lfs \
    sudo \
    zsh
RUN rm -rf /usr/lib/python3.12/EXTERNALLY-MANAGED

# ─── pip ─────────────────────────────────────────────────────────────────────

WORKDIR /tmp
RUN wget https://bootstrap.pypa.io/get-pip.py -O /tmp/get-pip.py && \
    python3 get-pip.py && \
    rm -rf /tmp/get-pip.py


# ─── python packages ─────────────────────────────────────────────────────────────────────

# install pytorch 2.9 with cuda 13.0 support
RUN pip3 install -U torch torchvision torchaudio xformers --index-url https://download.pytorch.org/whl/cu130

RUN pip3 install -U \
    imageio \
    jupyterlab \
    matplotlib \
    notebook \
    numpy \
    opencv-contrib-python \
    opencv-python \
    pandas \
    scipy \
    seaborn \
    tifffile \
    timm \
    torchtext \
    tqdm \
    prophet \
    xgboost \
    scikit-learn \
    scikit-image


# note
RUN pip3 install transformers huggingface_hub tokenizers datasets gradio diffusers bitsandbytes optimum accelerate deepspeed transformer-engine


# for MLOPs
RUN pip3 install -U wandb mlflow minio


# ─── User ─────────────────────────────────────────────────────────────────────

ARG USERNAME=rose
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user
# If the specified UID/GID is already occupied by an existing user/group, remove it first
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
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && rm -rf /var/lib/apt/lists/*

# Zsh
RUN sudo chsh -s /bin/zsh $USERNAME

# ohmyzsh and vimrc (installed for user rose)
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

# Code Server (browser-based VS Code) — system-wide install
USER root
RUN curl -fsSL https://code-server.dev/install.sh | sh

USER $USERNAME
RUN mkdir -p ~/.config/code-server \
    && echo 'bind-addr: 0.0.0.0:7568' > ~/.config/code-server/config.yaml \
    && echo 'auth: none' >> ~/.config/code-server/config.yaml \
    && echo 'cert: false' >> ~/.config/code-server/config.yaml

# VS Code extensions (for user rose)
RUN code-server --install-extension ms-python.python \
    && code-server --install-extension aaron-bond.better-comments \
    && code-server --install-extension MS-CEINTL.vscode-language-pack-zh-hans