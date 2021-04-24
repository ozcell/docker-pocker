######################################## MAIN ########################################
# install anaconda, gym, mujoco, pytorch
ARG UBUNTU_VERSION=16.04
FROM nvidia/cuda:9.0-base-ubuntu${UBUNTU_VERSION}

ARG DOCKERUSER=user
ARG DOCKERPASS=pass
ARG DOCKERID=1000
ARG RESOLUTION=1920x1080

# install sudo
RUN apt-get update --fix-missing && apt-get install -y sudo
# create sudo user with name ${USERNAME} and with password ${PASSWORD}
RUN useradd -m ${DOCKERUSER} -u ${DOCKERID} -s /bin/bash && echo ${DOCKERUSER}:${DOCKERPASS} | chpasswd && adduser ${DOCKERUSER} sudo
RUN chgrp -R ${DOCKERUSER} $HOME/ && chown -R ${DOCKERUSER} $HOME/

USER ${DOCKERUSER}
ENV HOME=/home/${DOCKERUSER} \  
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8
ENV PATH=$PATH:$HOME/conda/bin
ENV DOCKERUSER=${DOCKERUSER} \
    DOCKERPASS=${DOCKERPASS} \
    DOCKERID=${DOCKERID} \
    RESOLUTION=${RESOLUTION} 

WORKDIR $HOME

# install some common tools
RUN echo ${DOCKERPASS} | sudo -S apt-get update --fix-missing && \ 
    echo ${DOCKERPASS} | sudo -S apt-get install -y wget bzip2 ca-certificates \
    libglib2.0-0 libxext6 libsm6 libxrender1 git mercurial subversion nano

# install anaconda
RUN wget --quiet https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh -O ~/anaconda.sh && \
    /bin/bash ~/anaconda.sh -b -p $HOME/conda && \
    rm ~/anaconda.sh && \
    echo ${DOCKERPASS} | sudo -S ln -s $HOME/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". $HOME/conda/etc/profile.d/conda.sh" >> $HOME/.bashrc && \
    echo "conda activate base" >> $HOME/.bashrc

# install tini
RUN echo ${DOCKERPASS} | sudo -S apt-get install -y curl grep sed dpkg && \
    TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
    curl -L "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini_${TINI_VERSION}.deb" > tini.deb && \
    echo ${DOCKERPASS} | sudo -S dpkg -i tini.deb && \
    rm tini.deb && \
    echo ${DOCKERPASS} | sudo -S apt-get clean

# install OpenAI gym
# if 16.04
RUN echo ${DOCKERPASS} | sudo -S apt-get install -y python-pyglet python3-opengl zlib1g-dev libjpeg-dev patchelf \
        cmake swig libboost-all-dev libsdl2-dev libosmesa6-dev xvfb ffmpeg unzip libglew-dev

# if 18.04
#RUN apt-get install -y python3-dev zlib1g-dev libjpeg-dev cmake swig \
#    python-pyglet python3-opengl libboost-all-dev libsdl2-dev \
#    libosmesa6-dev patchelf ffmpeg xvfb unzip

#mujoco binaries and the licence and env variables
RUN wget --quiet https://www.roboti.us/download/mjpro150_linux.zip -O ./temp.zip && \
    unzip temp.zip -d $HOME/.mujoco && rm temp.zip
COPY ./mjkey.txt $HOME/.mujoco
RUN echo ${DOCKERPASS} | sudo -S chown -R ${DOCKERUSER} $HOME/.mujoco
ENV MUJOCO_PY_MJKEY_PATH=$HOME/.mujoco/mjkey.txt \
    MUJOCO_PY_MJPRO_PATH=$HOME/.mujoco/mjpro150/ \
    LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/.mujoco/mjpro150/bin \
    CPATH=$CPATH:$HOME/.mujoco/mjpro150/include \
    PATH=$PATH:$HOME/.mujoco/mjpro150/bin \
    LIBRARY_PATH=$LIBRARY_PATH:$HOME/.mujoco/mjpro150/bin \
    LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libGLEW.so

#install mujoco-py, gym, pytorch, torchvision
RUN pip --no-cache-dir install mujoco-py
RUN pip --no-cache-dir install 'gym[all]'
RUN pip --no-cache-dir install https://download.pytorch.org/whl/cu90/torch-0.4.1-cp36-cp36m-linux_x86_64.whl torchvision
RUN pip --no-cache-dir install tensorboardX
RUN pip --no-cache-dir install mpi4py

RUN echo 'for d in $HOME/git/*; do export PYTHONPATH="$PYTHONPATH:$d"; done' >> $HOME/.bashrc
######################################## MAIN ########################################

##################################### TENSORFLOW #####################################
# <---- if tensorflow is not needed this part could be commented up to 'here' label
EXPOSE 6006

#Pick up some TF dependencies
RUN echo ${DOCKERPASS} | sudo -S apt-get update && \ 
    echo ${DOCKERPASS} | sudo -S apt-get install -y --no-install-recommends \
        cuda-command-line-tools-9-0 \
        cuda-cublas-9-0 \
        cuda-cufft-9-0 \
        cuda-curand-9-0 \
        cuda-cusolver-9-0 \
        cuda-cusparse-9-0 \
        libcudnn7=7.2.1.38-1+cuda9.0 \
        libnccl2=2.2.13-1+cuda9.0 \
        libfreetype6-dev \
        libhdf5-serial-dev \
        libpng12-dev \
        libzmq3-dev \
        pkg-config \
        rsync \
        software-properties-common \
        unzip \
        && \
    echo ${DOCKERPASS} | sudo -S apt-get clean && \
    echo ${DOCKERPASS} | sudo -S rm -rf /var/lib/apt/lists/*

RUN echo ${DOCKERPASS} | sudo -S apt-get update && \
    echo ${DOCKERPASS} | sudo -S apt-get install nvinfer-runtime-trt-repo-ubuntu1604-4.0.1-ga-cuda9.0 && \
    echo ${DOCKERPASS} | sudo -S apt-get update && \
    echo ${DOCKERPASS} | sudo -S apt-get install libnvinfer4=4.1.2-1+cuda9.0

# Install TensorFlow GPU version.
RUN pip --no-cache-dir install tensorflow-gpu
# Install openai baselines
WORKDIR $HOME/conda/lib/python3.6/site-packages/
RUN git clone https://github.com/openai/baselines.git && cd baselines && \
    pip --no-cache-dir install -e .
WORKDIR $HOME

# For CUDA profiling, TensorFlow requires CUPTI.
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/extras/CUPTI/lib64
# <---- here
##################################### TENSORFLOW #####################################

ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD [ "/bin/bash" ]
