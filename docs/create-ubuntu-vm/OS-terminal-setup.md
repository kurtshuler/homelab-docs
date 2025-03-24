---
title: OS Terminal Setup
parent: Create Linux VM
nav_order: 3
---

# OS Terminal Setup
{: .no_toc }

Ubuntu OS setup
{: .label .label-red }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
These instructions are for setting up the terminal for a Linux OS, like Ubuntu, running in a Proxmox VM or LXC.

I use additional bash aliases from [Anand's SimpleHomelab](https://github.com/SimpleHomelab/docker-traefik/blob/master/shared/config/bash_aliases){:target="_blank"} when I have a Linux VM or LXC running Docker. These aliases greatly simplify Docker management and are installed using Anand's excellent [SimpleHomelab Deployarr](https://github.com/SimpleHomelab/deployarr){:target="_blank"} application.

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