# Docker Images

该项目维护多个 Docker 开发镜像。所有镜像均以 **多架构清单（multi-arch manifest list）** 形式发布，同一标签同时提供 `linux/amd64` 与 `linux/arm64` 两个架构的镜像。

| 镜像 | 基础镜像 | 架构 | 用户 | 核心用途 |
|------|----------|------|------|----------|
| `cuda12.8-dev` | nvidia/cuda:12.8.0-cudnn-devel-ubuntu24.04 | amd64 / arm64 | user | 极简基础 CUDA 12.8 深度学习研究环境 |
| `pulp-ui` | node:24-alpine → nginxinc/nginx-unprivileged:1.28-alpine | amd64 / arm64 | nginx (101) | Pulp 管理 UI 生产镜像（nginx 静态托管 + API 反代） |


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


## `pulp-ui` — Pulp 管理 UI 生产镜像

基于 [pulp/pulp-ui](https://github.com/pulp/pulp-ui) 源码多阶段构建：`node:24-alpine` 执行 `npm ci && npm run build` 编译生产 bundle，`nginxinc/nginx-unprivileged` 以非 root 用户托管静态文件。

- **基础**: `node:24-alpine`（构建阶段）→ `nginxinc/nginx-unprivileged:1.28-alpine`（运行阶段）
- **多架构**: `linux/amd64` 与 `linux/arm64`（两个基础镜像均原生多架构，buildx 直接构建）
- **生产特性**:
  - 源码按 `PULP_UI_REF` 固定 tag 拉取，构建可复现
  - `npm ci` 可复现构建，依赖层独立缓存
  - nginx 托管 SPA：history 路由回退（`try_files ... /index.html`）、hash 资源一年不可变缓存、gzip 压缩
  - API 路径反向代理（`/api` `/assets` `/auth` `/extensions` `/pulp` `/static/rest_framework` `/v2`，与上游 dev proxy 一致），上传不受 nginx body 大小限制
  - 非 root 运行、`SIGQUIT` 优雅停止、容器内置健康检查
- **端口自定义**:
  - 容器监听端口: `LISTEN_PORT`（默认 `8002`，非 root 需 `>= 1024`）
  - 宿主机端口: 运行时 `-p <host>:<container>` 决定
  - Pulp API 地址: `PULP_API_URL`（默认 `http://127.0.0.1:8080`，按部署拓扑改为服务名或 `http://host.docker.internal:8080`）

### 构建方法

镜像自包含（构建时按 `PULP_UI_REF` 固定 tag 拉取 pulp-ui 源码，默认 `v0.1.28`），无需外部构建上下文：

```bash
docker build -t ghcr.io/pengweewang/pulp-ui:latest src/pulp-ui

# 指定上游版本（tag 或 commit sha）
docker build --build-arg PULP_UI_REF=v0.1.28 -t ghcr.io/pengweewang/pulp-ui:latest src/pulp-ui

# 多架构
docker buildx build --platform linux/amd64,linux/arm64 \
  -t ghcr.io/pengweewang/pulp-ui:latest --push src/pulp-ui
```

### 使用示例

```bash
# 容器监听 8002，宿主机 8080
docker run -d --name pulp-ui -p 8080:8002 \
  -e PULP_API_URL=http://pulp:8080 ghcr.io/pengweewang/pulp-ui:latest

# 自定义端口: 容器内监听 3000，宿主机发布到 80
docker run -d --name pulp-ui -e LISTEN_PORT=3000 -p 80:3000 \
  -e PULP_API_URL=http://pulp:8080 ghcr.io/pengweewang/pulp-ui:latest
```
