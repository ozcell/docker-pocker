# This Dockerfile is used to build an headles vnc image based on Ubuntu
FROM command-line:u16.04-c9.0-a5.2-g0.11-p0.41-t1.12

USER $DOCKERUSER 
WORKDIR $HOME

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

# RUN echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$HOME/.mujoco/mjpro150/bin" >> ~/.bashrc && \
#     echo "export LIBRARY_PATH=$LIBRARY_PATH:$HOME/.mujoco/mjpro150/bin" >> ~/.bashrc && \
#     echo "export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libGLEW.so" >> ~/.bashrc 

#RUN echo "export PYTHONPATH=$PYTHONPATH:$HOME/git/multiagent-detection:$HOME/git/multiagent-communication:$HOME/git/sparse_rewards:$HOME/git/lalnets-pytorch" >> ~/.bashrc

ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD [ "/bin/bash" ]
