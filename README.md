# 1. Install docker in Ubuntu

## 1.1 Update the system

```bash
sudo apt-get update
sudo apt-get upgrade -y
```
##  1.2 Install required packages

```bash
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
## 1.3 Add Docker’s official GPG key

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
## 1.4 Set up the Docker repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Update the package index:

```bash
sudo apt-get update
```
## 1.5 Install Docker Engine and Docker Compose

```bash
sudo apt-get install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin
```
## 1.6 Verify installation

```bash
docker --version
docker compose version
```

## 1.7 (Optional) Run Docker as a non-root user

Add your user to the docker group:

```bash
sudo groupadd docker   # (if the group does not already exist)
sudo usermod -aG docker $USER
newgrp docker
```

Test without sudo:

```bash
docker run hello-world
```

# 2. Build and Run Ubuntu 22.04 Docker with GUI (SSH + VNC + X11)

## 2.1 Build the Image

```bash
docker build --no-cache \
  --build-arg "host_uid=$(id -u)" \
  --build-arg "host_gid=$(id -g)" \
  --build-arg "USERNAME=$USER" \
  --build-arg "TZ_VALUE=$(cat /etc/timezone)" \
  --tag ubuntu-22.04 \
  --file Dockerfile.ubuntu-22.04 .
```

## 2.2 Allow local X11 access

Run this once on your host:

```bash
xhost +local:root
```

## 2.3 Start the Container (with SSH + VNC + Display)

```bash
docker run -it \
  --name my_container_for_22.04 \
  -e DISPLAY=$DISPLAY \ # X11
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -p 5901:5901 \ # VNC
  -p 2222:22 \   # SSH
  --volume="./share:/home/$USER/yocto" \
  --workdir="/home/$USER" \
  ubuntu-22.04
```

## 2.4 Exit Container Without Stopping It

Exit and stop:

```bash
exit
```
## 2.5 Manage the Container

Start container (if stopped):

```bash
docker start -ai my_container_for_22.04
```
Attach new shell to running container:

```bash
docker exec -it my_container_for_22.04 /bin/bash
```

Stop container:

```bash
docker stop my_container_for_22.04
```

## 2.6 Save Container as New Image

Commit your container into a new image

```bash
docker commit my_container_for_22.04 ubuntu-22.04-vnc:latest
```

Save that image into a .tar archive

```bash
docker save -o ubuntu-22.04-vnc.tar ubuntu-22.04-vnc:latest
```
Now you’ll have a file named ubuntu-22.04-vnc.tar in your current directory.

Load it back on another machine

```bash
docker load -i ubuntu-22.04-vnc.tar
```
## 2.7 Connect

SSH:

```bash
ssh -p 2222 user@localhost
```
VNC:

Open VNC Viewer → connect to localhost:5901

X11 GUI apps:

```bash
firefox &
qtcreator &
```

## 3. Setup SSH + VNC + Display inside Docker

## 3.1 Install SSH server

```bash
sudo apt install -y openssh-server
```

Start the SSH service:

```bash
sudo service ssh start
```

## 3.2 Install GUI + VNC server

```bash
sudo apt-get install -y xfce4 xfce4-goodies tightvncserver
```
## 3.3 Initialize VNC server (first time)

```bash
vncserver :1
```
You will be asked to set a password (max 8 characters).

## 3.4 Configure VNC startup script

Export environment variable for safety:

```bash
export USER=$(whoami)
```

Edit the startup file:

```bash
vim ~/.vnc/xstartup
```
Replace the content with:

```bash
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```
Make it executable:

```bash
chmod +x ~/.vnc/xstartup
```
## 3.5 Restart and run VNC server

```bash
vncserver -kill :1
vncserver :1 -geometry 1280x800 -depth 24
```
## 3.6 Export Display

```bash
export DISPLAY=:0
```
## 3.7 Install Firefox (ESR version – lighter than Snap)

```bash
sudo apt-get update
sudo apt-get install -y software-properties-common
sudo add-apt-repository -y ppa:mozillateam/ppa
sudo apt-get update
sudo apt-get install -y firefox-esr
```
## 3.8 Install Required Libraries for Qt and GUI Apps

These libraries fix missing dependencies for Qt6 apps, Qt Creator, and other GUI tools.

```bash
sudo apt-get install -y libxkbcommon-x11-0
sudo apt install libxcb-cursor0 libxcb-cursor-dev
sudo apt-get install -y \
    libdbus-1-3 \
    libxcb-icccm4 \
    libxcb-image0 \
    libxcb-keysyms1 \
    libxcb-render-util0 \
    libxcb-xinerama0 \
    libxcb-xfixes0 \
    libxcb-shape0 \
    libxcb-randr0 \
    libxcb-xkb1 \
    libxcb-sync1
```