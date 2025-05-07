+++
title = "Deployment"
type = "chapter"
weight = 2
+++

## What to do when your machine is not being deployed. 

If your machine is not PXE booting, please see [the dedicated section about PXE](/troubleshooting/pxe).

### DataSourceNotFoundException

This is the segfault for MAAS. Many bad things might have happened when you get this. Based on my experience, usually that means that the machine has failed to retrieve the preseed file from MAAS - hence you have some issues with DNS or DHCP. Please check the MAAS logs for more information about what has failed. Also, ensure you don't have a rogue DHCP server on the network. 

Another reason for this to happen is that there is a bug in cloud-init. This is not common, but it has happened from time to time. Usually we spot these bugs before the cloud-init package reaches the official Ubuntu images we distribute, but if you are building custom images you might face them. 

### Ensure you uploaded your custom images the right way. 

If you are deploying a custom image, you must ensure you uploaded it the right way. 

1) You must have used the correct base image name: if you built a Red Hat image, the name *must* be like `rhel/<whateverr>`. In short, it must start with `rhel/`. Here the table for all the operating systems

| OS    | name |
| -------- | ------- |
| Red Hat Enterprise Linux  | rhel/    |
| Oracle Linux | ol/     |
| CentOS    | centos/    |
| Azure Linux    | custom/    |
| Alma Linux    | rhel/    |
| Rocky Linux    | rhel/    |
| SUSE    | suse/    |
| ESXI    | esxi/    |
| Windows    | windows/    |
| XenServer    | custom/    |
| Ubuntu    | custom/    |

This is required to instruct MAAS about the type of image being deployed, because depending on the OS it must use some specific logic. 

2) If you are using MAAS 3.5 or above in HA mode, you must upload your custom image *to a single region controller*. You *must not* upload it through a load balancer or a VIP. In any case, ensure the custom image is properly in sync with all the region controllers in the MAAS UI. 

