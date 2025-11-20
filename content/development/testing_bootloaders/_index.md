+++
title = "Testing bootloaders"
type = "chapter"
weight = 6
+++

How to test different bootloaders without using MAAS. 

## Setup

Install LXD

```
sudo snap install lxd
```

and initialize it 

```
lxd init --auto
```

create a new network 

```
lxc network create net-test --type=bridge ipv4.address=10.0.1.1/24 ipv4.dhcp=false ipv4.nat=true ipv6.dhcp=false ipv6.address=none
```

start a new container 

```
lxc launch ubuntu:24.04 pxe-server
```

attach the network to the container

```
lxc config device add pxe-server eth0 nic network=net-test name=eth0
```

Now get a shell in the container with 

```
lxc shell pxe-server
```

Create the netplan configuration file:

```
sudo tee /etc/netplan/99-static.yaml > /dev/null << 'EOF'
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 10.0.1.2/24
      nameservers:
        addresses:
          - 8.8.8.8
      routes:
        - to: default
          via: 10.0.1.1
EOF
```

Apply the changes 

```
sudo netplan apply
```

Always inside the container, install 

```
sudo apt-get update
sudo apt install -y isc-dhcp-server tftpd-hpa
```

Create the DHCP server configuration:

```
sudo tee /etc/dhcp/dhcpd.conf > /dev/null << 'EOF'
default-lease-time 600;
max-lease-time 7200;

authoritative;

option arch code 93 = unsigned integer 16; # RFC4578

subnet 10.0.1.0 netmask 255.255.255.0 {
    range 10.0.2.50 10.0.1.200;
    option routers 10.0.1.1;
    option domain-name-servers 10.0.1.1;
    next-server 10.0.1.2;
    if option arch = 00:0B {
      filename "bootaa64.efi";
    }
}
EOF
```

Create the TFTP server configuration:

```
sudo tee /etc/default/tftpd-hpa > /dev/null << 'EOF'
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/srv/tftp"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="--secure --verbose"
EOF
```

Restart the services

```
sudo systemctl restart tftpd-hpa.service
sudo systemctl restart isc-dhcp-server
```

Now create a folder where you'll serve the kernel, initrd and squashfs 

```
mkdir ~/images
cd ~/images
```

and create a couple of folders

```
mkdir ~/images/noble
mkdir ~/images/xenial
```

Place the images for xenial

```
wget https://images.maas.io/ephemeral-v3/stable/xenial/arm64/20210928/squashfs -O ~/images/xenial/squashfs
wget https://images.maas.io/ephemeral-v3/stable/xenial/arm64/20210928/ga-16.04/generic/boot-initrd -O ~/images/xenial/boot-initrd
wget https://images.maas.io/ephemeral-v3/stable/xenial/arm64/20210928/ga-16.04/generic/boot-kernel -O ~/images/xenial/boot-kernel
```

and for noble

```
wget https://images.maas.io/ephemeral-v3/stable/noble/arm64/20250921/squashfs -O ~/images/noble/squashfs
wget https://images.maas.io/ephemeral-v3/stable/noble/arm64/20250921/ga-24.04/generic/boot-initrd -O ~/images/noble/boot-initrd
wget https://images.maas.io/ephemeral-v3/stable/noble/arm64/20250921/ga-24.04/generic/boot-kernel -O ~/images/noble/boot-kernel
```

and put the bootloaders in the tftp root

```
sudo mkdir -p /srv/tftp
sudo mkdir -p /srv/tftp/jammy

cd /tmp

# noble bootloaders
wget https://images.maas.io/ephemeral-v3/candidate/bootloaders/uefi/arm64/20250715.0/grub2-signed.tar.xz
tar -xvf grub2-signed.tar.xz
wget https://images.maas.io/ephemeral-v3/candidate/bootloaders/uefi/arm64/20250715.0/shim-signed.tar.xz
tar -xvf shim-signed.tar.xz

sudo mv grubaa64.efi /srv/tftp
sudo mv mmaa64.efi /srv/tftp
sudo mv bootaa64.efi /srv/tftp
rm shim-signed.tar.xz
rm grub2-signed.tar.xz

# jammy bootloaders
wget https://images.maas.io/ephemeral-v3/candidate/bootloaders/uefi/arm64/20250821.0/grub2-signed.tar.xz
tar -xvf grub2-signed.tar.xz
wget https://images.maas.io/ephemeral-v3/candidate/bootloaders/uefi/arm64/20250821.0/shim-signed.tar.xz
tar -xvf shim-signed.tar.xz
sudo mv grubaa64.efi /srv/tftp/jammy
sudo mv mmaa64.efi /srv/tftp/jammy
sudo mv bootaa64.efi /srv/tftp/jammy
rm shim-signed.tar.xz
rm grub2-signed.tar.xz
```

Now, create the grub configs

```
sudo mkdir -p /srv/tftp/grub/
```

Create the main grub configuration:

```
sudo tee /srv/tftp/grub/grub.cfg > /dev/null << 'EOF'
# MAAS GRUB2 pre-loader configuration file

# Load based on MAC address first.
configfile /grub/grub.cfg-${net_default_mac}

# Failed to load based on MAC address. Load based on the CPU
# architecture.
configfile /grub/grub.cfg-default-${grub_cpu}
EOF
```

Create the default ARM64 grub configuration:

```
sudo tee /srv/tftp/grub/grub.cfg-default-arm64 > /dev/null << 'EOF'
set default="0"
set timeout=0

menuentry 'Ephemeral' {
    echo   'Booting under MAAS direction...'
    linux  (http,10.0.1.2:8080)/boot-kernel nomodeset root=squash:http://10.0.1.2:8080/squashfs ip=::::maas-enlist:BOOTIF ip6=off cc:\{'datasource_list': ['MAAS']\}end_cc cloud-config-url=http://10.0.1.2:8080/get_enlist_preseed ro overlayroot=tmpfs overlayroot_cfgdisk=disabled BOOTIF=01-${net_default_mac}
    initrd (http,10.0.1.2:8080)/boot-initrd
}
EOF
```

Change permissions of the tftp directory 

```
sudo chmod -R 755 /srv/tftp
```

And finally start an http server for the images

```
cd ~/images/xenial/
python3 -m http.server 8080
```


## Create a disk and start the machine

In another shell, go back to the **host**. 

Install qemu

```
sudo apt install qemu-system-arm -y 
```

Configure qemu bridge:

```
sudo mkdir -p /etc/qemu
echo "allow net-test" | sudo tee /etc/qemu/bridge.conf > /dev/null
```

finally 

```
qemu-img create -f qcow2 disk.qcow2 20G
sudo qemu-system-aarch64   -machine virt   -cpu cortex-a57   -m 4096   -bios /usr/share/qemu-efi-aarch64/QEMU_EFI.fd   -boot order=n   -drive file=./disk.qcow2,format=qcow2   -netdev bridge,id=net0,br=net-test   -device virtio-net-device,netdev=net0,mac=52:54:00:00:00:02 -nographic
```

In the terminal you should see the machine getting an IP and "booting under MAAS". And it should fail. Then, you can go back to the container

```
lxc shell pxe-server
cd /srv/ftfp
sudo cp jammy/* . 
sudo chmod -R 755 /srv/tftp
```

and in the host 

```
sudo pkill qemu
```

and restart the emulator. This time, it should succeed. I.e. 

- xenial kernel/initrd/squashfs works with jammy bootloaders
- xenial kernel/initrd/squashfs does not work with noble bootloaders.