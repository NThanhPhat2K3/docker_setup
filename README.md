# Install docker in ubuntu 

## 1. Update the system
```bash
sudo apt-get update
sudo apt-get upgrade -y
```
##  2. Install required packages

```bash
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
## 3. Add Docker’s official GPG key

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
## 4. Set up the Docker repository

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
## 5. Install Docker Engine and Docker Compose

```bash
sudo apt-get install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin
```
## 6. Verify installation

```bash
docker --version
docker compose version
```

## 7. (Optional) Run Docker as a non-root user

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