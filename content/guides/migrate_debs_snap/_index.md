+++
title = "How to migrate from deb to snap"
type = "chapter"
weight = 8
+++

# How to migrate your deb installation to snap 

NOTE: if you have custom images from 3.5 you need some extra steps that are not described here. So DO NOT USE THIS!


extract your db uri with 

```
#!/bin/bash

conf_file="/etc/maas/regiond.conf"

db_user=$(sudo grep '^database_user:' "$conf_file" | awk '{print $2}')
db_pass=$(sudo grep '^database_pass:' "$conf_file" | awk '{print $2}')
db_host=$(sudo grep '^database_host:' "$conf_file" | awk '{print $2}')
db_port=$(sudo grep '^database_port:' "$conf_file" | awk '{print $2}')
db_name=$(sudo grep '^database_name:' "$conf_file" | awk '{print $2}')

db_uri="postgres://${db_user}:${db_pass}@${db_host}:${db_port}/${db_name}"

echo "$db_uri"
```

and note it somewhere. Then you remove the MAAS debs entirely with 

```
sudo DEBIAN_FRONTEND=noninteractive apt remove --purge -y 'maas*' && sudo DEBIAN_FRONTEND=noninteractive apt -y autoremove
```

and then you 

```
sudo snap install maas --channel=3.x/stable
sudo maas init region+rack --database-uri "<THE DB URI YOU EXTRACTED BEFORE>"
```