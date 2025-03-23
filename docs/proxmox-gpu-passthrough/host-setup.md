---
title: Host Setup
parent: Proxmox GPU Passthrough
nav_order: 1
---

# Host Setup: Proxmox GPU Passthrough
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .important }
>These server-side or host steps can be done before or after you have created a VM.


## Source Information

{: .note-title }
> Sources:
>1. iKoolcore R2 script code at [https://github.com/KoolCore/Proxmox_VE_Status/blob/main/passthrough.sh](https://github.com/KoolCore/Proxmox_VE_Status/blob/main/passthrough.sh){:target="_blank"} 
>2. Craft Computing video at [https://www.youtube.com/watch?v=_hOBAGKLQkI](https://www.youtube.com/watch?v=_hOBAGKLQkI){:target="_blank"}
{: .fs-3 }

I created the steps manually below from the script source code at [https://github.com/KoolCore/Proxmox_VE_Status/blob/main/passthrough.sh](https://github.com/KoolCore/Proxmox_VE_Status/blob/main/passthrough.sh){:target="_blank"}.

Executing the iKoolcore R2 hardware passthrough script as-is at [https://github.com/KoolCore/Proxmox_VE_Status](https://github.com/KoolCore/Proxmox_VE_Status){:target="_blank"} did not work for me. It ran but didn't make the file changes.

Also, the Craft Computing video at [https://www.youtube.com/watch?v=_hOBAGKLQkI](https://www.youtube.com/watch?v=_hOBAGKLQkI){:target="_blank"} follows these steps almost exactly, with the only difference being the changes to `/etc/kernel/cmdline`.

<iframe width="560" height="315" src="https://www.youtube.com/embed/_hOBAGKLQkI?si=gKDLccL8bXHdAn2S" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


## Make IOMMU changes at boot

{: .note }
>There are two possible boot systems, Systemd (EFI) or Grub.
>
>The **'Boot Mode'** in the Proxmox GUI summary page for a node (like `pve`) indicates whether it is EFI (systemd) or Grub booted.

I chose to do both the Grub and EFI steps below.


### For Grub boot, edit `/etc/default/grub`
Do
{: .label .label-green}
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

This ensures nothing else on Proxmox can use the GPU that you want to pass through to a VM.

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

Once we have confirmed GPU passthough works on the Proxmox VE host, we have to enable it within a VM to use it. Those steps are in the next section.