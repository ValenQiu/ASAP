# ASAP Docker 快速参考

## 🚀 一键启动

```bash
# 方式 1: 从项目根目录（推荐）
cd /home/qiulm/sources/ASAP
./docker_start.sh

# 方式 2: 从 docker 文件夹
cd /home/qiulm/sources/ASAP/docker
./docker_quick_start.sh
```

## 📦 文件位置

所有 Docker 文件位于 `docker/` 文件夹：

- `docker/Dockerfile` - Docker 镜像定义
- `docker/docker-compose.yml` - Docker Compose 配置
- `docker/.dockerignore` - Docker 构建忽略文件
- `docker/docker_quick_start.sh` - 启动脚本
- `docker/DOCKER_SETUP_CN.md` - 完整中文文档
- `docker_start.sh` - 根目录快捷启动脚本

## ⚡ 常用命令

所有命令从 `docker/` 文件夹执行：

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

# 重新构建
docker-compose build --no-cache
```

## 🔧 容器内环境

### Conda 环境

1. **hvgym** (Python 3.8) - 训练环境
   - PyTorch 1.13.1 + CUDA 11.7
   - Isaac Gym 支持
   - GPU Pipeline 已启用

2. **asap_deploy** (Python 3.10) - 部署环境
   - ROS2 Humble
   - MuJoCo + ONNX Runtime

### 快速开始

```bash
# 进入容器
docker exec -it asap_container bash

# 训练
conda activate hvgym
cd /workspace/ASAP
python humanoidverse/train_agent.py ...

# 部署
conda activate asap_deploy
cd /workspace/ASAP/sim2real
python sim_env/base_sim.py ...
```

## 📖 完整文档

详细配置和故障排除请参考：`DOCKER_SETUP_CN.md`

## ✅ 环境验证

```bash
# 检查 GPU
docker exec asap_container nvidia-smi

# 检查环境
docker exec asap_container bash -c "conda activate hvgym && python -c 'import torch; print(torch.cuda.is_available())'"

# 检查 Isaac Gym
docker exec asap_container bash -c "conda activate hvgym && python -c 'from isaacgym import gymapi; print(\"Isaac Gym OK\")'"
```

## 🎯 下一步

1. ✅ 启动容器: `./docker_start.sh` (根目录) 或 `cd docker && ./docker_quick_start.sh`
2. ✅ 进入环境: `docker exec -it asap_container bash`
3. ✅ 激活 conda: `conda activate hvgym`
4. ✅ 开始训练或评估！

---

**需要帮助？** 查看完整文档 `docker/DOCKER_SETUP_CN.md`

