---
title: Installation and Post-Install
parent: Proxmox Hardware Setup
nav_order: 1
---

# <i class="fas fa-compact-disc"></i> Installation and Post-Install
{: .no_toc }

<i class="fab fa-mixer" style="color: black"></i> Proxmox host setup
{: .label .label-proxhost }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Install Proxmox VE from ISO

Installing Proxmox VE from a Ventoy USB on bare metal is easy. Proxmox documentation with details is at [Proxmox Installation Documentation](https://pve.proxmox.com/pve-docs/chapter-pve-installation.html){:target="_blank"}.

### Make a bootable Ventoy USB drive

First, we need to create bootable USB drive with the Proxmox VE ISO. We will use Ventoy to do this.

The latest Ventoy installers are at [https://sourceforge.net/projects/ventoy/files/](https://sourceforge.net/projects/ventoy/files/){:target="_blank"}.

{: .important }
>The Ventoy USB can be created in Linux or Windows only. For **Mac**, use Parallels Windows VM or Linux VM.
>
> **After** you have created the Ventoy USB, you can **copy files to it** using your Mac or PC.

Important downloads and ISO images to install on Ventoy USB are below:

Downloads
{: .label .label-blue}

   | Download                                 | URL                                                  |  
   |:-------------------------------------|:-----------------------------------------------------|
   | Ventoy Installer                     | [https://sourceforge.net/projects/ventoy/files/](https://sourceforge.net/projects/ventoy/files/){:target="_blank"}|
   | System Rescue ISO                    | [https://www.system-rescue.org/Download/](https://www.system-rescue.org/Download/){:target="_blank"}|
   | Proxmox PVE and PBS ISOs             | [https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads){:target="_blank"}|
   | Ubuntu Server Live Install ISO       | [https://releases.ubuntu.com/](https://releases.ubuntu.com/)|
   | Ubuntu Server Cloud-Init Install ISO | [https://cloud-images.ubuntu.com/noble/current/](https://cloud-images.ubuntu.com/noble/current/){:target="_blank"}|
   | Ubuntu Desktop ISO                   | [https://ubuntu.com/download/desktop/](https://ubuntu.com/download/desktop/){:target="_blank"}|

### Boot from USB Drive and Install

If you need a walkthough on how to boot from USB and install, there's one at [https://medium.com/@r00tb33r/how-to-install-proxmox-a-step-by-step-guide-5d00439bc551](https://medium.com/@r00tb33r/how-to-install-proxmox-a-step-by-step-guide-5d00439bc551){:target="_blank"}.

## Proxmox post-install setup

Do
{: .label .label-green}

### Check that SSH is running

```shell
systemctl status ssh.service
```

### Learn about tteck's Proxmox VE scripts

#### Thank tteck

{: .note-title}
> Who was tteck?
>
> tteck was a volunteer who created a ton of scripts to install applications and environments within Proxmox. He did a lot to make Proxmox virtualization easy to use for mere mortals. He passed away November 2024.
>
> tteck’s Helper-Scripts are now managed by a group of commuity members.
>
> 
>
> {: .opaque .fs-2}
> <div markdown="block">
> {: .important-title }
> ***A message from tteck's wife, Angie:*** This is Angie, and to say I am overwhelmed by all the comments, thoughts, love, prayers, and condolences is an understatement. As well as the donations. My husband loved this site, and he was passionate about computers, coding, and all the stuff that goes with it. He showed me a lot of the comments that were made when he posted about his cancer and it made him smile. Thank you all for the love and support that you have shown to him, and now me.
>He would be amazed that I even found his sight and was able to make a post. Thank you all again, it warms my heart that he touched so many lives.
> </div>
> The Ko-Fi link is still available if you want to support Angie and tteck's family (unfortunately I never knew his name):
> [https://ko-fi.com/proxmoxhelperscripts](https://ko-fi.com/proxmoxhelperscripts){:target="_blank"}
> 
> <span class="fs-3">
> [Honor tteck's Contributions!](https://ko-fi.com/proxmoxhelperscripts){: .btn .btn-green}
> </span>

{: .note-title }
>See all tteck's Proxmox scripts
>
> You can download and install the Proxmox scripts from [https://community-scripts.github.io/ProxmoxVE/](https://community-scripts.github.io/ProxmoxVE/){:target="_blank"}
> 
><span class="fs-3">
>[Proxmox Scripts Webpage](https://community-scripts.github.io/ProxmoxVE/){: .btn }
> </span>
 
{: .important }
>View the Proxmox scripts source code at [https://github.com/community-scripts/ProxmoxVE/](https://github.com/community-scripts/ProxmoxVE/){:target="_blank"}
<span class="fs-3">
[Proxmox Scripts Github](https://github.com/community-scripts/ProxmoxVE/){: .btn }
</span>


#### Watch Techno Tim's video

Here is a great video from Techno Tim about the scripts:

<iframe width="560" height="315" src="https://www.youtube.com/embed/kcpu4z5eSEU?si=k5GkGNZ4zJZ-w1kf" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### Run tteck's Proxmox VE Post Install Script

{: .note }
> This script cleans up a lot of cruft after Proxmox installation. Specifically, "this script provides options for managing Proxmox VE repositories, including disabling the Enterprise Repo, adding or correcting PVE sources, enabling the No-Subscription Repo, adding the test Repo, disabling the subscription nag, updating Proxmox VE, and rebooting the system."

{: .warning }
Run tteck scripts from the **Proxmox GUI shell**, not SSH!

Do
{: .label .label-green}

```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/post-pve-install.sh)"
```

{: .note }
> You can view the "Proxmox VE Post Install" script source code at [https://community-scripts.github.io/ProxmoxVE/scripts?> id=post-pve-install](https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pve-install){:target="_blank"}

### Run tteck's Proxmox VE Processor Microcode Script

This procedure updates the microcode on your Intel or AMD processor, if required. Intel and AMD update processor microcode usually to fix bugs and security issues.

{: .warning }
Run tteck scripts from the **Proxmox GUI shell**, not SSH!

Do
{: .label .label-green}

```shell
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/microcode.sh)"
```

Then reboot:

```sh
reboot
```

Check whether any microcode updates are currently in effect:

```sh
journalctl -k | grep -E "microcode" | head -n 1
```


### Set up iKoolcore-specific Proxmox summary (OPTIONAL)

{: .note}
> These instructions are specific to my iKoolcore R2 server hardware. Don't do it if you aren't installing on one.

Follow the steps in the iKoolcore R2 wiki at [https://github.com/KoolCore/Proxmox_VE_Status](https://github.com/KoolCore/Proxmox_VE_Status){:target="_blank"}

Download
{: .label .label-blue}

Add iKoolcore R2 hardware stats to the Proxmox summary page by downloading and running this shell script that I modified here [https://github.com/kurtshuler/proxmox-ubuntu-server/blob/main/Proxmox%20files/Proxmox_VE_Status_zh.sh](https://github.com/kurtshuler/proxmox-ubuntu-server/blob/main/Proxmox%20files/Proxmox_VE_Status_zh.sh){:target="_blank"}

Do
{: .label .label-green}

```sh
cd Proxmox_VE_Status
```

```sh
bash ./Proxmox_VE_Status_zh.sh
```
