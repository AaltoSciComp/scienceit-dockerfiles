FROM nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04

# Install tzdata so that apt won't fail on it
RUN export DEBIAN_FRONTEND=noninteractive; \
    export DEBCONF_NONINTERACTIVE_SEEN=true; \
    echo 'tzdata tzdata/Areas select Europe' | debconf-set-selections; \
    echo 'tzdata tzdata/Zones/Etc select Helsinki' | debconf-set-selections; \
    apt-get update -qqy \
 && apt-get install -qqy --no-install-recommends \
        tzdata \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Update all system packages
RUN apt-get update && apt-get dist-upgrade -y \
    && rm -rf /var/lib/apt/lists/*

# Install essential software for building
RUN apt-get update && apt-get install -y --no-install-recommends \
        apt-transport-https \
        build-essential \
        ca-certificates \
        git \
        gnupg \
        libboost-all-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        libhdf5-serial-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopenblas-dev \
        libopencv-dev \
        libprotobuf-dev \
        libsnappy-dev \
        protobuf-compiler \
        python-dev \
        python-numpy \
        python-pip \
        python-scipy \
        python-setuptools \
        software-properties-common \
        wget \
    && rm -rf /var/lib/apt/lists/*

# Install newer version of cmake for kitware's repository, required by caffe
RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | apt-key add - \
   && apt-add-repository 'deb https://apt.kitware.com/ubuntu/ bionic main' \
   && apt-get update && apt-get install -y --no-install-recommends \
   cmake \
   && rm -rf /var/lib/apt/lists/*

ENV CAFFE_ROOT=/opt/caffe
WORKDIR $CAFFE_ROOT

ARG CAFFE_VERSION=1.0

# Clone caffe and create build directory
RUN git clone -b ${CAFFE_VERSION} --depth 1 https://github.com/BVLC/caffe.git . && \
    pip install --upgrade pip && \
    cd python && for req in $(cat requirements.txt) pydot; do pip install $req; done && cd .. && \
    mkdir build

# Change CUDA build architectures to support newer model cards

ARG CUDA_ARCHS="60 61 70 75"

RUN sed -i "s/20 21(20) 30 35 50 60 61/${CUDA_ARCHS}/g" /opt/caffe/cmake/Cuda.cmake

# Build caffe
RUN cd $CAFFE_ROOT/build && \
    cmake -DUSE_CUDNN=1 -DUSE_NCCL=1 -DBLAS=open .. && \
    make -j"$(nproc)"

# Add caffe to environment
ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

# Fix caffe headers with symlink
RUN ln -s /opt/caffe/build/include/caffe/proto /opt/caffe/include/caffe

# Install openpose dependencies
RUN apt-get update -y && apt-get --assume-yes install \
    build-essential \
    cmake \
    doxygen \
    libboost-all-dev \
    libeigen3-dev \
    libgflags-dev \
    libgoogle-glog-dev \
    libhdf5-serial-dev \
    libleveldb-dev \
    liblmdb-dev \
    libopenblas-dev \
    libopencv-dev \
    libprotobuf-dev \
    libsnappy-dev \
    libtiff5-dev \
    libviennacl-dev \
    protobuf-compiler \
    python-dev \
    python-opencv \
    python-pip \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# Clone openpose
RUN cd /opt && \
    wget -O openpose.zip https://github.com/CMU-Perceptual-Computing-Lab/openpose/archive/v1.5.1.zip && \
    unzip openpose.zip && \
    rm -f openpose.zip && \
    mv openpose-1.5.1 openpose

ENV OPENPOSE_ROOT /opt/openpose

WORKDIR /opt/openpose

# Clone pybind11 which is a requirement for openpose
RUN git clone https://github.com/pybind/pybind11.git $OPENPOSE_ROOT/3rdparty/pybind11

# Build openpose
RUN cd $OPENPOSE_ROOT && \
    mkdir -p build && cd build && \
    cmake \
      -DCMAKE_BUILD_TYPE="Release" \
      -DBUILD_CAFFE=OFF \
      -DBUILD_EXAMPLES=ON \
      -DBUILD_DOCS=ON \
      -DBUILD_SHARED_LIBS=ON \
      -DDOWNLOAD_BODY_25_MODEL=ON \
      -DDOWNLOAD_BODY_COCO_MODEL=ON \
      -DDOWNLOAD_BODY_MPI_MODEL=ON \
      -DDOWNLOAD_HAND_MODEL=ON \
      -DDOWNLOAD_FACE_MODEL=ON \
      -DWITH_3D_RENDERER:BOOL=OFF \
      -DCaffe_INCLUDE_DIRS="/opt/caffe/include" \
      -DCaffe_LIBS="/opt/caffe/build/lib/libcaffe.so" \
      -DBUILD_PYTHON=ON ../ && \
    make all -j"$(nproc)"

RUN cd $OPENPOSE_ROOT/build && \
  make install && \
  cp examples/openpose/openpose.bin /usr/local/bin/openpose

ENV PYTHONPATH="${PYTHONPATH}:/usr/local/python"
