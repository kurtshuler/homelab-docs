---
title: OS Prep - Users, Packages, and SSH
parent: Create Linux VM
nav_order: 2
---

# OS Prep: Users, Packages, and SSH
{: .no_toc }

<i class="fab fa-ubuntu"></i> Ubuntu OS setup
{: .label .label-ubuntu }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Source

These steps are based on Anand's OS setup guide at [Ultimate Docker Server: Getting Started with OS Preparation Part 1](https://www.simplehomelab.com/ultimate-docker-server-1-os-preparation/){:target="_blank"}. 

- The steps below are adapted from this guide to match Ubuntu 24.04 SSH requirements.

- These steps will prepared us to install Docker later using Anand's excellent [SimpleHomelab Deployarr](https://github.com/SimpleHomelab/deployarr){:target="_blank"} application.

Do
{: .label .label-green}

## Create new user and give sudo privileges

```sh
adduser kurt
adduser kurt sudo
```

## Update Ubuntu OS

```sh
sudo apt update
sudo apt upgrade
```

## Install basic and required packages

```sh
sudo apt install sudo linux-generic ca-certificates curl gnupg lsb-release ntp htop zip unzip gnupg apt-transport-https ca-certificates net-tools ncdu apache2-utils git neofetch vsftpd mc
```

{: .warning }
> `linux-generic` is required to enable GPU passthrough from Proxmox to the Ubuntu VM. It is not included in the Ubuntu Cloud-Init distribution used by the Tteck Ubuntu installation script.

## Change SSH port to 2053

{: .note }
> These instructions are different than what is on Anand's site due to changes in Ubuntu 24.04 SSH.


### Change `systemd` configuration for `ssh.socket`

To change the port of the SSH server, the systemd configuration for ssh.socket must be changed or supplemented. The configuration adjustment is made by editing the socket using `systemctl`.

1. Edit `ssh.socket` file:

    ```sh
    systemctl edit ssh.socket
    ```

2. Add the following lines at the top under the two initial commented lines:
    
    ```yaml
    [Socket]
    ListenStream=
    ListenStream=2053
    ```

    {: .note }
    > The blank line `ListenStream=` is required to ensure that port 22 is no longer used. Without this line, the SSH server would then be accessible via port 22 (default) *and* 2053.
 
 3. Restart SSH:

    ```sh
    systemctl daemon-reload
    sudo systemctl restart ssh  
    ```

4. Reboot the VM:

    ```sh
    reboot 
    ```
    
## Check our work

1. Ensure that port 22 is *closed* and port 2053 is *open*:

    ```sh
    netstat -ant | grep 2053
    ```

    ```sh
    sudo lsof -nP -iTCP -sTCP:LISTEN
    ```

    ```sh
    sudo netstat -tunpl
    ```

2. SSH `root` check: Log into Ubuntu server using new SSH port 2053

    ```sh
    ssh root@192.168.1.100 -p 2053
    ```

3. SSH `user` check: Log into Ubuntu server using new SSH port 2053

    ```sh
    ssh kurt@192.168.1.100 -p 2053
    ```