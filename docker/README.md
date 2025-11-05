# ASAP Docker 环境

这个文件夹包含所有 Docker 相关的配置文件。

## 📁 文件结构

```
docker/
├── Dockerfile              # Docker 镜像定义
├── docker-compose.yml      # Docker Compose 配置
├── .dockerignore          # Docker 构建忽略文件
├── docker_quick_start.sh  # 快速启动脚本
├── DOCKER_SETUP_CN.md     # 完整中文文档（必读）
├── DOCKER_README.md       # 快速参考
└── README.md              # 本文件
```

## 🚀 快速开始

### 方式 1: 从根目录启动（推荐）

```bash
cd /home/qiulm/sources/ASAP
./docker_start.sh
```

### 方式 2: 从 docker 文件夹启动

```bash
cd /home/qiulm/sources/ASAP/docker
./docker_quick_start.sh
```

### 方式 3: 使用 Docker Compose

```bash
cd /home/qiulm/sources/ASAP/docker
docker-compose up -d
docker exec -it asap_container bash
```

## 📚 完整文档

- **DOCKER_SETUP_CN.md** - 详细的中文安装和使用指南
- **DOCKER_README.md** - 快速参考卡

## ⚡ 常用命令

所有命令都从 `docker` 文件夹执行：

```bash
cd /home/qiulm/sources/ASAP/docker

# 构建镜像
docker-compose build

# 启动容器
docker-compose up -d

# 进入容器
docker exec -it asap_container bash

# 停止容器
docker-compose down

# 查看日志
docker-compose logs -f

# 重新构建（无缓存）
docker-compose build --no-cache
```

## 🎯 容器内环境

### Conda 环境

1. **hvgym** (Python 3.8) - 训练和评估
   ```bash
   conda activate hvgym
   cd /workspace/ASAP
   python humanoidverse/train_agent.py ...
   ```

2. **asap_deploy** (Python 3.10) - Sim2Sim/Sim2Real
   ```bash
   conda activate asap_deploy
   cd /workspace/ASAP/sim2real
   python sim_env/base_sim.py ...
   ```

## 🔧 文件说明

### Dockerfile
定义 Docker 镜像，包括：
- 基础镜像：NVIDIA CUDA 11.7.1 + cuDNN 8
- Python 环境：Miniconda with Python 3.8 & 3.10
- 依赖包：PyTorch, Isaac Gym, ROS2 等
- 环境变量：GPU Pipeline 支持

### docker-compose.yml
Docker Compose 配置，包括：
- GPU 支持配置
- 卷挂载（代码、X11 等）
- 环境变量设置
- 网络和资源配置

### .dockerignore
构建时忽略的文件，避免：
- 不必要的大文件
- 临时文件和缓存
- 训练日志和检查点
- Git 仓库

## 🎨 自定义配置

### 修改 Python 版本

编辑 `Dockerfile`:
```dockerfile
RUN conda create -n hvgym python=3.9 -y  # 改成 3.9
```

### 添加依赖包

编辑 `Dockerfile`:
```dockerfile
RUN source activate hvgym && \
    pip install your_package_here
```

### 修改 CUDA 版本

编辑 `Dockerfile` 第一行:
```dockerfile
FROM nvidia/cuda:11.8.0-cudnn8-devel-ubuntu20.04
```

## 📊 目录映射

| 容器内路径 | 主机路径 | 说明 |
|-----------|---------|------|
| `/workspace/ASAP` | `ASAP/` | 项目根目录 |
| `/tmp/.X11-unix` | `/tmp/.X11-unix` | X11 显示 |
| `/root/.cache/torch_extensions` | `~/.cache/torch_extensions` | PyTorch 扩展缓存 |

## ⚠️ 注意事项

1. **路径问题**
   - 所有 Docker 命令应该从 `docker/` 文件夹执行
   - 或使用根目录的 `./docker_start.sh` 快捷方式

2. **构建位置**
   - `docker-compose.yml` 中 `context: ..` 指向项目根目录
   - `dockerfile: docker/Dockerfile` 指向 Docker 文件夹中的 Dockerfile

3. **文件权限**
   - 容器内创建的文件可能属于 root
   - 需要时使用 `chown` 修改所有者

## 🔍 故障排除

### 问题：找不到 Dockerfile

**错误:** `Cannot locate specified Dockerfile: docker/Dockerfile`

**解决:**
```bash
# 确保在 docker 文件夹中
cd /home/qiulm/sources/ASAP/docker
docker-compose up -d
```

### 问题：代码修改不生效

**原因:** 代码是挂载的，应该实时同步

**检查:**
```bash
# 容器内
ls -la /workspace/ASAP

# 应该看到主机上的所有文件
```

### 问题：GPU 不可用

**检查:**
```bash
# 容器内
nvidia-smi

# 应该显示 GPU 信息
```

## 📖 更多信息

详细文档请参考：
- **DOCKER_SETUP_CN.md** - 完整安装和配置指南
- **DOCKER_README.md** - 快速参考手册

## ✨ 快速验证

```bash
# 1. 启动容器
cd /home/qiulm/sources/ASAP/docker
docker-compose up -d

# 2. 验证环境
docker exec asap_container bash -c "conda activate hvgym && python -c 'import torch; print(f\"PyTorch: {torch.__version__}, CUDA: {torch.cuda.is_available()}\")'"

# 3. 验证 Isaac Gym
docker exec asap_container bash -c "conda activate hvgym && python -c 'from isaacgym import gymapi; print(\"Isaac Gym OK\")'"
```

如果都成功，说明环境配置正确！🎉

---

**需要帮助？** 查看 `DOCKER_SETUP_CN.md` 获取详细文档

