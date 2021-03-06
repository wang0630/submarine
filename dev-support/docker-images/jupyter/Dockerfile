# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM ubuntu:18.04

ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_PREFIX="/"
ARG NB_PORT=8888

USER root
ENV NB_USER $NB_USER
ENV NB_UID $NB_UID
ENV NB_GID $NB_GID
ENV NB_PREFIX $NB_PREFIX
ENV NB_PORT $NB_PORT
ENV CONDA_DIR=/opt/conda
ENV PATH=$CONDA_DIR/bin:$PATH
ENV HOME=/home/$NB_USER

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    apt-transport-https \
    build-essential \
    curl \
    wget \
    vim \
    bzip2 \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
    run-one && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN echo "$LOG_TAG Set locale" && \
    echo "LC_ALL=en_US.UTF-8" >> /etc/environment && \
    echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    echo "LANG=en_US.UTF-8" > /etc/locale.conf && \
    locale-gen en_US.UTF-8

ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

# Create NB_USER user with UID=1000 and in the 'users' group
RUN useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    chown -R ${NB_USER}:users /usr/local/bin && \
    mkdir -p $HOME && \
    chown -R ${NB_USER}:users ${HOME}


# Add Tini
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN mv /tini /usr/local/bin/tini && chmod +x /usr/local/bin/tini

# Install conda
ARG PYTHON_VERSION=default
ENV MINICONDA_VERSION=4.8.3 \
    MINICONDA_MD5=751786b92c00b1aeae3f017b781018df \
    CONDA_VERSION=4.8.3

WORKDIR /tmp
RUN wget --quiet https://repo.continuum.io/miniconda/Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "${MINICONDA_MD5} *Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh" | md5sum -c - && \
    /bin/bash Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-py37_${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "conda ${CONDA_VERSION}" >> $CONDA_DIR/conda-meta/pinned && \
    conda config --system --prepend channels conda-forge && \
    conda config --system --set auto_update_conda false && \
    conda config --system --set show_channel_urls true && \
    conda config --system --set channel_priority strict && \
    if [ ! $PYTHON_VERSION = 'default' ]; then conda install --yes python=$PYTHON_VERSION; fi && \
    conda list python | grep '^python ' | tr -s ' ' | cut -d '.' -f 1,2 | sed 's/$/.*/' >> $CONDA_DIR/conda-meta/pinned && \
    conda install --quiet --yes conda && \
    conda install --quiet --yes pip && \
    conda clean --all -f -y && \
    rm -rf /home/$NB_USER/.cache/yarn

# Install jupyter
RUN conda install --quiet --yes notebook=6.0.3 && \
    conda clean --all -f -y

USER root
RUN mkdir -p $CONDA_DIR && \
    chown -R ${NB_USER}:users $CONDA_DIR

USER $NB_USER
EXPOSE $NB_PORT
ENTRYPOINT ["tini", "-g", "--"]
WORKDIR ${HOME}
CMD ["start-notebook.sh"]
COPY start-notebook.sh /usr/local/bin
