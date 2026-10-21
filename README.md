# Docker Images

该项目维护多个 Docker 开发镜像。所有镜像均以 **多架构清单（multi-arch manifest list）** 形式发布，同一标签同时提供 `linux/amd64` 与 `linux/arm64` 两个架构的镜像。

| 镜像 | 基础镜像 | 架构 | 用户 | 核心用途 |
|------|----------|------|------|----------|
| `cuda12.8-dev` | nvidia/cuda:12.8.0-cudnn-devel-ubuntu24.04 | amd64 / arm64 | user | 极简基础 CUDA 12.8 深度学习研究环境 |


## `cuda12.8-dev` — 基础 CUDA 12.8 深度学习研究镜像

- **基础**: `nvidia/cuda:12.8.0-cudnn-devel-ubuntu24.04` | **CUDA**: 12.8 | **用户**: `user` | **主机名**: `develop` | **时区**: Asia/Shanghai
- **多架构**: `linux/amd64` 与 `linux/arm64`（arm64 使用 NVIDIA SBSA / Grace 平台镜像）
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

### 架构说明

Dockerfile 通过 BuildKit 的 `TARGETARCH` 自动识别目标架构，并下载对应架构的二进制文件：

| 工具 | amd64 | arm64 |
|------|-------|--------|
| neovim | `nvim-linux-x86_64.tar.gz` | `nvim-linux-arm64.tar.gz` |
| helix | `helix-<tag>-x86_64-linux.tar.xz` | `helix-<tag>-aarch64-linux.tar.xz` |
| eza | `eza_x86_64-unknown-linux-musl.tar.gz` | `eza_aarch64-unknown-linux-gnu.tar.gz` |
| uv | `ghcr.io/astral-sh/uv:latest`（多架构镜像，自动匹配） | 同左 |
| starship | 安装脚本自动识别 `uname -m` | 同左 |

### 使用示例

```bash
# 自动拉取与当前机器匹配的架构
docker pull ghcr.io/pengweewang/cuda12.8-dev:latest

# 手动指定架构
docker pull --platform linux/arm64 ghcr.io/pengweewang/cuda12.8-dev:latest
```
