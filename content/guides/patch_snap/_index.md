+++
title = "How to live patch MAAS snap"
type = "chapter"
weight = 7
+++

# How to live patch MAAS snap

The Python package path depends on the Ubuntu release and MAAS version:

```
Jammy (MAAS 3.3-3.5):   /snap/maas/current/lib/python3.10/site-packages/
Noble (MAAS 3.6-3.7):   /snap/maas/current/usr/lib/python3.12/dist-packages/
Resolute (MAAS 3.8):    /snap/maas/current/usr/lib/python3.14/dist-packages/
```

Set `PACKAGE_DIR` to the path for your installation. For example, on Resolute
with MAAS 3.8:

```
PACKAGE_DIR=/snap/maas/current/usr/lib/python3.14/dist-packages
```

Copy the file you want to edit to your home directory, then modify it:

```
sudo cp "$PACKAGE_DIR/maasserver/rpc/boot.py" .
```

Bind-mount your edited file over the original:

```
sudo mount -o ro,bind boot.py "$PACKAGE_DIR/maasserver/rpc/boot.py"
```

If you reboot your system, the mount will be lost and the changes will not
persist. To unmount the file:

```
sudo umount "$PACKAGE_DIR/maasserver/rpc/boot.py"
```
