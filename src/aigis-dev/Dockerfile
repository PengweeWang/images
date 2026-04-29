FROM mcr.microsoft.com/devcontainers/base:ubuntu22.04

# Ali mirror
RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG="en_US.UTF-8"
ENV LANGUAGE="en_US:en"
ARG TZ=Asia/Shanghai

RUN chmod 777 /tmp \
    && apt-get update && apt-get -y upgrade \
    && apt-get -y install \
        tzdata locales lsb-release wget software-properties-common gnupg \
        vim git curl zip unzip sudo zsh \
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone \
    && locale-gen en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

# Update environment variables
ENV LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/usr/local/lib"

# Python
RUN apt-get update && apt-get install -y python3.10 python3-setuptools \
    && curl -LsSf https://bootstrap.pypa.io/pip/get-pip.py | python3.10 \
    && pip3 config set global.index-url https://mirrors.aliyun.com/pypi/simple/ \
    && pip3 install setuptools pandas numpy ipython \
    && pip3 install pytest pytest-xdist pytest-html pytest-mock \
    && pip3 install pytest-randomly pytest-timeout coverage pytest-cov filelock \
    && pip3 install pre-commit rich BeautifulSoup4 allure-pytest \
    && pip3 install plottable matplotlib opencv_python openpyxl jira python-gitlab \
    && pip3 install flake8 black ruff mypy delegator.py build \
    && pip3 install psycopg2-binary peewee python-calamine \
    && pip3 install loguru fastapi aiofiles "uvicorn[standard]" python-multipart "celery[redis]" \
    && pip3 install "pybind11[global]" httpx mcp \
    && curl -LsSf https://astral.sh/uv/install.sh | sh \
    && rm -rf /var/lib/apt/lists/* \
    && pip3 cache purge

# ─── User ─────────────────────────────────────────────────────────────────────

ARG USERNAME=aigis
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

USER $USERNAME
WORKDIR /home/$USERNAME

# Zsh
RUN sudo chsh -s /bin/zsh $USERNAME

# ohmyzsh and vimrc
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

# OpenCode
RUN curl -fsSL https://opencode.ai/install | bash

