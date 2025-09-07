# Ubuntu ML Setup in Q2 2025

## Constraints
- 5000 Series GPU
- Dual boot
- Secure boot enabled
- Ubuntu 24.04 Desktop
  
## Instructions

Reference: [Ubuntu Nvidia Driver Instructions](https://documentation.ubuntu.com/server/how-to/graphics/install-nvidia-drivers/)

Notes:
- The ubuntu-drivers tool is recommended if your computer uses Secure Boot, since it always tries to install signed drivers which are known to work with Secure Boot.
- 5000 series GPUs only support the open driver

### Update everything first

```sh
sudo apt update && sudo apt upgrade -y
```

### Check the available drivers for your hardware

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

### Install the latest open driver

note: use ubuntu-drivers if you are using secure boot as it always tries to install signed drivers 

```sh
sudo ubuntu-drivers install nvidia:575-open
sudo reboot
```

### Check

```sh
nvidia-smi
```

