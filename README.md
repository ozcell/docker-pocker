# Docker setup

## Installing Docker

For more info: https://docs.docker.com/install/linux/docker-ce/ubuntu/

    $ sudo apt-get update
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    $ sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"   


    $ sudo apt-get update
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io

These two steps are required to run docker without sudo

    $ sudo groupadd docker
    $ sudo usermod -aG docker $USER

You may need to log off for this change to be effective 

## Pulling docker_wmgds

Pull the files through git 

    $ git clone https://ozsel@bitbucket.org/ozsel/docker_wmgds.git

or download the zip file and unzip

    $ wget --quiet https://www.dropbox.com/s/x905obnfuu8b6l1/docker_wmgds.zip?dl=0 -O ./docker_wmgds.zip && unzip docker_wmgds.zip -d $HOME/ && rm docker_wmgds.zip
## Building the Dockerfiles
    $ docker build --build-arg DOCKERUSER=<YOUR_USERNAME> \
                   --build-arg DOCKERPASS=<SOME_PASSWORD> \ 
                   --build-arg DOCKERID=<YOUR $UID IN SERVERS> \ 
                   --build-arg RESOLUTION=<SCREEN_RESOLUTION> \ 
                   --tag=<IMAGE_TAG> /path/to/docker_wmgds/stand-alone/


- <YOUR_USERNAME> should be your username used in the workstations to prevent ownership problems in files shared between the container and the main system
- <SOME_PASSWORD> could be anything. This is your sudo password and also the VNC password
- <YOUR $UID IN SERVERS> should be your $UID in wmgds servers starting with 100*. Default is 1000 but it is good to set this to prevent some file ownership issues.
- <SCREEN_RESOLUTION is the screen resolution for the VNC connection. Default is 1920x1080. Set something lower for slow internet connections
- <IMAGE_TAG> This will be used to run the image. Tag could anything be anything. 
  - In my case, I’m using something like this **stand-alone:stand-alone:u16.04-c9.0-a5.2-g0.11-p0.41-t1.12** to remember the versions of the libraries installed inside the image 
    - ubuntu16.04
    - cuda9.0
    - anaconda5.2
    - gym0.11
    - pytorch0.41
    - tensorflow1.12
## Running the Image → Making it a Container
    docker run -u <YOUR_USERNAME> \
               -p <XXXX>:5901 -p <YYYY>:8888 -p <ZZZZ>:6901 \
               -v /path/to/your/git/folder:/home/<YOUR_USERNAME>/git \
               -v /path/to/your/Jupyter/folder:/home/<YOUR_USERNAME>/Jupyter \
               --runtime=nvidia -it --rm <IMAGE_TAG>


- XXXX is the port to be used for the VNC connection. XXXX will be the port of the real server getting the VNC data. Do not forget to forward this port for the connection between your local computer and the server.
- YYYY is the port to be used Jupyter notebook. YYYY will be the port of the real server getting the Jupyter data. Do not forget to forward this port for the connection between your local computer and the server.
- You can also reach your desktop gui through your browser. ZZZZ is the port to be used for this purpose. Do not forget to forward this port for the connection between your local computer and the server.
- Mounting of some drives of the real server to the Container may be useful. -v is the option to be used.
- Please see the Docker documentation for other option -it or —rm.
## Inside the Container

**Running Jupyter Notebook**
You will get a prompt similar to this 

    (base) ok18@6c100e415232:~$ 

You can work here if you don’t need to render, e.g.

    (base) xx18@docker:~$ CUDA_VISIBLE_DEVICES=0 jupyter notebook --no-browser --ip=0.0.0.0 &

Please note that you need to write —ip option inside the container.

If you configure the port mapping for your ssh connection successfully, e.g.

    yourname@yourlocalpc:~$ ssh -t -L <YYYY>:localhost:<YYYY> xx18@montana-titanx8.wmg.warwick.ac.uk

(note that <YYYY> is the same as above), then you will be able to reach Jupyter notebook by visiting

- http://localhost:<YYYY> 

in your local pc.

**Running the GUI**
When you need a gui, simple run the following script

    (base) xx18@docker:~$ gui/vnc_startup.sh &

Given that you configured your ssh connection successfully, e.g.

    yourname@yourlocalpc:~$ ssh -t -L <XXXX>:localhost:<XXXX> xx18@montana-titanx8.wmg.warwick.ac.uk

and/or

    yourname@yourlocalpc:~$ ssh -t -L <ZZZZ>:localhost:<ZZZZ> xx18@montana-titanx8.wmg.warwick.ac.uk

then you will be able to see your desktop by visiting 

- http://localhost:<ZZZZ> 

and/or 

- vnc://localhost:<XXXX>

You can still use Jupyter Notebook directly from your local browser. When Rendering window pops up, you will be able to see this window either in http://localhost:<ZZZZ> or vnc://localhost:<XXXX>


## About Dockerfile

As you will notice Dockerfile is simple text file including bash comments required for libraries to be installed. You can change the corresponding parts to create your own setup. e.g. Ubuntu18.04 with cuda10 and Pytorch1.0. All you need to do is to find the commands to install new libraries and write them with RUN command. Most of the time the link of the installation files are consistent. So changing file names might even work for most of the time. 

