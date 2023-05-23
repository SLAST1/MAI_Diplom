FROM nvidia/cudagl:11.3.1-base-ubuntu20.04

LABEL AUTHOR="IT-MAI"
LABEL TITLE="MAI FloodLib Docker"

# Enviorment variables
ENV DEBIAN_FRONTEND noninteractive
ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8
ENV PATH "$PATH:/bin/3.0/python/bin/"
ENV BLENDER_PATH "/bin/3.0"
ENV HW="GPU"
ENV DOCKER_RUN="True"

# Set timezone
ENV TZ=Europe/Moscow
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone


# Install dependencies
RUN apt-get update
RUN apt-get -y install apt-utils
RUN apt-get install -y --allow-downgrades --allow-change-held-packages --no-install-recommends \
    build-essential cmake curl vim wget ca-certificates iputils-ping net-tools \
    ethtool perl lsb-release iproute2 pciutils libnl-route-3-200 kmod libnuma1 lsof \
    libjpeg-dev libpng-dev libgnutls30 libopenexr-dev bzip2 python-all-dev zlib1g-dev libxmu-dev \ 
    libxi-dev libxxf86vm-dev libfontconfig1 libxrender1 libgl1-mesa-glx xz-utils git libtiff-dev \
    libsqlite3-dev automake libtool pkg-config libsqlite3-dev sqlite3 libpq-dev \
    libcurl4-gnutls-dev libproj-dev libxml2-dev libgeos-dev libnetcdf-dev \
    libpoppler-dev libspatialite-dev libhdf4-alt-dev libhdf5-serial-dev libopenjp2-7-dev ninja-build


# We create linux user uid 1000 and guid 1000
# In base ml-images it has username "user"
# In jupyter images username will be "jovyan" (we didn't add new user, just replace user->jovyan)
RUN useradd -u 1000 -m user --shell /bin/bash

# Making /home/jovyan folder it will be main mounted user storage (either in base and jupyter images)
# In ml-images owner will be shown as user
# In jupyter-images owner will be shown as jovayn
# uid and guid will be the same and
RUN mkdir -p /home/jovyan && chown user:user /home/jovyan 

ENV CONDA_DIR=/home/user/conda
ENV PATH $CONDA_DIR/bin:$PATH

USER user
ENV MINICONDA_VERSION py39_4.12.0
# WORKDIR doesn't respoect USER change
RUN mkdir -p /tmp/aispace/conda && cd /tmp/aispace/conda && \
    mkdir -p $CONDA_DIR && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    /bin/bash Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p ${CONDA_DIR} && \
    rm Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    conda config --system --prepend channels conda-forge && \
    conda config --system --set auto_update_conda false && \
    conda config --system --set show_channel_urls true && \
    conda update --all && \
    conda update conda && \
    conda clean --all -y && \
    rm -rf /tmp/aispace

USER root
# Download and install Blender
RUN wget https://mirror.clarkson.edu/blender/release/Blender3.0/blender-3.0.0-linux-x64.tar.xz \
	&& tar -xvf blender-3.0.0-linux-x64.tar.xz --strip-components=1 -C /bin \
	&& rm -rf blender-3.0.0-linux-x64.tar.xz \
	&& rm -rf blender-3.0.0-linux-x64

# Link python
RUN rm -rf ${BLENDER_PATH}/python \
    && ln -s ${CONDA_DIR} ${BLENDER_PATH}/python

USER root

ENV GDAL_VERSION="3.5.1"

RUN conda install -c conda-forge gdal==${GDAL_VERSION}
ENV PROJ_LIB=/home/user/conda/share/proj
RUN cd /etc && touch asound.conf
RUN echo 'pcm.!default { type plug slave.pcm "null" }' > /etc/asound.conf


ENV CUDA_VERSION="cu113"
ENV PYTORCH_VERSION="1.12.1+${CUDA_VERSION}"
ENV TORCHVISION_VERSION="0.13.1+${CUDA_VERSION}"
ENV TORCHAUDIO_VERSION="0.12.1"
ENV PYTORCH_WHEELS_URL=https://download.pytorch.org/whl/torch_stable.html
ENV MMCV_WHEELS_URL=https://download.openmmlab.com/mmcv/dist/${CUDA_VERSION}/torch1.11.0/index.html


USER user

RUN pip install --no-cache-dir --upgrade setuptools wheel pip
RUN pip install --no-cache-dir torch==${PYTORCH_VERSION} torchvision==${TORCHVISION_VERSION} torchaudio==${TORCHAUDIO_VERSION} -f ${PYTORCH_WHEELS_URL}

RUN pip install --no-cache scikit-learn matplotlib pandas openmesh==1.2.1 ipywidgets jupyterlab pytest



USER root
RUN chown -R user /home/jovyan
USER user

COPY --chown=user:user . /home/jovyan
RUN cd /home/jovyan && python setup.py install && pip install -e . --user

WORKDIR /home/jovyan

EXPOSE 8888

ENTRYPOINT jupyter lab \
    --notebook-dir=/home/jovyan \
    --ip=0.0.0.0 \
    --no-browser \
    --allow-root \
    --port=8888 \
    --NotebookApp.token='' \
    --NotebookApp.password='' \
    --FileContentsManager.delete_to_trash=True
