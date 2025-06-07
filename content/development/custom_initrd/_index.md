+++
title = "BYO kernel and initrd for development"
type = "chapter"
weight = 8
+++

Sometimes I needed to test my own initrd and kernel with MAAS, so this is how I do it. 

## Prerequisites

Build the kernel `boot-kernel` and initrd `boot-initrd`! Then, create a qemu disk:

```bash
qemu-img create -f qcow2 rootfs.qcow2 2G
```

and say you have MAAS running at `172.0.2.2`. Then you would simply 

```bash
qemu-system-x86_64 \
  -kernel boot-kernel \
  -initrd boot-initrd \
  -append "nomodeset root=squash:http://172.0.2.2:5248/images/6cd63c9/ubuntu/amd64/ga-20.04/focal/stable/squashfs ip=::::C220M4:BOOTIF ip6=off cc:{'datasource_list': ['MAAS']}end_cc cloud-config-url=http://172.0.2.2:5248/MAAS/metadata/latest/enlist-preseed/?op=get_enlist_preseed ro overlayroot=tmpfs overlayroot_cfgdisk=disabled apparmor=0 log_host=172.0.2.2 log_port=5247 --- iscsi_auto BOOTIF=01-52-54-00-ab-cd-ef" \
  -drive file=rootfs.qcow2,format=qcow2,if=virtio \
  -m 2048M \
  -netdev user,id=net-test \
  -device virtio-net-pci,netdev=net-test,mac=52:54:00:ab:cd:ef
```

Just to test a valid scenario, just download the kernel and the initrd from images.maas.io and try them. Then you can try your ones!