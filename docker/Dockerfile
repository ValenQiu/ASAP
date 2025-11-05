# ASAP Docker Environment
# Based on NVIDIA CUDA + Ubuntu 20.04
FROM nvidia/cuda:11.7.1-cudnn8-devel-ubuntu20.04

# Prevent interactive prompts during installation
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai

# Install basic dependencies
RUN apt-get update && apt-get install -y \
    wget \
    curl \
    git \
    vim \
    build-essential \
    cmake \
    libssl-dev \
    libgl1-mesa-glx \
    libglib2.0-0 \
    libsm6 \
    libxext6 \
    libxrender-dev \
    libgomp1 \
    python3-pip \
    python3-dev \
    vulkan-utils \
    mesa-vulkan-drivers \
    && rm -rf /var/lib/apt/lists/*

# Install Miniconda
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/miniconda.sh && \
    bash /tmp/miniconda.sh -b -p /opt/conda && \
    rm /tmp/miniconda.sh

ENV PATH="/opt/conda/bin:${PATH}"

# Configure conda to use conda-forge and accept TOS
RUN conda config --set channel_priority flexible && \
    conda config --add channels conda-forge && \
    conda config --set solver libmamba && \
    echo "yes" | conda tos accept 2>/dev/null || true

# Create conda environment for training (hvgym)
RUN conda create -n hvgym python=3.8 -y && \
    echo "source activate hvgym" > ~/.bashrc

# Activate environment and install PyTorch
SHELL ["/bin/bash", "-c"]
RUN source activate hvgym && \
    pip install torch==1.13.1+cu117 torchvision==0.14.1+cu117 --extra-index-url https://download.pytorch.org/whl/cu117

# Install common Python packages for hvgym
RUN source activate hvgym && \
    pip install numpy scipy matplotlib \
    hydra-core omegaconf \
    tensorboard wandb \
    loguru termcolor \
    joblib pyyaml \
    mujoco \
    keyboard

# Set Vulkan environment variable for Isaac Gym GPU pipeline
ENV VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/nvidia_icd.json

# Create deployment environment (asap_deploy) with ROS2
RUN conda create -n asap_deploy python=3.10 -y && \
    source activate asap_deploy && \
    conda config --env --add channels conda-forge && \
    conda config --env --add channels robostack-staging && \
    conda config --env --remove channels defaults || true

# Install ROS2 Humble in asap_deploy environment
RUN source activate asap_deploy && \
    conda install ros-humble-desktop -y

# Install deployment packages
RUN source activate asap_deploy && \
    pip install mujoco onnxruntime numpy scipy

# Set up conda activation scripts for hvgym
RUN mkdir -p /opt/conda/envs/hvgym/etc/conda/activate.d && \
    mkdir -p /opt/conda/envs/hvgym/etc/conda/deactivate.d && \
    echo '#!/bin/sh' > /opt/conda/envs/hvgym/etc/conda/activate.d/env_vars.sh && \
    echo 'export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH' >> /opt/conda/envs/hvgym/etc/conda/activate.d/env_vars.sh && \
    echo 'export VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/nvidia_icd.json' >> /opt/conda/envs/hvgym/etc/conda/activate.d/env_vars.sh && \
    chmod +x /opt/conda/envs/hvgym/etc/conda/activate.d/env_vars.sh

# Set working directory
WORKDIR /workspace

# Copy ASAP codebase (will be mounted as volume in practice)
# COPY . /workspace/ASAP

# Set default conda environment
ENV CONDA_DEFAULT_ENV=hvgym

# Entry point
CMD ["/bin/bash"]

