+++
title = "Uninstall"
type = "chapter"
weight = 99
+++

Some instructions on how to completely remove MAAS and all its data.

### Snap

```
sudo snap remove --purge maas
sudo snap remove --purge maas-test-db
```

### Debs

```
sudo DEBIAN_FRONTEND=noninteractive apt remove --purge -y 'maas*' && sudo DEBIAN_FRONTEND=noninteractive apt -y autoremove
```
