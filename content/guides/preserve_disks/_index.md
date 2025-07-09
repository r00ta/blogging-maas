+++
title = "How to preserve disks when deploying"
type = "chapter"
weight = 6
+++

You have 2 ways to preserve disks when deploying a machine in MAAS: 

1) You pull out the disks before deploying the machine, and then you put them back after the deployment.
2) You remove the disks from MAAS before deploying the machine. This is because MAAS will wipe only the superblock of the disks it knows about.