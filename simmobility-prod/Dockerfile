FROM ubuntu:20.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
  && apt-get upgrade -y \
  && apt-get install -y \
    binutils \
    cmake \
    doxygen \
    g++ \
    gdal-bin \
    git \
    libboost-all-dev \
    libcppunit-dev \
    libglpk-dev \
    libjsoncpp-dev \
    liblua5.2-dev \
    libpq-dev \
    libproj-dev \
    libsoci-dev \
    libssl-dev \
    libssl-dev \
    libtinyxml-dev \
    libxerces-c-dev \
    postgresql \
    python3 \
    python3-crypto \
    python3-matplotlib \
    python3-numpy \
    python3-pandas \
    python3-pip \
    python3-psycopg2 \
    qgit \
    xsdcxx \
  && rm -rf /var/lib/apt/lists/*

RUN cd /opt \
  && git clone https://github.com/smart-fm/simmobility-prod.git

ARG BUILD_TIMESTAMP=$(date -Imin)

RUN mkdir /opt/simmobility-prod/dev/Basic/Release \
  && cd /opt/simmobility-prod/dev/Basic/Release \
  && cmake ../ \
     -DCMAKE_BUILD_TYPE:STRING=Release -DBUILD_SHORT:BOOL=ON \
     -DBUILD_MEDIUM:BOOL=ON -DBUILD_LONG:BOOL=ON -DBUILD_TESTS:BOOL=OFF \
     -DBUILD_TESTS_LONG:BOOL=OFF 2>&1 | tee /opt/simmobility-prod/cmake.log \
  && make -j 6 all 2>&1 | tee /opt/simmobility-prod/make.log \
  && echo $BUILD_TIMESTAMP > /opt/simmobility-prod/build_timestamp.log

ENV PATH=$PATH:/opt/simmobility-prod/dev/Basic/Release

ENV LC_ALL=C
