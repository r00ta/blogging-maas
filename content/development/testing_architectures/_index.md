+++
title = "Testing architectures"
type = "chapter"
weight = 5
+++

How to test different architectures in MAAS without hardware! For example, you can test arm64 and others.

## Setup MAAS

As usual, install the MAAS dev environment with 2 networks

```
$ lxc network list

+-----------------+----------+---------+----------------+---------------------------+-------------+---------+---------+
|      NAME       |   TYPE   | MANAGED |      IPV4      |           IPV6            | DESCRIPTION | USED BY |  STATE  |
+-----------------+----------+---------+----------------+---------------------------+-------------+---------+---------+
| net-lab         | bridge   | YES     | 10.0.1.1/24    | none                      |             | 26      | CREATED |
+-----------------+----------+---------+----------------+---------------------------+-------------+---------+---------+
| net-test        | bridge   | YES     | 10.0.2.1/24    | none                      |             | 26      | CREATED |
+-----------------+----------+---------+----------------+---------------------------+-------------+---------+---------+

where `net-test` is the network where the machine will be deployed. LXD DHCP is disabled on this network, because it will be managed by MAAS. Of course, download arm64 images in MAAS.

## Install qemu

```
sudo apt install qemu-system-arm
```


## Create a disk and start the machine

```
qemu-img create -f qcow2 disk.qcow2 20G

sudo qemu-system-aarch64 \
  -machine virt \
  -cpu cortex-a57 \
  -m 4096 \
  -bios /usr/share/qemu-efi-aarch64/QEMU_EFI.fd \
  -boot order=n \
  -drive file=./disk.qcow2,format=qcow2 \
  -netdev bridge,id=net0,br=net-test \
  -device virtio-net-device,netdev=net0,mac=52:54:00:00:00:02
```

Enjoy :D