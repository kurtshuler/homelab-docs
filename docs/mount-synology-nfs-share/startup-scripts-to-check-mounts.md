---
title: Startup Scripts to Check Mounts
parent: Mount Synology NFS Share
nav_order: 3
---

# Startup Scripts to Check Mounts
{: .no_toc }

<i class="fab fa-ubuntu"></i> Ubuntu OS setup
{: .label .label-ubuntu }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Motivation

I have sometimes had issues where my Linux OS did not mount the Synology NAS NFS share folders at startup, sometimes due to NAS issues and other times due to Linux OS or VM issues. To help address this, I run two scripts that check my two mounted NAS NFS shares every time my computer boots.

### Benefit #1: Check mounts before Docker tries to access them

The big issue when this happens is that I will have issues with my Docker media containers and not know why. The second script helps ensure my Synology NAS NFS media shares are connected properly **before** Docker containers that use them are started.

### Benefit #2: Safer recovery from power loss

The second script automatically starts my media Docker compose at startup once their NFS shares are connected. This allows for an unattended full restart after power losses.

{: .note-title}
> Source
>
> My scripts are derived from [Anand's SimpleHomelab scripts](https://github.com/SimpleHomelab/docker-traefik/tree/master/scripts/hs){:target="_blank"}.

## Two files to check mounts at startup

I run two different files at startup by including them at the end of my `.bashrc` file.

I store them in my `/usr/local/bin/` directory.

| File        | Location         | Purpose |
|:-------------|:------------------|:------|
| `check-mounts.sh` | `/usr/local/bin/` | Gives a simple text confirmation that each share is mounted.  |
| `start-media-after-boot.sh` | `/usr/local/bin/` | Confirms shares are mounted before automatically running Docker media compose at startup. Allows safe restart after power loss and reboot. |

## Why `.bashrc` to run these scripts at boot time?

I tried using `cron` and `systemd` but could not get this to work properly. Eventually, I simply added the scripts to the end of my `.bashrc` file and everything has worked.

In the `.bashrc` file, I use the following text to run them at startup:

```bash
source /usr/local/bin/check-mounts.sh
source /usr/local/bin/start-media-after-boot.sh
```

Having them in the `.bashrc` also makes it easier for me to find the script files later to edit them or run them manually.

## Create and use `check-mounts.sh`

### `check-mounts.sh` sample output

```sh
/mnt/nas mounted.
/mnt/paperless mounted.
All drives mounted.
```

Do
{: .label .label-green}

### `check-mounts.sh` source code

Here is the source code for the version of this script running on my Docker media server.

```bash
#!/bin/bash
# This script checks for all required drive mounts. It is stripped down source code from
# Anand's SimpleHomelab at https://github.com/SimpleHomelab/docker-traefik/tree/master/scripts/hs

# Add your mount points here
hdd1="/mnt/nas"
hdd2="/mnt/paperless"
num_drives=2

mounted=0
notmounted=""

# Check if first mount point is connected and echo status 
if mount | grep ${hdd1} > /dev/null; then
	mounted=$((mounted+1))
	echo -e ${hdd1}' mounted.'
else 
	echo -e ${hdd1}' not mounted.'
	notmounted="${notmounted} ${hdd1}\n"
fi

# Check if second mount point is connected and echo status 
if mount | grep ${hdd2} > /dev/null; then
	mounted=$((mounted+1))
	echo -e ${hdd2}' mounted.'
else 
	echo -e ${hdd2}' not mounted.'
	notmounted="${notmounted} ${hdd2}\n"
fi

# Echo status summary
if [[ "$mounted" -ne "$num_drives" ]]; then
	echo -e "These drives are not mounted properly:\n\n${notmounted}\n\." 
else
	echo "All drives mounted." 
fi
```


## Create and use `start-media-after-boot.sh`

Do
{: .label .label-green}

### `start-media-after-boot.sh` source code

```bash
#!/bin/bash
# All containers (profile "media") that access NFS mounts are set to NOT restart automatically at boot
# time. This is because, it can take a few seconds to mount remote drives.
# This script checks the required mounts every 5 seconds and as soon as required drives are mounted, it starts the "media" containers.
#
# This script is stripped down source code from Anand's SimpleHomelab at
# https://github.com/SimpleHomelab/docker-traefik/tree/master/scripts/hs

# CHECKING FOR DRIVE MOUNTS
num_drives=2 # number of mounts to check
# Drive 1 - UDMS media drive on Synology NAS
drive1="/mnt/nas"
drive1_seconds=0
drive1_status=0
# Drive 2 - Paperless docs drive on Synology NAS
drive2="/mnt/paperless"
drive2_seconds=0
drive2_status=0

mounted=0
rounds=0

while [[ "$mounted" -ne "$num_drives" ]]; do
	echo -e "round $rounds\n" # for testing
	if [[ "$(systemctl is-active docker)" == "active" ]]; then
		# Drive 1
		if mount | grep ${drive1} > /dev/null; then
			if [[ "$drive1_status" -eq 0 ]]; then
				mounted=$((mounted+1))
				drive1_seconds=$((rounds * 5))
				drive1_status=1
			fi
		fi
		# Drive 2
		if mount | grep ${drive2} > /dev/null; then
			if [[ "$drive2_status" -eq 0 ]]; then
				mounted=$((mounted+1))
				drive2_seconds=$((rounds * 5))
				drive2_status=1
			fi
		fi
		# Timeout if mounting is not successful after 15 min (180x5)
		if [[ $rounds -eq 180 ]]; then
		break
		fi
		sleep 5
		rounds=$((rounds + 1))
	fi
done

seconds=$((rounds * 5))
echo "seconds $seconds" # for testing

# Check mount status, echo a notification, and bring Docker up or down
if [[ "$mounted" -eq "$num_drives" ]]; then
	# echo "equal"
	echo -e "### `date +'%Y-%m-%d %H:%M'` ### \n\n All drives mounted: \n\n - $drive1 in $drive1_seconds\n\n - $drive2 in $drive2_seconds\n\nStarting media containers."
	docker compose --profile media -f /home/kurt/docker/docker-compose-udms.yml up -d
else
	# echo "not equal"
	echo -e "### `date +'%Y-%m-%d %H:%M'` ### \n\n Not all drives mounted after reboot: \n\n - $drive1 is $drive1_status\n\nTimed out after $seconds seconds." | mail -s "[$HOSTNAME] Mounted not equal to $num_drives."
	docker compose --profile media -f /home/kurt/docker/docker-compose-udms.yml down
fi
```