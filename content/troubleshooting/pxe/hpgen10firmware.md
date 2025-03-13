+++
date = '2025-03-14T00:25:12+01:00'
title = 'HP Gen10 initramfs error'
tags = ["bug", "initramfs", "HP", "Gen10", "BCM5719", "PXE", "firmware"]
+++

## Symptoms
If you are using a HP Gen10 server with a `BCM5719 Gigabit Ethernet PCIe Adapter` with the version `5719-v1.46 NCSI v1.5.1.0`, then you will not be able to pxe boot with MAAS. The symptom is that after getting the IP address from the MAAS DHCP server, you will be sent to the `initramfs` prompt. 


## Solution

Upgrade your firmware and do not blame MAAS :) 