---
title: Host Setup
parent: Proxmox GPU Passthrough
nav_order: 1
---

# Host Setup: Proxmox GPU Passthrough
{: .no_toc }

<i class="fab fa-mixer" style="color: black"></i> Proxmox host setup
{: .label .label-proxhost }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Warning: iGPU and VM Only!

{: .warning }
>These server-side or host steps are only for **Intel integrated GPU (iGPU)** passthrough. They will not work as written below with Intel discrete, AMD, or Nvidia GPUs.

{: .warning }
>These server-side or host steps are only been for passthrough to a **VM**, *not an LXC*.
>
> Anand has detailed instructions for LXC passthrough in his excellent article and video, [UDMS Part 11: GPU passthrough on Proxmox LXC for Superior HW Transcoding!](https://www.simplehomelab.com/udms-11-gpu-passthrough-on-proxmox-lxc/){:target="_blank"} 

{: .important }
>These server-side or host steps can be done before or after you have created a VM.

## Source Information

{: .note-title }
> Sources:
>1. iKoolcore R2 script code at [https://github.com/KoolCore/Proxmox_VE_Status/blob/main/passthrough.sh](https://github.com/KoolCore/Proxmox_VE_Status/blob/main/passthrough.sh){:target="_blank"} 
>2. Craft Computing's [Proxmox 8.0 - PCIe Passthrough Tutorial](https://www.youtube.com/watch?v=_hOBAGKLQkI){:target="_blank"} video is the best explainer I have seen of this process.
>3. [Proxmox PCI(e) Passthrough documentation](https://pve.proxmox.com/wiki/PCI(e)_Passthrough){:target="_blank"}
{: .fs-3 }

I created the steps manually below from the script source code at [https://github.com/KoolCore/Proxmox_VE_Status/blob/main/passthrough.sh](https://github.com/KoolCore/Proxmox_VE_Status/blob/main/passthrough.sh){:target="_blank"}.

The Craft Computing video, [Proxmox 8.0 - PCIe Passthrough Tutorial](https://www.youtube.com/watch?v=_hOBAGKLQkI){:target="_blank"}, follows these steps almost exactly, with the only difference being the changes to `/etc/kernel/cmdline`.

<iframe width="560" height="315" src="https://www.youtube.com/embed/_hOBAGKLQkI?si=gKDLccL8bXHdAn2S" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


## Make IOMMU changes at boot

{: .important }
>There are two possible boot systems, **Systemd (EFI)** or **Grub**.
>
>The **'Boot Mode'** in the Proxmox GUI summary page for a node (like `pve`) indicates whether it is EFI (systemd) or Grub booted.


Do
{: .label .label-green}

### For Grub boot, edit `/etc/default/grub`

1. Open `/etc/default/grub`

    ``` sh
    nano /etc/default/grub
    ```

2. Change this line to:

    ```sh
    GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
    ```

3. Save file and close

4. Run:

    ```sh
    update-grub
    ```

### For Systemd (EFI) boot, edit `/etc/kernel/cmdline`

1. Open `/etc/kernel/cmdline`
   
    ```sh
    nano /etc/kernel/cmdline
    ```

2. Add this to first line:
   
    ```sh
    intel_iommu=on iommu=pt
    ```

    {: .note }
    >All commands in `/etc/kernel/cmdline` must be in a **single line** on the **first line!**

3. Save file and close

4. Run:
   
    ```sh
    proxmox-boot-tool refresh
    ```

## Load VFIO modules at boot

1. Open `/etc/modules`
   
    ```sh
    nano /etc/modules
    ```

2. Add these lines:
    ```sh
    vfio
    vfio_iommu_type1
    vfio_pci
    vfio_virqfd
    ```

3. Save file and close

## Configure VFIO for PCIe Passthrough

1. Find your GPU PCI identifier

    ```sh
    lspci
    ```
    It will be something like `00:02`

2. Find your GPU's PCI HEX values

    Enter the PCI identifier (`00:02`) from above into the `lspci` command: 

    ```
    lspci -n -s 00:02 -v
    ```

    You will see an associated HEX value like `8086:46d0`

3. Edit `/etc/modprobe.d/vfio.conf`

    Copy the HEX values from your GPU into this command and hit enter:

    ```sh
    echo "options vfio-pci ids=8086:46d0 disable_vga=1"> /etc/modprobe.d/vfio.conf
    ```

4. Apply all changes
    ```sh
    update-initramfs -u -k all
    ```

## Blacklist Proxmox host device drivers

{: .note }
> This ensures nothing else on the Proxmox host can use the GPU that you want to pass through to a VM.

1. Edit `/etc/modprobe.d/iommu_unsafe_interrupts.conf`

    ```sh
    echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
    ```

2. Edit `/etc/modprobe.d/blacklist.conf`

    ```sh
    echo "blacklist i915" >> /etc/modprobe.d/blacklist.conf
    echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
    echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
    ```

3. Apply all changes

    ```sh
    update-initramfs -u -k all
    ```

4. Reboot to apply all changes

    ```sh
    reboot
    ```

## Verify all changes on the Proxmox host

1. Verify `vfio-pci` kernel driver being used:

    ```sh
    lspci -n -s 00:02 -v
    ```

    In the output, you should see:

    ```yaml
    Kernel driver in use: vfio-pci
    ```

2. Verify IOMMU is enabled:

    ```sh
    dmesg | grep -e DMAR -e IOMMU
    ```

    In the output, you should see:

    ```yaml
    DMAR: IOMMU enabled
    ```

3. Verify IOMMU interrupt remapping is enabled:

    ```sh
    dmesg | grep 'remapping'
    ```

    In the output, you should see something like:

    ```yaml
    DMAR-IR: Enabled IRQ remapping in x2apic mode
    ```

4. Verify that Proxmox recognizes the GPU:

    ```sh
    lspci -v | grep -e VGA
    ```

    In the output, you should see something like:

    ```yaml
    00:02.0 VGA compatible controller: Intel Corporation Alder Lake-N [UHD Graphics] (prog-if 00 [VGA controller])
    ```

5. To get more details about your GPU VGA controller:

    ```sh
    lspci -v -s 00:02.0
    ```

## Next Step: Setup passthrough within a VM

Once we have confirmed GPU passthough works on the Proxmox VE host, we have to enable it within a VM to use it. 
- To create a VM, perform the steps in [Creating a Ubuntu VM]({% link docs/create-ubuntu-vm/index.md %}) 
- After you have a working VM, perform the steps in [Proxmox GPU (PCI) Passthrough: VM Setup]({% link docs/proxmox-gpu-passthrough/vm-setup.md %}).



