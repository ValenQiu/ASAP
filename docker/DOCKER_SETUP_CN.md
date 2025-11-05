# ASAP Docker 环境配置指南

## 📋 前置要求

### 1. 安装 Docker
```bash
# 安装 Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 添加当前用户到 docker 组（避免每次使用 sudo）
sudo usermod -aG docker $USER

# 重新登录或运行
newgrp docker
```

### 2. 安装 NVIDIA Docker Runtime
```bash
# 添加 NVIDIA Docker 仓库
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# 安装 nvidia-docker2
sudo apt-get update
sudo apt-get install -y nvidia-docker2

# 重启 Docker 服务
sudo systemctl restart docker

# 测试 GPU 支持
docker run --rm --gpus all nvidia/cuda:11.7.1-base-ubuntu20.04 nvidia-smi
```

### 3. 安装 Docker Compose（如果未安装）
```bash
sudo apt-get install docker-compose-plugin
# 或
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

---

## 🚀 快速开始

### 方法 1: 使用快速启动脚本（最简单）

```bash
cd /home/qiulm/sources/ASAP

# 一键启动（推荐）
./docker_start.sh
```

### 方法 2: 使用 Docker Compose

```bash
cd /home/qiulm/sources/ASAP/docker

# 允许 X11 显示（用于 GUI）
xhost +local:docker

# 构建并启动容器
docker-compose up -d

# 进入容器
docker exec -it asap_container bash

# 在容器内激活环境
conda activate hvgym

# 开始使用！
cd /workspace/ASAP
```

### 方法 3: 使用 Docker 命令

```bash
cd /home/qiulm/sources/ASAP

# 构建镜像
docker build -t asap:latest -f docker/Dockerfile .

# 允许 X11 显示
xhost +local:docker

# 运行容器
docker run -it --rm \
  --gpus all \
  --name asap_container \
  --network host \
  --shm-size=8g \
  -e DISPLAY=$DISPLAY \
  -e NVIDIA_VISIBLE_DEVICES=all \
  -e NVIDIA_DRIVER_CAPABILITIES=all \
  -v $(pwd):/workspace/ASAP \
  -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
  -v ~/.Xauthority:/root/.Xauthority:rw \
  asap:latest bash
```

---

## 📦 容器内环境说明

### 已配置的 Conda 环境

1. **hvgym** (Python 3.8) - 用于训练和评估
   - PyTorch 1.13.1 + CUDA 11.7
   - Isaac Gym 支持
   - Hydra, Loguru, Wandb 等
   - GPU Pipeline 自动启用

2. **asap_deploy** (Python 3.10) - 用于 Sim2Sim/Sim2Real
   - ROS2 Humble
   - MuJoCo
   - ONNX Runtime

### 自动配置的环境变量

容器已自动设置：
- ✅ `LD_LIBRARY_PATH` - Isaac Gym 库路径
- ✅ `VK_ICD_FILENAMES` - GPU Pipeline 支持
- ✅ GPU 加速启用

---

## 🎯 使用示例

### 1. 训练模型

```bash
# 进入容器
docker exec -it asap_container bash

# 激活环境
conda activate hvgym
cd /workspace/ASAP

# 安装 ASAP 和依赖
pip install -e .
pip install -e isaac_utils

# 开始训练（示例）
python humanoidverse/train_agent.py \
  +simulator=isaacgym \
  +exp=motion_tracking \
  +robot=g1/g1_29dof_anneal_23dof \
  num_envs=4096 \
  project_name=MyTraining
```

### 2. 评估模型

```bash
conda activate hvgym
cd /workspace/ASAP

python humanoidverse/eval_agent.py \
  +checkpoint=logs/MotionTracking/.../model_15000.pt
```

### 3. 可视化运动数据

```bash
conda activate hvgym
cd /workspace/ASAP

python scripts/vis/vis_q_mj.py \
  +robot=g1/g1_29dof_anneal_23dof \
  +visualize_motion_file="humanoidverse/data/motions/ANTA/Jump/0-jump3_origin_inter0.5_S0-30_E60-30.pkl"
```

### 4. Sim2Sim 部署

```bash
# 终端 1 - 启动 MuJoCo 仿真
conda activate asap_deploy
cd /workspace/ASAP/sim2real
python sim_env/base_sim.py --config=config/g1_29dof_hist.yaml
```

```bash
# 终端 2 - 运行策略（新开一个终端）
docker exec -it asap_container bash
conda activate asap_deploy
cd /workspace/ASAP/sim2real
python rl_policy/deepmimic_dec_loco_height.py \
  --config=config/g1_29dof_hist.yaml \
  --loco_model_path=./models/dec_loco/.../model_6600.onnx \
  --mimic_model_paths=./models/mimic
```

---

## 🔧 常用 Docker 命令

### 容器管理

```bash
# 查看运行的容器
docker ps

# 查看所有容器（包括停止的）
docker ps -a

# 停止容器
docker stop asap_container

# 启动已停止的容器
docker start asap_container

# 重启容器
docker restart asap_container

# 删除容器
docker rm asap_container

# 删除镜像
docker rmi asap:latest
```

### 进入容器的不同方式

```bash
# 方式 1: 交互式 bash
docker exec -it asap_container bash

# 方式 2: 直接运行命令
docker exec asap_container conda activate hvgym && python script.py

# 方式 3: 以 root 用户进入
docker exec -u root -it asap_container bash
```

### 文件传输

```bash
# 从容器复制到主机
docker cp asap_container:/workspace/ASAP/logs ./local_logs

# 从主机复制到容器
docker cp ./local_file asap_container:/workspace/ASAP/
```

### 查看日志

```bash
# 查看容器日志
docker logs asap_container

# 实时查看日志
docker logs -f asap_container
```

---

## 🛠️ 故障排除

### 1. GPU 无法访问

**问题:** `nvidia-smi` 在容器内不工作

**解决方案:**
```bash
# 检查 NVIDIA Docker runtime
docker run --rm --gpus all nvidia/cuda:11.7.1-base-ubuntu20.04 nvidia-smi

# 如果失败，重新安装 nvidia-docker2
sudo apt-get install --reinstall nvidia-docker2
sudo systemctl restart docker
```

### 2. GUI 程序无法显示

**问题:** MuJoCo viewer 或其他 GUI 无法打开

**解决方案:**
```bash
# 允许 Docker 访问 X11
xhost +local:docker

# 重新启动容器
docker-compose restart

# 验证 DISPLAY 变量
docker exec asap_container echo $DISPLAY
```

### 3. 共享内存不足

**问题:** PyTorch DataLoader 报错 "cannot allocate memory in static TLS block"

**解决方案:**
```yaml
# 在 docker-compose.yml 中已设置
shm_size: '8gb'

# 或在 docker run 命令中添加
--shm-size=8g
```

### 4. 权限问题

**问题:** 容器内创建的文件在主机上无法访问

**解决方案:**
```bash
# 方式 1: 修改文件所有者
docker exec asap_container chown -R $(id -u):$(id -g) /workspace/ASAP/logs

# 方式 2: 使用用户命名空间（重建容器时）
docker run --user $(id -u):$(id -g) ...
```

### 5. Isaac Gym 导入错误

**问题:** `libpython3.8.so.1.0: cannot open shared object file`

**解决方案:**
容器已自动配置，但如果遇到问题：
```bash
# 进入容器并手动设置
conda activate hvgym
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
```

---

## 🎨 自定义 Docker 镜像

### 添加额外的依赖

编辑 `Dockerfile`，在对应环境中添加：

```dockerfile
# 为 hvgym 添加包
RUN source activate hvgym && \
    pip install your_package_here

# 为 asap_deploy 添加包
RUN source activate asap_deploy && \
    pip install another_package
```

重新构建：
```bash
docker-compose build --no-cache
```

### 修改基础镜像

如果需要不同的 CUDA 版本：

```dockerfile
# 在 Dockerfile 第一行修改
FROM nvidia/cuda:11.8.0-cudnn8-devel-ubuntu20.04
```

---

## 📊 性能优化

### 1. 使用 BuildKit 加速构建

```bash
export DOCKER_BUILDKIT=1
docker-compose build
```

### 2. 缓存 Conda 包

```yaml
# 在 docker-compose.yml 中添加 volume
volumes:
  - ~/.conda/pkgs:/opt/conda/pkgs
```

### 3. 多阶段构建（高级）

```dockerfile
# 示例：分离构建和运行环境
FROM nvidia/cuda:11.7.1-cudnn8-devel-ubuntu20.04 as builder
# ... 构建步骤 ...

FROM nvidia/cuda:11.7.1-cudnn8-runtime-ubuntu20.04
COPY --from=builder /opt/conda /opt/conda
```

---

## 🔐 安全建议

1. **不要在镜像中包含敏感信息**
   - 使用环境变量传递密钥
   - 使用 Docker secrets

2. **定期更新基础镜像**
   ```bash
   docker pull nvidia/cuda:11.7.1-cudnn8-devel-ubuntu20.04
   docker-compose build --no-cache
   ```

3. **限制容器资源使用**
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '8'
         memory: 32G
   ```

---

## 📚 更多资源

- [Docker 官方文档](https://docs.docker.com/)
- [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-docker)
- [Docker Compose 文档](https://docs.docker.com/compose/)

---

## ✅ 检查清单

配置完成后，验证以下项目：

- [ ] `docker --version` 显示版本
- [ ] `docker-compose --version` 显示版本
- [ ] `nvidia-docker` 已安装
- [ ] GPU 在容器内可访问（`nvidia-smi`）
- [ ] X11 转发工作（GUI 程序可显示）
- [ ] Conda 环境正常（`hvgym` 和 `asap_deploy`）
- [ ] Isaac Gym 可以导入
- [ ] 文件挂载正常（主机和容器文件同步）

---

## 🎉 完成！

现在你已经有一个完整配置的 Docker 环境，可以开始使用 ASAP 了！

**快速启动命令：**
```bash
cd /home/qiulm/sources/ASAP
xhost +local:docker
docker-compose up -d
docker exec -it asap_container bash
conda activate hvgym
```

祝训练愉快！🚀

