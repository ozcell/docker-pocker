######################################## MAIN ########################################
# install anaconda, pytorch
ARG UBUNTU_VERSION=16.04
FROM nvidia/cuda:10.2-base-ubuntu${UBUNTU_VERSION}

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
    libglib2.0-0 libxext6 libsm6 libxrender1 git mercurial subversion nano \
    unzip rsync pkg-config software-properties-common cmake swig xvfb ffmpeg unzip

# install anaconda
#RUN wget --quiet https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh -O ~/anaconda.sh && \
RUN wget --quiet https://repo.anaconda.com/archive/Anaconda3-2019.07-Linux-x86_64.sh -O ~/anaconda.sh && \
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

#RUN echo ${DOCKERPASS} | sudo -S apt-get install -y cmake swig xvfb ffmpeg unzip

#install pytorch, torchvision torchaudio
#RUN pip --no-cache-dir install torch==1.8.1+cu111 torchvision==0.9.1+cu111 torchaudio==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html
#RUN pip --no-cache-dir install torch torchvision torchaudio
#RUN pip --no-cache-dir install tensorboardX
#RUN pip --no-cache-dir install mpi4py

RUN echo 'for d in $HOME/git/*; do export PYTHONPATH="$PYTHONPATH:$d"; done' >> $HOME/.bashrc
######################################## MAIN ########################################

##################################### TENSORFLOW #####################################
# <---- if tensorflow is not needed this part could be commented up to 'here' label
EXPOSE 6006

#Pick up some TF dependencies
#RUN echo ${DOCKERPASS} | sudo -S apt-get update && \ 
#    echo ${DOCKERPASS} | sudo -S apt-get install -y --no-install-recommends \
#        cuda-command-line-tools-9-0 \
#        cuda-cublas-9-0 \
#        cuda-cufft-9-0 \
#        cuda-curand-9-0 \
#        cuda-cusolver-9-0 \
#        cuda-cusparse-9-0 \
#        libcudnn7=7.2.1.38-1+cuda9.0 \
#        libnccl2=2.2.13-1+cuda9.0 \
#        libfreetype6-dev \
#        libhdf5-serial-dev \
#        libpng12-dev \
#        libzmq3-dev \
#        pkg-config \
#        rsync \
#        software-properties-common \
#        unzip \
#        && \
 #   echo ${DOCKERPASS} | sudo -S apt-get clean && \
 #   echo ${DOCKERPASS} | sudo -S rm -rf /var/lib/apt/lists/*

#RUN echo ${DOCKERPASS} | sudo -S apt-get update && \
#    echo ${DOCKERPASS} | sudo -S apt-get install nvinfer-runtime-trt-repo-ubuntu1604-4.0.1-ga-cuda9.0 && \
#    echo ${DOCKERPASS} | sudo -S apt-get update && \
#    echo ${DOCKERPASS} | sudo -S apt-get install libnvinfer4=4.1.2-1+cuda9.0

# Install TensorFlow GPU version.
#RUN pip --no-cache-dir install "tensorflow>=1.15,<2.0"
#RUN pip --no-cache-dir install --upgrade tensorflow-hub

# For CUDA profiling, TensorFlow requires CUPTI.
#ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/extras/CUPTI/lib64
# <---- here
##################################### TENSORFLOW #####################################

######################################## GUI #########################################
## Connection ports for controlling the UI:
# VNC port:5901
# noVNC webport, connect via http://IP:6901/?password=vncpassword
ENV DISPLAY=:1 \
    VNC_PORT=5901 \
    NO_VNC_PORT=6901 \
    JUPYTER_PORT=8888
EXPOSE $VNC_PORT $NO_VNC_PORT $JUPYTER_PORT

### Envrionment config
ENV TERM=xterm \
    STARTUPDIR=$HOME/gui \
    NO_VNC_HOME=$HOME/gui/noVNC \
    DEBIAN_FRONTEND=noninteractive \
    VNC_COL_DEPTH=24 \
    VNC_RESOLUTION=${RESOLUTION} \
    VNC_PW=${DOCKERPASS} \
    VNC_VIEW_ONLY=false

### Add all install scripts for further steps
ADD ./src/common/xfce/ $HOME/
RUN echo ${DOCKERPASS} | sudo -S chown -R ${DOCKERUSER} $HOME/.config $HOME/Desktop 
ADD ./src/common/scripts $STARTUPDIR/
RUN echo ${DOCKERPASS} | sudo -S chown -R ${DOCKERUSER} $STARTUPDIR/

### Install some common tools
RUN echo ${DOCKERPASS} | sudo -S apt-get update && \
    echo ${DOCKERPASS} | sudo -S apt-get install -y vim wget net-tools locales bzip2 python-numpy && \
    echo ${DOCKERPASS} | sudo -S apt-get clean -y && \
    echo ${DOCKERPASS} | sudo -S locale-gen en_GB.UTF-8 

ENV LANG=en_GB.UTF-8 \ 
    LANGUAGE=en_GB:en \
    LC_ALL=en_GB.UTF-8 

### Install custom fonts
RUN echo ${DOCKERPASS} | sudo -S apt-get install -y ttf-wqy-zenhei

### Install xvnc-server & noVNC - HTML5 based VNC viewer
USER 0
RUN wget -qO- https://dl.bintray.com/tigervnc/stable/tigervnc-1.9.0.x86_64.tar.gz | tar xz --strip 1 -C /
USER $DOCKERUSER 

RUN mkdir -p $NO_VNC_HOME/utils/websockify && \
    wget -qO- https://github.com/novnc/noVNC/archive/v1.0.0.tar.gz | tar xz --strip 1 -C $NO_VNC_HOME && \
    wget -qO- https://github.com/novnc/websockify/archive/v0.6.1.tar.gz | tar xz --strip 1 -C $NO_VNC_HOME/utils/websockify && \
    chmod +x -v $NO_VNC_HOME/utils/*.sh && \
    ln -s $NO_VNC_HOME/vnc_lite.html $NO_VNC_HOME/index.html

### Install chrome browser
RUN echo ${DOCKERPASS} | sudo -S apt-get update && \
    echo ${DOCKERPASS} | sudo -S apt-get install -y chromium-browser chromium-browser-l10n chromium-codecs-ffmpeg && \
    echo ${DOCKERPASS} | sudo -S apt-get clean -y && \
    echo ${DOCKERPASS} | sudo -S ln -s /usr/bin/chromium-browser /usr/bin/google-chrome && \
    echo "CHROMIUM_FLAGS='--no-sandbox --start-maximized --user-data-dir'" > $HOME/.chromium-browser.init

### Install xfce UI
USER 0
RUN apt-get update && \
    apt-get install -y supervisor xfce4 xfce4-terminal xterm && \
    apt-get purge -y pm-utils xscreensaver* && \
    apt-get clean -y
USER $DOCKERUSER 
######################################## GUI #########################################

ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD [ "/bin/bash" ]
