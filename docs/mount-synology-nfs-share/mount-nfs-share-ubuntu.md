---
title: Mount NFS Share in Ubuntu
parent: Mount Synology NFS Share
nav_order: 2
---

# Mount NFS Share in Ubuntu
{: .no_toc }

<i class="fab fa-ubuntu"></i> Ubuntu OS setup
{: .label .label-ubuntu }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Source Information

There are many online walkthroughs on how to create a Synology NFS share, get its mount point for use in a Linux client's `fstab`, and implement `fstab` for mounting. One of the better ones I have found is [Setting up NFS on a Synology NAS at PiMyLifeUp](https://pimylifeup.com/synology-nas-nfs/){:target="_blank"}. The information below is based on multiple source and describes the implementation on one of my servers.

{: .note }
> We will be using Ubuntu Linux `fstab` so that our Linux OS VM will mount the Synology NFS share every time the OS boots.

## Mount Synology NAS shared NFS drive using `mount` and `fstab`

Do
{: .label .label-green}

### Install `nfs-common` for NFS client support

```sh
sudo apt install nfs-common
```

### Create the mount point and `mount`

1. Make the mount point:

    In this example, `/mnt/nas` will be the `[MOUNTPATH]` in the `mount` command, below.

    ```sh
    mkdir /mnt/nas
    ```

2. Mount your Synology NAS volume to the mount point:

    ```sh
    mount nas.kurtshuler.net:/volume1/data /mnt/nas
    ```

    This is the syntax:

    ```sh
    mount [SYNOLOGY-IPADDRESS]:[SYNOLOGY-REMOTEVOLUME] [MOUNTPATH]
    ```

3. Verify the mount is correct:

    ```sh
    df -HT
    ```

4. Create a file to test the mount

    ```sh
    cd /mnt/nas
    touch test.txt
    ```

5. Check your Synology NAS NFS share folder to confirm the file is in it.

## Update `/etc/fstab` 

We will add our new mount to the Linux OS's `fstab` file so the Synology shared drive will always be mounted at boot time.


1. Open `fstab`:

    ```sh
    nano /etc/fstab
    ```

2. Append this line:

    ```sh
    nas.kurtshuler.net:/volume1/data /mnt/nas nfs defaults        0       0
    ```

3. Reboot and check

    ```sh
    reboot
    ```

4. Verify the mount is correct:

    ```sh
    df -HT
    ```