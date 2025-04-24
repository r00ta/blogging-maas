+++
date = '2025-04-22T23:27:20+01:00'
type = "chapter"
title = 'Machine User/Password login'
weight = 3
+++

## How to login to a machine with user/password

You can achieve this in multiple ways: 
- use a cloud-init script to add a new user with a password
- create your own image and deploy it
- let curtin do the job

In this guide, we will use the curtin approach as it's the easies/fastest and it will work on all the deployments.

Create a password and get the hash 
```
python3 -c "import crypt; print(crypt.crypt('changeme', crypt.mksalt(crypt.METHOD_SHA512)))"
```

Simply place the following content in the file `/var/snap/maas/current/preseeds/curtin_userdata`

```
#cloud-config
debconf_selections:
 maas: |
  {{for line in str(curtin_preseed).splitlines()}}
  {{line}}
  {{endfor}}
late_commands:
  00_maas: [wget, '--no-proxy', {{node_disable_pxe_url|escape.json}}, '--post-data', {{node_disable_pxe_data|escape.json}}, '-O', '/dev/null']
  10_adduser: ["curtin", "in-target", "--", "sh", "-c", "adduser --disabled-password --gecos '' maas"]
  20_addpwd: ["curtin", "in-target", "--", "sh", "-c", "echo 'maas:maas' | chpasswd"]
  30_addsudo: ["curtin", "in-target", "--", "sh", "-c", "usermod -aG sudo maas"]
```
