+++
title = "HTTPS APT mirror"
type = "chapter"
weight = 5
+++

## How to use an HTTPS APT mirror with MAAS

APT packages are already secured cryptographically, but if for stupid compliance reasons you want to use an HTTPS mirror, you can do it.

In order to inject the self signed certificates to be trusted during commissioning and the deployment, you have to add the following files 

- `/var/snap/maas/current/preseeds/enlist`
- `/var/snap/maas/current/preseeds/commission`
- `/var/snap/maas/current/preseeds/curtin`

and add the following lines to them:

```
{{preseed_data}}
ca-certs:
  trusted:
  - |
    -----BEGIN CERTIFICATE-----
    xxxxx
    -----END CERTIFICATE-----
```

if you use debs, these files are under `/etc/maas/preseeds/`. 