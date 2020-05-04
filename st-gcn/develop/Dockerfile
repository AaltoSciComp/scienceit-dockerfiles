FROM ubuntu:18.04

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
        apt-transport-https            \
        build-essential                \
        ca-certificates                \
        curl                           \
        doxygen                        \
        git                            \
        gnupg                          \
        libgl1-mesa-glx                \
        software-properties-common     \
        unzip                          \
        wget                           \
    && rm -rf /var/lib/apt/lists/*

# Install Miniconda
RUN cd /tmp && \
    curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    bash Miniconda3-latest-Linux-x86_64.sh -b -p /opt/conda && \
    rm Miniconda3-latest-Linux-x86_64.sh

ENV PATH="/opt/conda/bin:${PATH}"

# Install packages that OpenPose requires
RUN conda install -n base -y \
        boost                \
        caffe-gpu            \
        cmake                \
        Cython               \
        gflags               \
        glog                 \
        h5py                 \
        imageio              \
        ipython              \
        leveldb              \
        lmdb                 \
        matplotlib           \
        networkx             \
        nose                 \
        numpy                \
        opencv               \
        pandas               \
        Pillow               \
        protobuf==3.6.0      \
        pydot                \
        python-dateutil      \
        python-gflags        \
        pyyaml               \
        scikit-image         \
        scipy                \
        six

RUN conda install -n base -y -c conda-forge \
        cudatoolkit-dev      \
        mkl-devel            \
        scikit-video

RUN conda install -y -c pytorch \
        cudnn \
        pytorch \
        torchvision

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
      -DCMAKE_PREFIX_PATH="/opt/conda" \
      -DBUILD_CAFFE=OFF \
      -DBUILD_EXAMPLES=ON \
      -DBUILD_DOCS=ON \
      -DBUILD_SHARED_LIBS=ON \
      -DUSE_MKL=ON \
      -DDOWNLOAD_BODY_25_MODEL=ON \
      -DDOWNLOAD_BODY_COCO_MODEL=ON \
      -DDOWNLOAD_BODY_MPI_MODEL=ON \
      -DDOWNLOAD_HAND_MODEL=ON \
      -DDOWNLOAD_FACE_MODEL=ON \
      -DWITH_3D_RENDERER:BOOL=OFF \
      -DCaffe_INCLUDE_DIRS="/opt/conda/include" \
      -DCaffe_LIBS="/opt/conda/lib/libcaffe.so" \
      -DBUILD_PYTHON=ON ../ && \
    make all -j"$(nproc)"

# Install openpose
RUN cd $OPENPOSE_ROOT/build && \
  make install && \
  cp examples/openpose/openpose.bin /usr/local/bin/openpose

ENV PYTHONPATH="${PYTHONPATH}:/usr/local/python"

# Install st-gcn
RUN cd /opt && \
    git clone https://github.com/yysijie/st-gcn.git

RUN cd /opt/st-gcn/torchlight && \
    python setup.py install

RUN cd /opt/st-gcn && \
    bash tools/get_models.sh
