# Docker Images

该项目维护四个 Docker 开发镜像。

| 镜像 | 基础镜像 | 用户 | 核心用途 |
|------|----------|------|----------|
| `aigis-dev` | ubuntu22.04 devcontainer | aigis | 通用 Python/Node.js 开发环境 |
| `aigis-inside` | 基于 aigis-dev | aigis | 浏览器 IDE（Code Server）远程开发 |
| `pytorch2.9-cuda13.0` | nvcr.io/nvidia/pytorch:25.10-py3 | rose | PyTorch 2.9 深度学习训练推理 |
| `pytorch2.11-cuda12.8` | pytorch/pytorch:2.11.0-cuda12.8-cudnn9-devel | rose | PyTorch 2.11 深度学习与微调训练 |
| `llamafactory` | nvcr.io/nvidia/pytorch:25.03-py3 | rose | LLaMA Factory 微调训练专用 |

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

- **基础**: `nvcr.io/nvidia/pytorch:25.03-py3` | **Ubuntu**: 24.04 | **CUDA**: 12.8 | **Python**: 3.12 | **用户**: `rose`
- **系统**: vim, wget, git-lfs, ffmpeg, build-essential, nodejs, npm
- **深度学习**: NGC 预装 PyTorch 2.7.0a0 + torchvision + torchaudio
- **LLaMA Factory**: 源码克隆至 `/work/LlamaFactory` 并以可编辑模式 (`-e`) 安装，包含 metrics 依赖
- **数据/ML**: numpy/pandas/scipy, matplotlib/opencv, scikit-learn/image, prophet/xgboost, jupyterlab/notebook, gradio
- **MLOps**: wandb, mlflow, minio
- **LLM 生态**: transformers, diffusers, trl, bitsandbytes, deepspeed, accelerate, flash-attn, liger-kernel
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

## 构建与运行

```bash
# 构建
docker build -t aigis-dev src/aigis-dev
docker build -t aigis-inside src/aigis-inside
docker build -t pytorch2.9-cuda13.0 src/pytorch2.9-cuda13.0
docker build -t pytorch2.11-cuda12.8 src/pytorch2.11-cuda12.8
docker build -t llamafactory src/llamafactory-dev

# 运行
docker run -it --rm -v "$(pwd):/workspace" aigis-dev
docker run -it --rm -p 7568:7568 -v "$(pwd):/work" aigis-inside
docker run -it --rm --gpus all -p 7568:7568 -v "$(pwd):/work" pytorch2.9-cuda13.0
docker run -it --rm --gpus all -v "$(pwd):/workspace" pytorch2.11-cuda12.8
docker run -it --rm --gpus all -p 6584:6584 -v "$(pwd):/work" llamafactory
```

> **端口说明**: `aigis-inside` 和 `pytorch2.9-cuda13.0` 的 Code Server 监听 7568 端口，`llamafactory` 监听 6584 端口，避免本地端口冲突。
