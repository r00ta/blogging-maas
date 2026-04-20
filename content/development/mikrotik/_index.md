+++
title = "Deploy a MikroTik VM with QEMU and connect via API"
type = "chapter"
weight = 9
+++

These are my notes on how to deploy a MikroTik RouterOS virtual machine using QEMU and interact with it through the REST API.

## Prerequisites

- A Linux host with KVM support (`kvm-ok` should return "KVM acceleration can be used")
- `qemu-system-x86_64` installed (`sudo apt install qemu-system-x86`)
- A bridge network already configured (e.g. `maas-ctrl`)

## Download the MikroTik image

Download the x86 ISO image from the official MikroTik website:

https://mikrotik.com/download?architecture=x86

At the time of writing, the latest version is `7.20.8`.

## Create the virtual disk

Create a qcow2 disk image that will be used as the MikroTik hard drive:

```
qemu-img create -f qcow2 mikrotik_disk.qcow2 256M
```

## Install MikroTik RouterOS

Boot from the ISO to install RouterOS onto the virtual disk:

```
qemu-system-x86_64 -enable-kvm -m 256M \
  -hda mikrotik_disk.qcow2 \
  -cdrom ~/Downloads/mikrotik-7.20.8.iso \
  -boot d
```

The MikroTik installer will appear in the QEMU window:

![MikroTik installation screen](https://github.com/user-attachments/assets/36e8e44e-d109-4cd1-a331-1686ddb30148)

Use the arrow keys or `p`/`n` to navigate, `spacebar` to select packages. You can press `a` to select all or `m` for the minimum set. At a minimum, make sure **system** is selected.

Press `i` to install. When prompted, confirm the installation. Once it completes, close the QEMU window (the VM will ask you to reboot).

## Boot the MikroTik VM with bridge networking

Now boot the VM from the installed disk, attaching it to your bridge network:

```
sudo qemu-system-x86_64 -enable-kvm -m 256M \
  -hda mikrotik_disk.qcow2 \
  -netdev bridge,id=net0,br=maas-ctrl \
  -device virtio-net-pci,netdev=net0,mac=52:54:00:ab:cd:ef
```

{{% notice info %}}
`sudo` is required to attach to the bridge interface. The MAC address `52:54:00:ab:cd:ef` is arbitrary — you can change it as needed.
{{% /notice %}}

## Configure the IP address

Once RouterOS boots, log in with the default credentials (user: `admin`, no password). You will be prompted to set a new password.

Assign a static IP address to the interface (adjust the address and interface name to match your network):

```
/ip address add address=10.0.0.100/24 interface=ether1
```

Verify the configuration:

```
/ip address print
```

## Enable the REST API (HTTP service)

Starting from RouterOS v7, MikroTik includes a built-in REST API that runs on top of the HTTP service (`www`). Make sure the `www` service is enabled:

```
/ip service print
```

If the `www` service is disabled, enable it:

```
/ip service enable www
```

By default, the HTTP service listens on port `80`. You can verify or change the port:

```
/ip service set www port=80
```

{{% notice warning %}}
The REST API uses HTTP by default (not HTTPS). For production environments, consider enabling `www-ssl` instead and disabling plain HTTP.
{{% /notice %}}

## Make a GET request to the MikroTik REST API

From your host (or any machine on the same network), you can now query the MikroTik REST API. The API follows the same structure as the CLI commands, using the path hierarchy as URL paths.

For example, to get the list of IP addresses configured on the router:

```
curl -u admin:yourpassword http://10.0.0.100/rest/ip/address
```

Example response:

```json
[
  {
    ".id": "*1",
    "address": "10.0.0.100/24",
    "network": "10.0.0.0",
    "interface": "ether1",
    "actual-interface": "ether1",
    "invalid": "false",
    "dynamic": "false",
    "disabled": "false"
  }
]
```

You can also retrieve the system identity:

```
curl -u admin:yourpassword http://10.0.0.100/rest/system/identity
```

Or get the full system resource information:

```
curl -u admin:yourpassword http://10.0.0.100/rest/system/resource
```

{{% notice tip %}}
The REST API maps directly to the RouterOS CLI. Any command path like `/ip/firewall/filter` becomes the URL `http://<router-ip>/rest/ip/firewall/filter`. Use `GET` to read, `PUT` to create, `PATCH` to update, and `DELETE` to remove entries.
{{% /notice %}}
