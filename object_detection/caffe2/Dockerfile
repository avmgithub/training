FROM nvidia/cuda-ppc64le:9.2-cudnn7-devel-ubuntu16.04

ENV PATH=/opt/miniconda/bin:$PATH
ENV BUILD_ENVIRONMENT=pytorch-ppc64le-cuda9.2-cudnn7-py2
#ENV PYTHONPATH=/usr/local/caffe2/lib/python2.7/site-packages:/opt/miniconda/envs/py2k/lib/python2.7/site-packages
#ENV CONDA_DEFAULT_ENV=py2k

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

# Install conda py3
RUN curl https://repo.continuum.io/miniconda/Miniconda3-4.5.4-Linux-ppc64le.sh \
    -o /root/miniconda.sh && \
    bash /root/miniconda.sh -b -p /opt/miniconda && \
    export PATH=/opt/miniconda/bin:$PATH
#    conda install -y pyyaml setuptools Cython mock && \
#    pip install numpy && \
#    pip install scipy==1.0.0 matplotlib && \
#    conda clean -ya

# Install conda py2
RUN export PATH=/opt/miniconda/bin:$PATH && \
    conda create -n py2k python=2 -y && \
    bash -c "source activate py2k" && \
    export CONDA_ROOT_PREFIX="/opt/miniconda/envs/py2k" && \
    conda install -y pyyaml setuptools Cython mock typing && \
    ${CONDA_ROOT_PREFIX}/bin/pip install numpy scipy==1.0.0 \
    matplotlib future protobuf && \
    conda clean -ya

#ENV PYTHONPATH=/usr/local/caffe2/lib/python2.7/site-packages:/opt/miniconda/envs/py2k/lib/python2.7/site-packages

#RUN pip install --upgrade pip
#RUN pip install \
#      future \
#      protobuf

RUN export PATH=/opt/miniconda/envs/py2k/bin:$PATH && cd /root && \
    git clone https://github.com/skvark/opencv-python.git && \
    cd opencv-python && git checkout 18 && \
    /opt/miniconda/envs/py2k/bin/python2 setup.py install

RUN cd /root && rm -rf opencv-python

# Detectron depends
#RUN conda install \
#     matplotlib opencv-python>=3.2 setuptools Cython mock scipy==1.0.0

# Hidden dependancies
# this needs to be done from py2k  env
RUN export CONDA_ROOT_PREFIX="/opt/miniconda/envs/py2k" && \
    ${CONDA_ROOT_PREFIX}/bin/pip install networkx # enum

# set LD_LIBRARY_PATH=/usr/local/caffe2/lib:$LD_LIBRARY_PATH
# set PYTHONPATH=/usr/local/caffe2/:/usr/local/caffe2/lib/python2.7/site-packages:/opt/miniconda/envs/py2k/lib/python2.7/site-packages
# Build and install caffe2
SHELL ["/bin/bash", "-c"]
WORKDIR /packages
RUN git clone https://github.com/pytorch/pytorch
WORKDIR /packages/pytorch
#RUN git checkout 3dd56c9580ad7ce6d20a4dd922aaac171acf582a
RUN git submodule update --init --recursive # ; exit 0
RUN source activate py2k && \
    sed -i s/^SCCACHE/#SCCACHE/g /packages/pytorch/.jenkins/caffe2/build.sh && \
    sed -i s/CMAKE_ARGS=\(\)/#CMAKE_ARGS=\(\)/ /packages/pytorch/.jenkins/caffe2/build.sh && \
    CMAKE_ARGS="-DUSE_MPI=OFF -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda" /packages/pytorch/.jenkins/caffe2/build.sh
# export BUILD_ENVIRONMENT=pytorch-ppc64le-cuda9.1-cudnn7-py3
#WORKDIR /packages/caffe2/third_party/aten
#RUN git pull origin master
#RUN git checkout fd0020fff6e9a42de8efa7d94589a9168bf721a6
#WORKDIR /packages/caffe2
#RUN mkdir build
#WORKDIR /packages/caffe2/build
#RUN cmake ..
#RUN make install -j16

# Build and install the COCO API
# this needs to be under py2k env
WORKDIR /packages
RUN git clone https://github.com/cocodataset/cocoapi.git
WORKDIR /packages/cocoapi/PythonAPI
RUN source activate py2k &&  make install

# Build detectron
WORKDIR /packages
RUN git clone https://github.com/ddkang/Detectron.git detectron
WORKDIR /packages/detectron
RUN git checkout benchmarking
WORKDIR /packages/detectron/lib
RUN source activate py2k && make
WORKDIR /packages/detectron/lib/datasets/data
# RUN ln -s /data coco/

# cp /packages/pytorch/build_caffe2/lib/libcaffe2_detectron_ops_gpu.so /usr/local/caffe2/lib/python2.7/site-packages/lib/libcaffe2_detectron_ops_gpu.so
WORKDIR /packages/detectron 
