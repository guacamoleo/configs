# Disable apt-key parse waring
ARG APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=1
ARG MUID=1000
ARG MGID=1000
ARG USERNAME=dtanner

#FROM ghcr.io/rocm/therock_pytorch_dev_ubuntu_24_04_gfx950:main
#FROM rocm/pytorch:rocm6.4_ubuntu22.04_py3.10_pytorch_release_2.6.0
FROM ubuntu:24.04

# Set environment variables to avoid interaction during installation
ENV DEBIAN_FRONTEND=noninteractive


# Basic development environment
RUN apt-get update && apt-get install -y \
  software-properties-common \
  curl wget openssh-client git tmux less \
  build-essential cmake ninja-build \
  clang lld ccache \
  python3 python3-pip \
  pkg-config fasd \
  sudo numactl

RUN add-apt-repository -y ppa:neovim-ppa/stable && \
  apt-get update && apt-get install -y vim neovim vim-youcompleteme

# Update and install build tools mi450
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    ninja-build \
    cmake \
    git \
    curl \
    libcurl4-openssl-dev \
    rsync \
    ssh \
    wget \
    ca-certificates \
    python3-pip \
    less \
    vim \
    python3.12-dev \
    libzstd-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Ccache settings
ENV CCACHE_DIR=/ccache
RUN ccache --max-size=15G

# Create mapping directories and chown before switching user
RUN mkdir -p /ccache && chown ${DOCKER_USERID}:${DOCKER_GROUPID} /ccache && \
  mkdir -p /data && chown ${DOCKER_USERID}:${DOCKER_GROUPID} /data

# Custom dtanner steps.
RUN git clone https://github.com/guacamoleo/configs.git repos/guacamoleo_configs
RUN cp repos/guacamoleo_configs/_bashrc ~/.bashrc
RUN cp repos/guacamoleo_configs/_vimrc ~/.vimrc
RUN cp repos/guacamoleo_configs/_gitconfig ~/.gitconfig

# mi450 ffm steps
# get the latest build from http://rocm-ci.amd.com/job/compute-rocm-npi-mi450/lastSuccessfulBuild/
RUN wget https://artifactory-cdn.amd.com/artifactory/list/amdgpu-deb/amdgpu-install-internal_7.1-24.04-1_all.deb
RUN apt-get install ./amdgpu-install-internal_7.1-24.04-1_all.deb
#RUN amdgpu-repo --amdgpu-build=2204668 --rocm-build=compute-rocm-npi-mi450/544
RUN amdgpu-repo --amdgpu-build=2219470 --rocm-build=compute-rocm-npi-mi450/605
RUN apt -y install rocm-llvm rocm-llvm-dev rocm-device-libs rocprofiler-register comgr rocm-hip-runtime-dev rocblas rocblas-dev hipblas-common-dev hipblaslt hipblaslt-dev hipblas

RUN wget --no-check-certificate https://confluence.amd.com/download/attachments/1035169166/AMD_CA.crt

RUN pip config set global.break-system-packages true

RUN apt-get remove -y python3-distro && pip install conan einops

RUN echo "export PATH=~/.local/bin:/opt/rocm/bin:$PATH" >> ~/.bashrc
RUN echo "export NODE_EXTRA_CA_CERTS=~/AMD_CA.crt" >> ~/.bashrc
# end mi450 ffm steps

ARG MUID
ARG MGID
ARG USERNAME

RUN addgroup --gid ${MGID} ${USERNAME}
RUN useradd -d /home/${USERNAME} -g ${MGID} --no-create-home -u ${MUID} --shell /usr/bin/bash ${USERNAME}
RUN adduser ${USERNAME} sudo
RUN adduser ${USERNAME} video
#RUN adduser ${USERNAME} render
# The /var is only needed on lockhart
RUN mkdir /home/${USERNAME} /var/${USERNAME}
RUN chown ${USERNAME}:${USERNAME} /home/${USERNAME} /var/${USERNAME}
#RUN chown -R ${USERNAME}:${USERNAME} /opt/conda


# Make sudo work without a password
RUN sed -i~ -e 's/%sudo\tALL=(ALL:ALL) ALL/%sudo\tALL=(ALL:ALL) NOPASSWD:ALL/g' /etc/sudoers

USER ${USERNAME}
WORKDIR /home/${USERNAME}
ENV HOME=/home/${USERNAME}

# Keep container alive
CMD tmux new-session -d && tail -f /dev/null
