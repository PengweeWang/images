# Docker Images

该项目维护多个 Docker 开发镜像。

| 镜像 | 基础镜像 | 用户 | 核心用途 |
|------|----------|------|----------|
| `cuda12.8-dev` | nvidia/cuda:12.8.0-devel-ubuntu24.04 | user | 极简基础 CUDA 12.8 深度学习研究环境 |


## `cuda12.8-dev` — 基础 CUDA 12.8 深度学习研究镜像

- **基础**: `nvidia/cuda:12.8.0-devel-ubuntu24.04` | **CUDA**: 12.8 | **用户**: `user` | **主机名**: `develop` | **时区**: Asia/Shanghai
- **系统工具**: sudo (免密), git, git-lfs, gh (GitHub CLI), curl, wget, cmake, build-essential, zsh, tzdata
- **现代化 CLI**:
  - `neovim`: 官方最新稳定版，集成 Catppuccin-Mocha 主题与常用编程配置
  - `helix (hx)`: 现代化多模态文本编辑器，预装并配置 Catppuccin-Mocha 官方主题与优雅编辑设置
  - `uv`: 官方二进制快速安装，内置阿里云 PyPI 镜像源
  - `eza`: 现代化替代 ls，支持图标与 git 状态展示
  - `nodejs`: 22.x LTS (NodeSource)，内置淘宝 npmmirror 镜像源
  - `tatr`: Tsoding 任务管理工具（Task Tracker），编译安装至 `/usr/local/bin/tatr`
  - `starship`: 预设 `catppuccin-powerline` 风格提示符（已关闭当前时间显示，命令耗时隐藏毫秒）
- **Python 环境**: Python 3.12 + pip，已移除系统 `EXTERNALLY-MANAGED` 限制，支持全局与虚拟环境极速安装
- **Shell**: Oh My Zsh 配合 `git`, `sudo`, `zsh-autosuggestions`, `zsh-syntax-highlighting`, `zsh-completions` 插件
- **场景**: 极简、无冗余、纯净的 CUDA 12.8 深度学习研究与算子开发环境