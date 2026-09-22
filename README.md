# Docker Images

该项目维护多个 Docker 开发镜像。

| 镜像 | 基础镜像 | 用户 | 核心用途 |
|------|----------|------|----------|
| `aigis-dev` | ubuntu22.04 devcontainer | aigis | 通用 Python/Node.js 开发环境 |
| `aigis-inside` | 基于 aigis-dev | aigis | 浏览器 IDE（Code Server）远程开发 |
| `cuda12.8-dev` | nvidia/cuda:12.8.0-devel-ubuntu24.04 | user | 极简基础 CUDA 12.8 深度学习研究环境 |
| `pytorch2.9-cuda13.0` | nvcr.io/nvidia/pytorch:25.10-py3 | rose | PyTorch 2.9 深度学习训练推理 |
| `pytorch2.11-cuda12.8` | pytorch/pytorch:2.11.0-cuda12.8-cudnn9-devel | rose | PyTorch 2.11 深度学习与微调训练 |
| `llamafactory` | pytorch/pytorch:2.11.0-cuda12.8-cudnn9-devel | rose | LLaMA Factory 微调训练专用 |

---

## `aigis-dev` — 通用开发镜像

- **基础**: `mcr.microsoft.com/devcontainers/base:ubuntu22.04` | **用户**: `aigis` | **时区**: Asia/Shanghai
- **系统**: vim, git, curl, wget, zip, unzip, sudo, zsh
- **Node**: 24.x (NodeSource)
- **Python**: 3.10 + uv + 阿里云镜像源
- **Python 包**: pytest 全家桶, flake8/black/ruff/mypy, fastapi/uvicorn/celery, pandas/numpy/matplotlib/opencv, httpx/mcp, psycopg2/peewee, pybind11
- **CLI**: OpenCode
- **Shell**: Oh My Zsh (ys 主题, zsh-autosuggestions/syntax-highlighting/completions), amix/vimrc
- **场景**: 通用 Python/Node.js 开发、DevContainer

---

## `aigis-inside` — 浏览器 IDE 镜像

- **基础**: 在 `aigis-dev` 之上增加
- **附加**: Code Server (0.0.0.0:7568, 无认证), `/work` 挂载目录
- **场景**: 远程开发、云端 IDE、无 GUI 环境编码

---

## `pytorch2.9-cuda13.0` — 深度学习镜像

- **基础**: `nvcr.io/nvidia/pytorch:25.10-py3` | **Ubuntu**: 24.04 | **CUDA**: 13.0.2 | **Python**: 3.12 | **用户**: `rose`
- **系统**: vim, wget, git-lfs, ffmpeg, build-essential
- **深度学习**: NGC 预装 PyTorch 2.9 + torchvision + torchaudio, transformers/diffusers/timm, bitsandbytes/deepspeed/accelerate/optimum/transformer-engine
- **LLaMA Factory**: 作为 Python 包集成安装
- **数据/ML**: numpy/pandas/scipy, scikit-learn/image, prophet/xgboost, jupyterlab/notebook
- **MLOps**: wandb, mlflow, minio
- **Shell**: Oh My Zsh + amix/vimrc + Code Server (7568)
- **VS Code 扩展**: Python, Better Comments, 中文语言包
- **Python 包**: 通过单独 `requirements.txt` 管理，便于维护
- **场景**: PyTorch 训练推理、NLP/CV/多模态大模型、模型量化与调优

> 仅支持 CUDA 13.0+ GPU。低版本 CUDA 需切换 PyTorch 索引源。

---

## `llamafactory` — LLaMA Factory 微调训练镜像

- **基础**: `pytorch/pytorch:2.11.0-cuda12.8-cudnn9-devel` | **CUDA**: 12.8 | **用户**: `rose`
- **系统**: vim, wget, git-lfs, ffmpeg, build-essential, nodejs, npm
- **深度学习**: 官方预装 PyTorch 2.11 + torchvision + torchaudio, deepspeed
- **LLaMA Factory**: 源码克隆至 `/work/LlamaFactory` 并以可编辑模式 (`-e`) 安装，包含 metrics 依赖
- **Shell**: Oh My Zsh + amix/vimrc + Code Server (6584)
- **VS Code 扩展**: Python, Better Comments, 中文语言包
- **场景**: LLaMA Factory 大模型微调训练、RLHF/DPO、模型评估


---

## `pytorch2.11-cuda12.8` — PyTorch 2.11 官方基础开发镜像

- **基础**: `pytorch/pytorch:2.11.0-cuda12.8-cudnn9-devel` | **CUDA**: 12.8 | **用户**: `rose`
- **系统**: vim, wget, git-lfs, ffmpeg, build-essential, zsh
- **深度学习**: 官方 PyTorch 2.11 + torchvision + torchaudio 官方预装环境
- **包输出**: 镜像构建阶段输出官方基础镜像内所有已安装的 pip 包列表 (`pip list`)
- **Shell**: Oh My Zsh (ys 主题, zsh-autosuggestions/syntax-highlighting/completions) + amix/vimrc
- **场景**: PyTorch 2.11 官方纯净环境检测与容器交互式开发

---

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

---

## 构建与运行

```bash
# 构建
docker build -t aigis-dev src/aigis-dev
docker build -t aigis-inside src/aigis-inside
docker build -t cuda12.8-dev src/cuda12.8-dev
docker build -t pytorch2.9-cuda13.0 src/pytorch2.9-cuda13.0
docker build -t pytorch2.11-cuda12.8 src/pytorch2.11-cuda12.8
docker build -t llamafactory src/llamafactory-dev

# 运行
docker run -it --rm -v "$(pwd):/workspace" aigis-dev
docker run -it --rm -p 7568:7568 -v "$(pwd):/work" aigis-inside
docker run -it --rm --gpus all -h develop -v "$(pwd):/workspace" cuda12.8-dev
docker run -it --rm --gpus all -p 7568:7568 -v "$(pwd):/work" pytorch2.9-cuda13.0
docker run -it --rm --gpus all -v "$(pwd):/workspace" pytorch2.11-cuda12.8
docker run -it --rm --gpus all -p 6584:6584 -v "$(pwd):/work" llamafactory
```

> **端口说明**: `aigis-inside` 和 `pytorch2.9-cuda13.0` 的 Code Server 监听 7568 端口，`llamafactory` 监听 6584 端口，避免本地端口冲突。
>
> **CI/CD 包清单导出**: 若需在 GitHub Actions 构建时导出镜像内的 Python 包清单并生成 Markdown 表格与 Artifact，只需在对应镜像目录中放置 `.list-packages` 文件。

