---
title: OS Firewall and Performance Tweaks
parent: Create Linux VM
nav_order: 4
---

# OS Firewall and Performance Tweaks
{: .no_toc }

Ubuntu OS setup
{: .label .label-red }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Do
{: .label .label-green}

## Enable UFW firewall

Apply UFW settings:

```sh
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.0.0/24
sudo ufw allow 80
sudo ufw allow 443
```

Activate UFW using the command:

```sh
sudo ufw enable
```

Check  UFW status
```sh
sudo ufw status
```

##  Make performance tweaks

{: .note }
> This is based on Anand's OS setup guide at [Ultimate Docker Server: Getting Started with OS Preparation Part 1](https://www.simplehomelab.com/ultimate-docker-server-1-os-preparation/).

These are system configuration tweaks to enhance the performance and handling of large lists of files (e.g. Plex/Jellyfin metadata).

Edit `/etc/sysctl.conf` using the following command:

```shell
sudo nano /etc/sysctl.conf
```

Add the following 3 lines at the end of the file:

```python
vm.swappiness=10
vm.vfs_cache_pressure=50
fs.inotify.max_user_watches=262144
```