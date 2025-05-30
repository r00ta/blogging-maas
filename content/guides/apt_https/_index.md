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

Then, add `/var/snap/maas/current/preseeds/curtin_userdata` with the following content:

```
#cloud-config
debconf_selections:
 maas: |
  {{for line in str(curtin_preseed).splitlines()}}
  {{line}}
  {{endfor}}
curthooks_commands:
  01_add_cert: ["curtin", "in-target", "--", "sh", "-c", "echo '-----BEGIN CERTIFICATE-----\n CHANGEME \n-----END CERTIFICATE-----' > /usr/local/share/ca-certificates/apt.crt"]
  02_update_cert: ["curtin", "in-target", "--", "update-ca-certificates"]
  03_curthooks: ["curtin", "curthooks"]
early_commands:
  apt_00: ["curtin", "in-target", "--",  "// ADD CERTIFICATES TO THE LOCATION]"
{{if third_party_drivers and driver}}
  {{py: key_string = ''.join(['\\x%x' % x for x in driver['key_binary']])}}
  {{if driver['key_binary'] and driver['repository'] and driver['package']}}
  driver_00_get_key: /bin/echo -en '{{key_string}}' > /tmp/maas-{{driver['package']}}.gpg
  driver_01_add_key: ["apt-key", "add", "/tmp/maas-{{driver['package']}}.gpg"]
  {{endif}}
  {{if driver['repository']}}
  driver_02_add: ["add-apt-repository", "-y", "deb {{driver['repository']}} {{node.get_distro_series()}} main"]
  {{endif}}
  {{if driver['package']}}
  driver_03_update_install: ["sh", "-c", "apt-get update --quiet && apt-get --assume-yes install {{driver['package']}}"]
  {{endif}}
  {{if driver['module']}}
  driver_04_load: ["sh", "-c", "depmod && modprobe {{driver['module']}} || echo 'Warning: Failed to load module: {{driver['module']}}'"]
  {{endif}}
{{else}}
  driver_00: ["sh", "-c", "echo third party drivers not installed or necessary."]
{{endif}}
late_commands:
  maas: [wget, '--no-proxy', {{node_disable_pxe_url|escape.json}}, '--post-data', {{node_disable_pxe_data|escape.json}}, '-O', '/dev/null']
{{if third_party_drivers and driver}}
  {{if driver['key_binary'] and driver['repository'] and driver['package']}}
  driver_00_key_get: curtin in-target -- sh -c "/bin/echo -en '{{key_string}}' > /tmp/maas-{{driver['package']}}.gpg"
  driver_02_key_add: ["curtin", "in-target", "--", "apt-key", "add", "/tmp/maas-{{driver['package']}}.gpg"]
  {{endif}}
  {{if driver['repository']}}
  driver_03_add: ["curtin", "in-target", "--", "add-apt-repository", "-y", "deb {{driver['repository']}} {{node.get_distro_series()}} main"]
  {{endif}}
  driver_04_update_install: ["curtin", "in-target", "--", "apt-get", "update", "--quiet"]
  {{if driver['package']}}
  driver_05_install: ["curtin", "in-target", "--", "apt-get", "-y", "install", "{{driver['package']}}"]
  {{endif}}
  driver_06_depmod: ["curtin", "in-target", "--", "depmod"]
  driver_07_update_initramfs: ["curtin", "in-target", "--", "update-initramfs", "-u"]
{{endif}}
```