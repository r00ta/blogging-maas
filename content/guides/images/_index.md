+++
title = "Managing images"
type = "chapter"
weight = 3
+++

## Tricks to manage the images in MAAS

Here's some tricks to manage your images. 

### Create your own mirror

Please follow https://discourse.maas.io/t/how-to-mirror-maas-images/5927 . 

### Rollback to a previous version of an image

MAAS can only use the latest greatest image available on your mirror. But what can you do if the latest greatest image on `http://images.maas.io` broke all your deployments? The only salvation for you is to use an older version of the image, but how do you do that? 

The answer is simple: you need to create a new mirror and patch the simplestream manifest. 

#### Setup the environment

For simpliciy, I'll use LXD to create a container and setup the mirror. 

```
lxc launch ubuntu:24.04 maas-images
```
and then
```
lxc shell maas-images
```

Install the required packages to expose a webserver on the port 80 of the container:
```
apt-get update && apt-get install apache2
```

Create the base directories

```
mkdir -p /var/www/html/maas/images/ephemeral-v3/stable
```

#### Setup the base mirror

First, download all the bootloaders

```
sudo mkdir -p /var/www/html/maas/images/ephemeral-v3/stable
cd /var/www/html/maas/images/ephemeral-v3/stable
wget -r -nH --cut-dirs=2 --no-parent --reject="index.html*" https://images.maas.io/ephemeral-v3/stable/bootloaders/
```

Now, say you are interested in the noble AMD64 images in the stable channel. 

```
cd /var/www/html/maas/images/ephemeral-v3/stable
wget -r -nH --cut-dirs=2 --no-parent --reject="index.html*" https://images.maas.io/ephemeral-v3/stable/noble/amd64/
```

Download the simplestream manifest

```
cd /var/www/html/maas/images/ephemeral-v3/stable
wget -r -nH --cut-dirs=2 --no-parent --reject="index.html*,*.sjson,*.gpg" https://images.maas.io/ephemeral-v3/stable/streams/
```

#### Patch the simplestream manifest

You'll need a gpg key, so 

```
gpg --full-generate-key
```

and insert all the required info (you can also use the default values, you'll just need to insert your name and email).

Now, edit `com.ubuntu.maas:stable:v3:download.json` and remove all the occurrences for the image that is causing the problem. Please ensure that the edited file is still a valid json!

Then, you'll have to sign the files with 

```
cd /var/www/html/maas/images/ephemeral-v3/stable/streams/v1
sed -i 's/\.json/\.sjson/g' index.json
for file in *.json; do sudo gpg --clearsign -u <your email in the gpg key> --output "${file%.json}.sjson" "$file"; done
```

At this point, when you point your MAAS to your LXD container, you have to provide also the keyring data. 

Execute
```
gpg --output public.gpg --export -u <your email in the gpg key>
```

and finally 

```
cat public.gpg | base64
```

At this point, open the MAAS UI and use `http://<the container ip>/maas/images/ephemeral-v3/stable/` and paste the base64 of the key in the `keyring_data`. 

If after you updated the source the images are stuck downloading, please open the MAAS regiond logs to understand what's going on.