# Copyright (c) 2021 Intel Corporation
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

ARG BASE=openvino/ubuntu20_data_dev:2021.4.2
ARG PIPELINE_ZOO_PLATFORM=DEFAULT
ARG REGISTRY=

FROM ${REGISTRY}${BASE} as media-analytics-pipeline-zoo
ARG BASE=openvino/ubuntu20_data_dev:2021.3_vaapi_fix
ARG PIPELINE_ZOO_PLATFORM=DEFAULT

ENV PIPELINE_ZOO_PLATFORM=$PIPELINE_ZOO_PLATFORM
ENV PIPELINE_ZOO_BASE_IMAGE=${REGISTRY}$BASE
ENV INTEL_OPENVINO_DIR=/opt/intel/openvino

USER root

# Dependencies for OpenVINO
RUN if [ -f /opt/intel/openvino/install_dependencies/install_openvino_dependencies.sh ]; then \
       /opt/intel/openvino/install_dependencies/install_openvino_dependencies.sh -y ;\
    fi

RUN if [ -f /opt/intel/openvino/install_dependencies/install_NEO_OCL_driver.sh ]; then \
       /opt/intel/openvino/install_dependencies/install_NEO_OCL_driver.sh -y ; exit 0; \ 
    fi    

RUN if [ -f /opt/intel/openvino/install_dependencies/install_NCS_udev_rules.sh ]; then \
       apt-get install -y -q --no-install-recommends sudo; \ 
       /opt/intel/openvino/install_dependencies/install_NCS_udev_rules.sh -y ;\
    fi    


# Dependencies installed via apt-get
RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
    apt-get install -y -q --no-install-recommends \
    python3-setuptools \
    python3-pip \
    graphviz-dev \
    pciutils \
    dmidecode \
    cmake \
    bash-completion \
    libyaml-cpp-dev \
    nlohmann-json3-dev \
    numactl \
    libjemalloc-dev \
    python3 \
    lsb-release \
    vim && \
    rm -rf /var/lib/apt/lists/*

ENV nlohmann_json_DIR=/usr/lib/cmake
ENV LD_PRELOAD=libjemalloc.so

# bash completion
RUN printf "if [ -f /etc/bash_completion ] && ! shopt -oq posix; then \n\
    . /etc/bash_completion \n\
fi\n" >> ~/.bashrc

# pipebench
COPY ./pipebench/requirements.txt /
RUN pip3 install  --no-cache-dir -r /requirements.txt
RUN rm -f /requirements.txt
RUN ln -s /home/pipeline-zoo/tools/pipebench/pipebench/__main__.py /usr/local/bin/pipebench
ENV PYTHONPATH=$PYTHONPATH:/home/pipeline-zoo/tools/pipebench
# Install bash-completion
RUN printf 'eval "$(shtab --shell=bash pipebench.arguments._get_parser_shtab)"' \
  | tee "$(pkg-config --variable=completionsdir bash-completion)"/pipebench

# systeminfo
COPY ./systeminfo/requirements.txt /
RUN pip3 install  --no-cache-dir -r /requirements.txt
RUN rm -f /requirements.txt

# uploader
COPY ./systeminfo/requirements.txt ./uploader/requirements.tx[t] /
RUN pip3 install  --no-cache-dir -r /requirements.txt
RUN rm -f /requirements.txt

# downloader
COPY ./downloader/requirements.txt /
RUN pip3 install  --no-cache-dir -r/requirements.txt
RUN rm -f /requirements.txt

# tests
COPY ./systeminfo/requirements.txt ./tests/requirements.tx[t] /
RUN pip3 install  --no-cache-dir -r /requirements.txt
RUN rm -f /requirements.txt

# Patches
COPY ./docker/patches/video_frame.py /opt/intel/openvino/data_processing/dl_streamer/python/gstgva/video_frame.py

# model zoo
ENV OPEN_MODEL_ZOO_ROOT=/opt/intel/openvino/deployment_tools/open_model_zoo

# model proc
ENV DLSTREAMER_MODEL_PROC_ROOT=/opt/intel/openvino/data_processing/dl_streamer/samples/model_proc

# XDG_RUNTIME_DIR
RUN mkdir -p /home/.xdg_runtime_dir
ENV XDG_RUNTIME_DIR=/home/.xdg_runtime_dir

#Open CL Cache
RUN mkdir -p $SOURCE_DIR/workspace/.cl-cache
ENV cl_cache_dir=/home/pipeline-zoo/workspace/.cl-cache

# Bash History

ENV HISTFILE=/home/pipeline-zoo/workspace/.bash_history

# work dir
WORKDIR /home/pipeline-zoo/workspace
