+++
title = "How to live patch MAAS snap"
type = "chapter"
weight = 7
+++

# How to live patch MAAS snap

You can copy the file you want to edit to your home directory with for example

```
sudo cp /snap/maas/current/lib/python3.10/site-packages/maasserver/rpc/boot.py .
```

then you can modify it and then you 

```
sudo mount -o ro,bind boot.py /snap/maas/current/lib/python3.10/site-packages/maasserver/rpc/boot.py
```

NOTE: if you reboot your system, the mount will be lost and the changes are not persisted. If you want to unmount, then 

```
sudo umount /snap/maas/current/lib/python3.10/site-packages/maasserver/rpc/boot.py
```
