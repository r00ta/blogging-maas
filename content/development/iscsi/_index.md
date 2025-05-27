+++
title = "Setup iSCSI with Qemu"
type = "chapter"
weight = 7
+++

## Setup iSCSI with Qemu

This is a guide to setup iSCSI with QEMU and then point your server to it. 

### Setup the environment

```bash
sudo apt get install tgt qemu-utils -y
```

### Setup the iSCSI target

```
tgtd --iscsi portal=10.0.1.100:3260
qemu-img create -f raw /home/ubuntu/disk.img 10G
sudo tgtadm --lld iscsi --op new --mode target --tid 1 -T iqn.qemu.test
sudo tgtadm --lld iscsi --mode logicalunit --op new   --tid 1 --lun 1 --backing-store /home/ubuntu/disk.img
sudo tgtadm --lld iscsi --op bind --mode target --tid 1 -I ALL
sudo tgtadm --lld iscsi --op show --mode target
```
