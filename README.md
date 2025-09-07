# Ubuntu ML Setup in Q2 2025

## Constraints
- 5000 Series GPU
- Dual boot
- Secure boot enabled
- Ubuntu 24.04 Desktop
- Nvidia container toolkit will be used - development will occur in devcontainers
 - No need for conda  
  
## Instructions

### Overview

- We will install nvidia gpu drivers on ubuntu. Then install docker and the nvidia container toolkit so we can use CUDA toolkit in development on containers as to create separation between the machine and development/training environments.

### Installing Nvidia Drivers

Reference: [Ubuntu Nvidia Driver Instructions](https://documentation.ubuntu.com/server/how-to/graphics/install-nvidia-drivers/)

Notes:
- The ubuntu-drivers tool is recommended if your computer uses Secure Boot, since it always tries to install signed drivers which are known to work with Secure Boot.
- 5000 series GPUs only support the open driver

#### Update everything first

```sh
sudo apt update && sudo apt upgrade -y
```

#### Check the available drivers for your hardware

```sh
sudo ubuntu-drivers list
> nvidia-driver-575-open, (kernel modules provided by linux-modules-nvidia-575-open-generic-hwe-24.04)
> nvidia-driver-570-open, (kernel modules provided by linux-modules-nvidia-570-open-generic-hwe-24.04)
> nvidia-driver-575, (kernel modules provided by linux-modules-nvidia-575-generic-hwe-24.04)
> nvidia-driver-570, (kernel modules provided by linux-modules-nvidia-570-generic-hwe-24.04)
> nvidia-driver-570-server-open, (kernel modules provided by linux-modules-nvidia-570-server-open-generic-hwe-24.04)
> nvidia-driver-575-server-open, (kernel modules provided by linux-modules-nvidia-575-server-open-generic-hwe-24.04)
> nvidia-driver-570-server, (kernel modules provided by linux-modules-nvidia-570-server-generic-hwe-24.04)
> nvidia-driver-575-server, (kernel modules provided by linux-modules-nvidia-575-server-generic-hwe-24.04)
```

#### Install the latest open driver

note: use ubuntu-drivers if you are using secure boot as it always tries to install signed drivers 

```sh
sudo ubuntu-drivers install nvidia:575-open
sudo reboot
```

#### Check

```sh
nvidia-smi
```
#### Install nvtop [optional]

nvtop is a terminal application for monitoring your gpu

```sh
sudo apt install nvtop
nvtop
```

### Install Docker and Nvidia Container Toolkit

Reference: [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

Best approach is to follow the docs. Below is a cut down version of what worked.

Remove old Docker if present:
```sh
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

Set up Docker's apt repository.

```sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install the Docker packages to the latest version:
```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Enable non-root use:

```sh
sudo usermod -aG docker $USER
newgrp docker
```

Verify that the installation:

```sh
sudo docker run hello-world
```
This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.

### Installing the Nvidia Container Toolkit

Reference: [Installing the NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
Best approach is to follow the docs. Below is a cut down version of what worked.

Set up Nvidia Container Toolkit apt repository.
```sh
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Update + install (latest stable, docs pin to a version)
```sh
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

Configure the container runtime by using the nvidia-ctk command:
```sh
sudo nvidia-ctk runtime configure --runtime=docker
```

The nvidia-ctk command modifies the /etc/docker/daemon.json file on the host. The file is updated so that Docker can use the NVIDIA Container Runtime.

Restart the Docker daemon:
```sh
sudo systemctl restart docker
```
### Testing a sample workload container

Reference:[Running a Sample Workload with Docker](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/sample-workload.html)

Run a sample CUDA container:
```sh
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

Your output should resemble the following output:
```sh
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 535.86.10    Driver Version: 535.86.10    CUDA Version: 12.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
| N/A   34C    P8     9W /  70W |      0MiB / 15109MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

