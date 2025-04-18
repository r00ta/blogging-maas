+++
title = "Cloud-init"
type = "chapter"
weight = 1
+++

Here's some tricks when you have to deal with cloud-init during development and debugging.

## Use a specific clooud-init version on a deployed machine

You can change the version of cloud-init on a deployed machine by adding the file `/var/snap/maas/current/preseeds/curtin_userdata` and adding for example the following lines:

```
#cloud-config
debconf_selections:
 maas: |
  {{for line in str(curtin_preseed).splitlines()}}
  {{line}}
  {{endfor}}
late_commands:
  maas: [wget, '--no-proxy', {{node_disable_pxe_url|escape.json}}, '--post-data', {{node_disable_pxe_data|escape.json}}, '-O', '/dev/null']
  cloud_init_00: ["curtin", "in-target", "--", "add-apt-repository", "-y", "ppa:cloud-init-dev/daily"]
  cloud_init_01: ["curtin", "in-target", "--", "apt-get", "install", "-y", "cloud-init"]
```

You can change them accordingly in order to use a specific version of cloud-init. Then, you just need to deploy the machine.

