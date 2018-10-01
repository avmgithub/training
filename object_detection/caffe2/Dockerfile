FROM nvidia/cuda-ppc64le:9.2-cudnn7-devel-ubuntu16.04

ENV PATH=/opt/miniconda/bin:$PATH
ENV BUILD_ENVIRONMENT=pytorch-ppc64le-cuda9.2-cudnn7-py2

# Caffe2 depends
RUN apt-get update && \ 
    apt-get upgrade --no-install-recommends -y && \
    apt autoremove -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN apt-get update && \
      apt-get install -y --no-install-recommends \
      build-essential \
      cmake \
      git \
      libgoogle-glog-dev \
      libgtest-dev \
      libiomp-dev \
      libleveldb-dev \
      liblmdb-dev \
      libopencv-dev \
      libopenmpi-dev \
      libsnappy-dev \
      libprotobuf-dev \
      openmpi-bin \
      openmpi-doc \
      protobuf-compiler \
      python-dev \
      python-pip \
      curl \
      qt-sdk \
      gfortran \
      libopenblas-dev
RUN apt-get install -y --no-install-recommends libgflags-dev

# Install conda Python2 
RUN curl https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-ppc64le.sh \
    -o /root/miniconda.sh && \
    bash /root/miniconda.sh -b -p /opt/miniconda && \
    export PATH=/opt/miniconda/bin:$PATH && \
    conda install -y pyyaml setuptools Cython mock typing && \
    pip install numpy scipy==1.0.0 matplotlib future protobuf && \
    conda clean -ya

RUN export PATH=/opt/miniconda/bin:$PATH &&  cd /root && \
    git clone https://github.com/skvark/opencv-python.git && \
    cd opencv-python && git checkout 18 && \
    python setup.py install

RUN cd /root && rm -rf opencv-python

# Detectron depends
#RUN conda install \
#     matplotlib opencv-python>=3.2 setuptools Cython mock scipy==1.0.0

# Hidden dependancies
RUN pip install networkx # enum

# Build and install caffe2
SHELL ["/bin/bash", "-c"]
WORKDIR /packages
RUN git clone https://github.com/pytorch/pytorch
WORKDIR /packages/pytorch
RUN git checkout e43ffb014806757ad91fc70c9ec5c5f44a4eaa2e
RUN git submodule update --init --recursive # ; exit 0

# slightly edit .jenkins/caffe2/build.sh so it will run correctly
RUN sed -i s/^SCCACHE/#SCCACHE/g /packages/pytorch/.jenkins/caffe2/build.sh && \
    sed -i s/CMAKE_ARGS=\(\)/#CMAKE_ARGS=\(\)/ /packages/pytorch/.jenkins/caffe2/build.sh && \
    CMAKE_ARGS="-DUSE_MPI=OFF -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda" /packages/pytorch/.jenkins/caffe2/build.sh

# Build and install the COCO API
WORKDIR /packages
RUN git clone https://github.com/cocodataset/cocoapi.git
WORKDIR /packages/cocoapi/PythonAPI
RUN make install

# Build detectron
WORKDIR /packages
RUN git clone https://github.com/ddkang/Detectron.git detectron
WORKDIR /packages/detectron
RUN git checkout benchmarking
WORKDIR /packages/detectron/lib
RUN make
WORKDIR /packages/detectron/lib/datasets/data

# Setup to run mlperf
# EXAMPLE:
# ls /home/freddie/projects/mlperf/training.AVM/object_detection/data/detectron/lib/data/coco
# annotations  annotations_trainval2014.zip  coco_annotations_minival.tgz  coco_train2014  coco_val2014  train2014.zip  val2014.zip
# NOTE: the above ls listing is where I downloaded the data and traning sets from the scripts download_dataset.sh
# sudo docker run -it -rm -e NVIDIA_VISIBLE_DEVICES=0,1  -v /home/freddie/projects/mlperf/training.AVM/object_detection/data/detectron/lib/data/coco:/packages/detectron/lib/datasets/data/coco 412fc0eaaa1b /bin/bash

# export LD_LIBRARY_PATH=/usr/local/caffe2/lib:$LD_LIBRARY_PATH
# export PYTHONPATH=/usr/local/caffe2/lib/python2.7/site-packages
# source activate py2k

# This is needed so that caffe2 finds detectron library
RUN mkdir /usr/local/caffe2/lib/python2.7/site-packages/lib/ && \
    cp /packages/pytorch/build_caffe2/lib/libcaffe2_detectron_ops_gpu.so /usr/local/caffe2/lib/python2.7/site-packages/lib/libcaffe2_detectron_ops_gpu.so

ENV LD_LIBRARY_PATH=/usr/local/caffe2/lib:$LD_LIBRARY_PATH

# Set PYTHONPATH so caffe2 can be found
ENV PYTHONPATH=/usr/local/caffe2/lib/python2.7/site-packages

WORKDIR /packages/detectron 
