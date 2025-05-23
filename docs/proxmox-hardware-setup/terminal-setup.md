---
title: Terminal Setup
parent: Proxmox Hardware Setup
nav_order: 2
---

# <i class="fas fa-terminal"></i> Terminal Setup
{: .no_toc }

<i class="fab fa-mixer" style="color: black"></i> Proxmox host setup
{: .label .label-proxhost }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .note }
These instructions are for setting up the terminal for the ***Proxmox bare metal server***, not a Proxmox VM or LXC. I use additional bash aliases from [Anand's SimpleHomelab](https://github.com/SimpleHomelab/docker-traefik/blob/master/shared/config/bash_aliases){:target="_blank"} when I have a Linux VM or LXC running Docker.

Do
{: .label .label-green}

## Install Neofetch

```shell
sudo apt install neofetch
```

## Install Oh My Bash

```shell
bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)"
```

Reload `.bashrc`

```shell
source ~/.bashrc
```

If there is an error loading OMB, set the proper OMB file location in `.bashrc`

```shell
export OSH='/root/.oh-my-bash'
```

## Add plugins and completions to `.bashrc`

You can edit your `.bashrc` by copying and comparing to my GitHub Proxmox [`.bashrc`](https://github.com/kurtshuler/proxmox-ubuntu-server/blob/main/Proxmox%20files/.bashrc){:target="_blank"}

```shell
nano .bashrc
```

Reload `.bashrc`

```shell
source ~/.bashrc
```

## Install iTerm shell integration:

In the iTerm 2 GUI, click on `iTerm2 → Iterm Shell Integration`