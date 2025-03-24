---
title: Proxmox GPU Passthrough
nav_order: 3
---

# Proxmox GPU Passthrough
{: .no_toc }

Proxmox host setup
{: .label .label-yellow }

VM setup
{: .label .label-purple }

This section describes Intel iGPU Hardware Passthrough from the Proxmox host to a virtual machine (VM). Steps need to be performed in two (2) separate systems on the Proxmox server:
{: .fs-6 .fw-300 }

1. On the Proxmox VE host server
2. Within each desired VM running on the Proxmox host server
{: .fs-6 .fw-300 }

Don't lose your place and forget which target you are working on, (1) bare metal Proxmox server or a (2) VM hosted on it, or you'll screw things up!