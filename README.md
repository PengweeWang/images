# Docker Images

该项目维护三个 Docker 开发镜像。

| 镜像 | 基础镜像 | 用户 | 核心用途 |
|------|----------|------|----------|
| `aigis-dev` | ubuntu22.04 devcontainer | aigis | 通用 Python/Node.js 开发环境 |
| `aigis-inside` | 基于 aigis-dev | aigis | 浏览器 IDE（Code Server）远程开发 |
| `pytorch2.9-cuda13.0` | cuda:13.0.2-cudnn-devel-ubuntu24.04 | rose | PyTorch 2.9 深度学习训练推理 |

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

- **基础**: `nvidia/cuda:13.0.2-cudnn-devel-ubuntu24.04` | **用户**: `rose`
- **系统**: vim, wget, git-lfs, ffmpeg, build-essential
- **深度学习**: PyTorch 2.9 + torchvision + torchaudio + xformers (cu130), transformers/diffusers/timm, bitsandbytes/deepspeed/accelerate/optimum/transformer-engine
- **数据/ML**: numpy/pandas/scipy, scikit-learn/image, prophet/xgboost, jupyterlab/notebook
- **MLOps**: wandb, mlflow, minio
- **Shell**: Oh My Zsh + amix/vimrc + Code Server (7568)
- **VS Code 扩展**: Python, Better Comments, 中文语言包
- **场景**: PyTorch 训练推理、NLP/CV/多模态大模型、模型量化与调优

> 仅支持 CUDA 13.0+ GPU。低版本 CUDA 需切换 PyTorch 索引源。

---

## 构建与运行

```bash
docker build -t aigis-dev src/aigis-dev
docker build -t aigis-inside src/aigis-inside
docker build -t pytorch2.9-cuda13.0 src/pytorch2.9-cuda13.0

# 运行
docker run -it --rm -v "$(pwd):/workspace" aigis-dev
docker run -it --rm -p 7568:7568 -v "$(pwd):/work" aigis-inside
docker run -it --rm --gpus all -p 7568:7568 -v "$(pwd):/work" pytorch2.9-cuda13.0
```
