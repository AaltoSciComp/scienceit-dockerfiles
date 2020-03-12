FROM bvlc/caffe:gpu

RUN apt-get update -y && apt-get --assume-yes install \
    build-essential unzip \
    # General dependencies
    libatlas-base-dev libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev protobuf-compiler \
    libboost-all-dev \
    # Remaining dependencies
    libgflags-dev libgoogle-glog-dev liblmdb-dev \
    # Python libs
    libopencv-dev python-opencv python-pip python-dev \
    cmake \
    libeigen3-dev libviennacl-dev \
    libnccl-dev \
    doxygen

RUN cd /opt && \
    wget -O openpose.zip https://github.com/CMU-Perceptual-Computing-Lab/openpose/archive/v1.5.1.zip && \
    unzip openpose.zip && \
    rm -f openpose.zip && \
    mv openpose-1.5.1 openpose-build

ENV OPENPOSE_ROOT /opt/openpose-build

RUN git clone https://github.com/pybind/pybind11.git /opt/openpose-build/3rdparty/pybind11

RUN cd /opt/openpose-build && \
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
      -DWITH_3D_RENDERER:BOOL=OFF \
      -DCaffe_INCLUDE_DIRS="/opt/caffe/include" \
      -DCaffe_LIBS="/opt/caffe/build/lib/libcaffe.so" \
      -DBUILD_PYTHON=ON ../ && \
    make all -j"$(nproc)"

