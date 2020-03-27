FROM nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04

RUN export DEBIAN_FRONTEND=noninteractive; \
    export DEBCONF_NONINTERACTIVE_SEEN=true; \
    echo 'tzdata tzdata/Areas select Europe' | debconf-set-selections; \
    echo 'tzdata tzdata/Zones/Etc select Helsinki' | debconf-set-selections; \
    apt-get update -qqy \
 && apt-get install -qqy --no-install-recommends \
        tzdata \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get dist-upgrade -y \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
        apt-transport-https \
        build-essential \
        ca-certificates \
        git \
        gnupg \
        libopenblas-dev \
        libboost-all-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        libhdf5-serial-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopencv-dev \
        libprotobuf-dev \
        libsnappy-dev \
        protobuf-compiler \
        python-dev \
        python-numpy \
        python-pip \
        python-setuptools \
        python-scipy \
        software-properties-common \
        wget \
    && rm -rf /var/lib/apt/lists/*

RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | apt-key add - \
   && apt-add-repository 'deb https://apt.kitware.com/ubuntu/ bionic main' \
   && apt-get update && apt-get install -y --no-install-recommends \
   cmake \
   && rm -rf /var/lib/apt/lists/*

ENV CAFFE_ROOT=/opt/caffe
WORKDIR $CAFFE_ROOT

# FIXME: use ARG instead of ENV once DockerHub supports this
# https://github.com/docker/hub-feedback/issues/460
ENV CLONE_TAG=1.0

RUN git clone -b ${CLONE_TAG} --depth 1 https://github.com/BVLC/caffe.git . && \
    pip install --upgrade pip && \
    cd python && for req in $(cat requirements.txt) pydot; do pip install $req; done && cd .. && \
    #git clone https://github.com/NVIDIA/nccl.git && cd nccl && make -j install && cd .. && rm -rf nccl && \
    mkdir build

# Cuda 10 and above require newer cmake, install it from Kitware's repo

RUN sed -i "s/20 21(20) 30 35 50 60 61/60 61 70 75/g" /opt/caffe/cmake/Cuda.cmake

RUN cd $CAFFE_ROOT/build && \
    cmake -DUSE_CUDNN=1 -DUSE_NCCL=1 -DBLAS=open .. && \
    make -j"$(nproc)"

ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

WORKDIR /workspace

RUN apt-get update -y && apt-get --assume-yes install \
    build-essential unzip \
    # General dependencies
    libopenblas-dev libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev protobuf-compiler \
    libboost-all-dev \
    # Remaining dependencies
    libgflags-dev libgoogle-glog-dev liblmdb-dev \
    # Python libs
    libopencv-dev python-opencv python-pip python-dev \
    cmake \
    libeigen3-dev libviennacl-dev \
    doxygen

RUN cd /opt && \
    wget -O openpose.zip https://github.com/CMU-Perceptual-Computing-Lab/openpose/archive/v1.5.1.zip && \
    unzip openpose.zip && \
    rm -f openpose.zip && \
    mv openpose-1.5.1 openpose-build

ENV OPENPOSE_ROOT /opt/openpose-build

RUN git clone https://github.com/pybind/pybind11.git /opt/openpose-build/3rdparty/pybind11

RUN apt-get update -y && apt-get --assume-yes install \
    libtiff5-dev

RUN ln -s /opt/caffe/build/include/caffe/proto /opt/caffe/include/caffe && \
    cd /opt/openpose-build && \
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

RUN cd /opt/openpose-build/build && \
  make install && \
  cp examples/openpose/openpose.bin /usr/local/bin/openpose.bin
