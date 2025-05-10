+++
title = "How to build SONiC images"
type = "chapter"
weight = 6
+++

I write these notes here because I'm going to forget how to do it over and over again. 

## Preerequisites

Deploy a 22.04 Ubuntu server on bare metal with at least 100GB of disk space and install  the following packages:


```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install -y make ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


sudo groupadd docker

sudo usermod -aG docker $USER

newgrp docker

sudo apt install -y python3-pip
sudo pip install j2cli
```

My switch `N3248TE` is currently affected by a [known bug](https://github.com/sonic-net/sonic-buildimage/issues/18421), so let's pick the branch `202305` that is supposed to be working fine.
``` 
git clone --recurse-submodules https://github.com/sonic-net/sonic-buildimage.git -b 202305
```

## Build the image

```
cd sonic-buildimage
make init
make configure PLATFORM=broadcom
ENABLE_ZTP=y SONIC_BUILD_JOBS=4 make target/sonic-broadcom.bin
```

