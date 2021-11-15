FROM ubuntu:20.04

ARG DEBIAN_FRONTEND="noninteractive"
ENV TZ=Europe/Helsinki

RUN apt-get update && apt-get install -y \
    curl                                 \
    libgomp1                             \
    qt5-default                          \
  && rm -rf /var/lib/apt/lists/*

ARG MESHLAB_VERSION=2021.10

RUN cd /opt && \
    curl -o MeshLab{MESHLAB_VERSION}-linux.tar.gz -L    \
        https://github.com/cnr-isti-vclab/meshlab/releases/download/Meshlab-${MESHLAB_VERSION}/MeshLab${MESHLAB_VERSION}-linux.tar.gz && \
    tar xf MeshLab{MESHLAB_VERSION}-linux.tar.gz && \
    rm MeshLab{MESHLAB_VERSION}-linux.tar.gz && \
    mv usr meshlab

ENV PATH="/opt/meshlab/bin:${PATH}"
