---
title: VM Setup
parent: Proxmox GPU Passthrough
nav_order: 2
---

# VM Setup: Proxmox GPU (PCI) Passthrough
{: .no_toc }

VM setup
{: .label .label-purple }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Warning: VM and iGPU only!

{: .warning }
>These VM steps are only for use within a **VM**. See previous section for Proxmox host-side GPU passthrough setup.
>
>These VM steps are only been for passthrough to a **VM**, *not an LXC*.
>
> Anand has detailed instructions for LXC passthrough in his excellent article and video, [UDMS Part 11: GPU passthrough on Proxmox LXC for Superior HW Transcoding!](https://www.simplehomelab.com/udms-11-gpu-passthrough-on-proxmox-lxc/){:target="_blank"}


{: .warning }
>These VM steps are only for **Intel integrated GPU (iGPU)** passthrough. They will not work as written below with Intel discrete, AMD, or Nvidia GPUs.

{: .warning }
>These steps are for a **Linux OS** running in the VM, specifically Ubuntu. For enabling passthrough in a Windows VM, see Derek Seaman's great tutorial, [Proxmox VE 8.3: Windows 11 vGPU (VT-d) Passthrough with Intel Alder Lake](https://www.derekseaman.com/2024/07/proxmox-ve-8-2-windows-11-vgpu-vt-d-passthrough-with-intel-alder-lake.html){:target="_blank"}




## Source Information

{: .note-title }
> Sources:
>1. Craft Computing's [Proxmox 8.0 - PCIe Passthrough Tutorial](https://www.youtube.com/watch?v=_hOBAGKLQkI){:target="_blank"} video is the best explainer I have seen of this process.
>2. [Proxmox PCI(e) Passthrough documentation](https://pve.proxmox.com/wiki/PCI(e)_Passthrough){:target="_blank"}
{: .fs-3 }

<iframe width="560" height="315" src="https://www.youtube.com/embed/_hOBAGKLQkI?si=gKDLccL8bXHdAn2S" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## In Proxmox GUI, add GPU to VM PCI devices

{: .note }
> These steps can only be done after you have created a VM and only affect that VM.

1. In Proxmox GUI, open the VM's "PCI Device" settings:

    `pve` &rarr; `[VM#]` &rarr; `Hardware` &rarr; `Add` &rarr; `PCI Device`
    ![images](/../../assets/images/iGPU-passthrough-add-pci-device-button.png)
2. In popup, select the following:

    ```yaml
    Raw Device: YES
    Device: Select your GPU
    ```

    Then click the following:
    ```yaml
    All Functions: YES
    ROM-Bar: YES
    Primary GPU: YES
    PCI-Express: YES (requires 'machine: q35' in VM config file)
    ```
    ![images](/../../assets/images/iGPU-passthrough-add-pci-device-button-screen.png){:width="75%"}
3. Check the results
    ![images](/../../assets/images/iGPU-passthrough-add-pci-device-check.png)

## Turn off VM Display

1. Turn off the VM "Display" so it will not use the GPU hardware that we have passed through:

    `pve` &rarr; `[VM#]` &rarr; `Hardware` &rarr; `Display` &rarr; `Edit` &rarr; `Graphic Card` &rarr; `none`

    {: .note }
    > This will mean that the VM ">_ Console" button on the left sidebar will no longer work. However, you can still access the VM terminal by using the ">_ Console" button on the top nav bar or through SSH.
2. Reboot the VM

## Verify iGPU hardware passthough is working in the Ubuntu VM

Run the following commands to confirm your GPU hardware is available for use by your Ubuntu VM.

1. Check to see if your VGA adapter is available:

    ```sh
    lspci -k | grep VGA
    ```

    ```
    lspci -n -s 01:00 -v
    ```

    ```sh
    lspci -nnv | grep VGA
    ```

2. Check to confirm `Kernel driver in use: i915`:

    ```sh
    lspci -n -s 01:00 -v
    ```

3. Check to see that you have `renderD128` in `/dev/dri`

    ```sh
    ls -l /dev/dri/by-path/
    ```

That's it. If all the above check out, your GPU is available within your Ubuntu VM to use for functions like Plex HW transcoding.