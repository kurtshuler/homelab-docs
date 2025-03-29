---
title: NUT Client - Proxmox Host
parent: NUT UPS Monitor
nav_order: 3
---

# <i class="fab fa-mixer" style="color: #D6762C"></i> NUT Client - Proxmox Host
{: .no_toc }

<i class="fab fa-mixer" style="color: black"></i> Proxmox host setup
{: .label .label-proxhost }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## <i class="fas fa-table"></i> Summary table of NUT Client - Proxmox Host files

These files are for my NUT Clients that are runnng Proxmox PVE systems (pve10 and pve20). 

The Synology NAS Client is set up with different code.

{: .important-title }
> Don't edit your VM with this code!
>
> All of these files are at the BARE METAL level on the Proxmox host.
>
> This is a reminder because I keep looking for the NUT files in my VMs...😏

All my files are in `etc/nut`.

| File              | Purpose                                                  | File Directory      | On Server? | On Client? |
|-------------------|-----------------------------------------------------------|---------------------|------------|------------|
| `upsmon.conf`     | Configures UPS monitoring and shutdown logic              | `/etc/nut/upsmon.conf`| ✅        | ✅          |
| `nut.conf`        | Declares system role (`netclient`) | `/etc/nut/nut.conf`| ✅          | ✅          |
| `upssched.conf`   | Configures timed/custom actions during UPS events         | `/etc/nut/upssched.conf`| ✅      | ✅          |
| `upssched-cmd`    | User-defined script triggered by `upssched`               | `/usr/nut/upssched-cmd`) | ✅ | ✅ |


## <i class="fas fa-stethoscope"></i> Client Monitoring Configuration Files

### `upsmon.conf`
- **Purpose**: Monitors UPS status and triggers system shutdown.
- **Used by**: `upsmon`
- **Location**: `/etc/nut/upsmon.conf`
- **My file** (client):
  ```sh
  MONITOR ups@localhost 1 admin secret master
  ```

---

### `nut.conf`
- **Purpose**: Defines the system role: `standalone`, `netserver`, or `netclient`.
- **Used by**: All NUT services
- **Location**: `/etc/nut/nut.conf`
- **My file**:
  ```sh
  MODE=netserver   # on server
  ```

## <i class="far fa-clock"></i> Event Scheduling and Custom Actions

### `upssched.conf`
- **Purpose**: Provides hooks for custom actions or scripts during UPS events (like power failure or low battery).
- **Used by**: `upsmon` via `upssched`
- **Location**: `/etc/nut/upssched.conf`
- **Example**:
  ```bash
  CMDSCRIPT /usr/local/bin/upssched-cmd
  PIPEFN /var/run/nut/upssched.pipe
  LOCKFN /var/run/nut/upssched.lock

  AT ONBATT * START-TIMER onbatt 60
  AT TIMEOUT onbatt EXECUTE powerfail
  ```

---

### `upssched-cmd` (User Script)
- **Purpose**: A user-defined script that gets triggered by `upssched` to handle UPS events.
- **Used by**: `upssched`
- **Location**: User-defined (e.g., `/usr/local/bin/upssched-cmd`)
- **Example**:
  ```bash
  #!/bin/bash
  case $1 in
    powerfail)
      logger "UPS power failure: shutting down"
      /sbin/shutdown -h now
      ;;
    *)
      logger "Unhandled event: $1"
      ;;
  esac
  ```

---