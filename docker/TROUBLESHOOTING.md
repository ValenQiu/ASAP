# Docker 构建故障排除

## 常见问题和解决方案

### 1. Conda Terms of Service 错误

**错误信息:**
```
CondaToSNonInteractiveError: Terms of Service have not been accepted
```

**原因:**
Anaconda 最近更新了服务条款，要求用户接受后才能使用。

**解决方案:**
✅ **已修复** - Dockerfile 已更新，会自动：
1. 配置使用 conda-forge 渠道
2. 自动接受服务条款
3. 使用 libmamba solver 加速

如果仍有问题，手动接受：
```bash
conda tos accept
```

---

### 2. Docker 构建缓存问题

**问题:**
构建过程中遇到错误，修复后重新构建仍然失败。

**解决方案:**
```bash
cd /home/qiulm/sources/ASAP/docker

# 清除缓存重新构建
docker-compose build --no-cache

# 或使用 docker compose
docker compose build --no-cache
```

---

### 3. 网络超时或下载失败

**错误信息:**
```
ERROR: failed to solve: failed to fetch
Could not connect to repo.anaconda.com
```

**解决方案:**
```bash
# 1. 检查网络连接
ping repo.anaconda.com

# 2. 使用国内镜像（中国用户）
# 在 Dockerfile 中添加（已在 conda config 部分）：
# RUN conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
# RUN conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free

# 3. 设置 Docker 代理（如果有）
export HTTP_PROXY=http://your-proxy:port
export HTTPS_PROXY=http://your-proxy:port
docker-compose build
```

---

### 4. GPU 支持问题

**错误信息:**
```
could not select device driver "" with capabilities: [[gpu]]
```

**解决方案:**
```bash
# 1. 检查 NVIDIA Docker Runtime
docker run --rm --gpus all nvidia/cuda:11.7.1-base-ubuntu20.04 nvidia-smi

# 2. 如果失败，安装/重新安装
sudo apt-get install --reinstall nvidia-docker2
sudo systemctl restart docker

# 3. 验证配置
cat /etc/docker/daemon.json
# 应该包含:
# {
#     "runtimes": {
#         "nvidia": {
#             "path": "nvidia-container-runtime",
#             "runtimeArgs": []
#         }
#     }
# }
```

---

### 5. 磁盘空间不足

**错误信息:**
```
no space left on device
```

**解决方案:**
```bash
# 1. 检查磁盘空间
df -h

# 2. 清理 Docker
docker system prune -a --volumes
# 警告：这会删除所有未使用的镜像、容器、卷

# 3. 清理构建缓存
docker builder prune

# 4. 检查具体占用
docker system df
```

---

### 6. Permission Denied 错误

**错误信息:**
```
permission denied while trying to connect to Docker daemon
```

**解决方案:**
```bash
# 1. 添加用户到 docker 组
sudo usermod -aG docker $USER

# 2. 重新登录或刷新组
newgrp docker

# 3. 验证
docker ps
```

---

### 7. X11 显示问题

**问题:**
容器启动成功，但 GUI 程序（如 MuJoCo viewer）无法显示。

**解决方案:**
```bash
# 1. 允许 X11 访问
xhost +local:docker

# 2. 检查 DISPLAY 变量
echo $DISPLAY

# 3. 验证容器内
docker exec asap_container bash -c "echo \$DISPLAY"

# 4. 如果通过 SSH 连接，启用 X11 转发
ssh -X user@host
```

---

### 8. PyTorch CUDA 版本不匹配

**错误信息:**
```
RuntimeError: CUDA error: no kernel image is available for execution
```

**解决方案:**
```bash
# 1. 检查主机 CUDA 版本
nvidia-smi

# 2. 如果驱动版本 < 450，需要降级 CUDA
# 在 Dockerfile 中修改基础镜像:
FROM nvidia/cuda:11.3.1-cudnn8-devel-ubuntu20.04

# 并修改 PyTorch 安装:
pip install torch==1.12.1+cu113 torchvision==0.13.1+cu113 --extra-index-url https://download.pytorch.org/whl/cu113
```

---

### 9. ROS2 安装失败

**错误信息:**
```
PackagesNotFoundError: The following packages are not available from current channels:
  - ros-humble-desktop
```

**解决方案:**
```bash
# 1. 验证 robostack 渠道配置
# 在容器内检查:
docker exec asap_container bash -c "conda activate asap_deploy && conda config --show channels"

# 2. 手动添加渠道
docker exec asap_container bash -c "conda activate asap_deploy && \
    conda config --add channels conda-forge && \
    conda config --add channels robostack-staging"

# 3. 如果仍然失败，使用系统 ROS2（需要修改 Dockerfile）
```

---

### 10. 容器启动后立即退出

**问题:**
`docker-compose up -d` 后容器立即停止。

**解决方案:**
```bash
# 1. 查看容器日志
docker-compose logs asap

# 2. 查看最后几个退出的容器
docker ps -a

# 3. 检查具体错误
docker logs asap_container

# 4. 以交互模式启动调试
docker run -it --rm --gpus all asap:latest bash
```

---

## 完全重建流程

如果遇到各种奇怪问题，完全重建：

```bash
cd /home/qiulm/sources/ASAP/docker

# 1. 停止并删除容器
docker-compose down

# 2. 删除镜像
docker rmi asap:latest

# 3. 清理 Docker 系统（可选，谨慎使用）
docker system prune -a

# 4. 无缓存重新构建
docker-compose build --no-cache

# 5. 启动
docker-compose up -d
```

---

## 获取更多帮助

### 查看完整日志

```bash
# Docker 构建日志
docker-compose build 2>&1 | tee build.log

# 容器运行日志
docker-compose logs -f

# 特定容器日志
docker logs asap_container
```

### 进入容器调试

```bash
# 进入正在运行的容器
docker exec -it asap_container bash

# 以 root 身份进入
docker exec -u root -it asap_container bash

# 临时启动容器调试
docker run -it --rm --gpus all asap:latest bash
```

### 检查配置

```bash
# Docker 信息
docker info

# Docker Compose 配置验证
docker-compose config

# 镜像历史
docker history asap:latest
```

---

## 联系支持

如果以上方法都无法解决问题，请提供：

1. 错误信息完整输出
2. Docker 版本: `docker --version`
3. 系统信息: `uname -a`
4. GPU 信息: `nvidia-smi`
5. 构建日志: `docker-compose build 2>&1 | tee build.log`

---

更新日期: 2025-11-03

