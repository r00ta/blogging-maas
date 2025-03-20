+++
date = '2025-03-13T23:27:20+01:00'
type = "chapter"
title = 'PXE'
weight = 2
+++

## What to do when your machine is not PXE booting

Here you can find the most common causes for your machine not PXE booting. If still you don't find the root problem, you can look at the other issues in the dropdown menu. 

### Of course, you must have enabled DHCP on the VLAN

This is self-explanatory, but it's always good to remind. 

### Rogue DHCP server

This is one of the most common problems when you fail do have failures in MAAS: *you must have only ONE DHCP server on one broadcast domain/VLAN*. If you have multiple DHCP servers, you'll likely see dragons. In order to check it, on the rack controller you can take a `tcpdump` and inspect the packets: if you see multiple DHCP offers, then it means that you have multiple DHCP servers on the network. 

### Images should be available in MAAS

The image that you are trying to deploy *must* be available in MAAS. Before 3.4, the images must be in sych with all the rack controllers, from 3.5 the images must be available in all the regions (the racks will only serve as a cache). 

### Bug in BCM5719 firmware (HP Gen10)

If you are using a HP Gen10 server with a `BCM5719 Gigabit Ethernet PCIe Adapter` with the version `5719-v1.46 NCSI v1.5.1.0`, then you will not be able to pxe boot with MAAS. The symptom is that after getting the IP address from the MAAS DHCP server, you will be sent to the `initramfs` prompt. Please upgrade your firmware. 

### Check your cables!

Again, self-explanatory.