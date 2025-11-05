# ASAP Docker 环境

所有 Docker 相关文件已整理到 `docker/` 文件夹。

## 🚀 快速启动

### 最简单的方式

```bash
cd /home/qiulm/sources/ASAP
./docker_start.sh
```

### 或者进入 docker 文件夹

```bash
cd /home/qiulm/sources/ASAP/docker
./docker_quick_start.sh
```

## 📁 文件位置

```
ASAP/
├── docker/                      # 所有 Docker 文件
│   ├── Dockerfile              # 镜像定义
│   ├── docker-compose.yml      # Compose 配置
│   ├── .dockerignore           # 构建忽略
│   ├── docker_quick_start.sh   # 启动脚本
│   ├── DOCKER_SETUP_CN.md      # 完整文档 ⭐
│   ├── DOCKER_README.md        # 快速参考
│   └── README.md               # Docker 文件夹说明
│
└── docker_start.sh             # 根目录快捷启动（推荐使用）
```

## 📚 完整文档

详细的安装、配置和使用说明，请查看：

**[docker/DOCKER_SETUP_CN.md](docker/DOCKER_SETUP_CN.md)**

该文档包含：
- ✅ 前置要求和安装
- ✅ 详细使用指南
- ✅ 常见问题解决
- ✅ 性能优化建议
- ✅ 安全配置

## ⚡ 快速参考

```bash
# 从根目录启动
./docker_start.sh

# 或使用 Docker Compose
cd docker && docker-compose up -d

# 进入容器
docker exec -it asap_container bash

# 停止容器
cd docker && docker-compose down
```

## 🎯 容器环境

- **hvgym** (Python 3.8) - 训练和评估
  - PyTorch 1.13.1 + CUDA 11.7
  - Isaac Gym 支持
  - GPU Pipeline 自动启用

- **asap_deploy** (Python 3.10) - Sim2Sim/Sim2Real
  - ROS2 Humble
  - MuJoCo + ONNX Runtime

## 💡 为什么使用 Docker？

| 优势 | 说明 |
|------|------|
| 🔒 环境隔离 | 避免依赖冲突 |
| 📦 一键部署 | 自动配置所有依赖 |
| 🔄 可重现性 | 保证环境一致 |
| 🚀 快速开始 | 无需手动配置 |
| ✅ GPU 支持 | 完整的 CUDA 支持 |

---

**开始使用:** `./docker_start.sh`

**获取帮助:** 查看 [docker/DOCKER_SETUP_CN.md](docker/DOCKER_SETUP_CN.md)

