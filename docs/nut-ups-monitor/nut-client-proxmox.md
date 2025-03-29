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

These files are for my NUT clients that are running Proxmox PVE systems (pve10 and pve20). 

The Synology NAS client is set up with different code.

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
- **My file** (client - Proxmox):
    ```sh
    RUN_AS_USER root

    MONITOR ups@192.168.0.10 1 upsmon secret slave

    MINSUPPLIES 1
    SHUTDOWNCMD "/sbin/shutdown -h"
    NOTIFYCMD /usr/sbin/upssched
    POLLFREQ 2
    POLLFREQALERT 1
    HOSTSYNC 15
    DEADTIME 15
    POWERDOWNFLAG /etc/killpower

    NOTIFYMSG ONLINE    "UPS %s on line power"
    NOTIFYMSG ONBATT    "UPS %s on battery"
    NOTIFYMSG LOWBATT   "UPS %s battery is low"
    NOTIFYMSG FSD       "UPS %s: forced shutdown in progress"
    NOTIFYMSG COMMOK    "Communications with UPS %s established"
    NOTIFYMSG COMMBAD   "Communications with UPS %s lost"
    NOTIFYMSG SHUTDOWN  "Auto logout and shutdown proceeding"
    NOTIFYMSG REPLBATT  "UPS %s battery needs to be replaced"
    NOTIFYMSG NOCOMM    "UPS %s is unavailable"
    NOTIFYMSG NOPARENT  "upsmon parent process died - shutdown impossible"

    NOTIFYFLAG ONLINE   SYSLOG+WALL+EXEC
    NOTIFYFLAG ONBATT   SYSLOG+WALL+EXEC
    NOTIFYFLAG LOWBATT  SYSLOG+WALL+EXEC
    NOTIFYFLAG FSD      SYSLOG+WALL+EXEC
    NOTIFYFLAG COMMOK   SYSLOG+WALL+EXEC
    NOTIFYFLAG COMMBAD  SYSLOG+WALL+EXEC
    NOTIFYFLAG SHUTDOWN SYSLOG+WALL+EXEC
    NOTIFYFLAG REPLBATT SYSLOG+WALL+EXEC
    NOTIFYFLAG NOCOMM   SYSLOG+WALL+EXEC
    NOTIFYFLAG NOPARENT SYSLOG+WALL

    RBWARNTIME 43200

    NOCOMMWARNTIME 600

    FINALDELAY 5
    ```

---

### `nut.conf`
- **Purpose**: Defines the system role: `standalone`, `netserver`, or `netclient`.
- **Used by**: All NUT services
- **Location**: `/etc/nut/nut.conf`
- **My file**:
  ```sh
  MODE=netclient   # on client
  ```

## <i class="far fa-clock"></i> Event Scheduling and Custom Actions

### `upssched.conf`
- **Purpose**: Provides hooks for custom actions or scripts during UPS events (like power failure or low battery).
- **Used by**: `upsmon` via `upssched`
- **Location**: `/etc/nut/upssched.conf`
- **My file**:
    ```bash
  # This script gets called to invoke commands for timers that trigger.
    CMDSCRIPT /etc/nut/upssched-cmd

    # Command pipe and lock-file
    PIPEFN /etc/nut/upssched.pipe
    LOCKFN /etc/nut/upssched.lock

    # Send alerts immediately on change in line power
    AT ONBATT * EXECUTE onbatt
    AT ONLINE * EXECUTE onpower

    # (Optional) Silence the beeper after 2 minutes
    # AT ONBATT * START-TIMER mute_beeper 120
    # AT ONLINE * CANCEL-TIMER mute_beeper

    # Shutdown after 10 minutes on battery (10 * 60 = 1200)
    AT ONBATT * START-TIMER onbatt_shutdown 600

    # Cancel timer if power is restored
    AT ONLINE * CANCEL-TIMER onbatt_shutdown

    # Battery replacement indicated by cron'd quick test
    AT REPLBATT * EXECUTE replace_batt

    AT LOWBATT * EXECUTE onbatt
    AT COMMBAD * START-TIMER commbad 30
    AT COMMOK * CANCEL-TIMER commbad commok
    AT NOCOMM * EXECUTE commbad
    AT SHUTDOWN * EXECUTE powerdown
    ```

---

### `upssched-cmd` (User Script)
- **Purpose**: A user-defined script that gets triggered by `upssched` to handle UPS events.
- **Used by**: `upssched`
- **Location**: User-defined (e.g., `/usr/nut/upssched-cmd`)
- **My file**:
    ```bash
    #! /bin/sh
    #
    # This script should be called by upssched via the CMDSCRIPT directive
    # in the first line of /etc/nut/upssched.conf
    #

    # START: User-specific settings                            
    #
    UPS_USERNAME="upsmon"
    UPS_PASSWORD="secret"
    UPS_LINK="ups@192.168.0.10"
    # END  

    case $1 in
        onbatt)
            # make sure beeper is enabled
            upscmd -u ${UPS_USERNAME} -p ${UPS_PASSWORD} ${UPS_LINK} beeper.enable
            # alert
            message="Power outage, on battery"
            logger -t upssched-cmd "$message"
            ;;
        onpower)
            message="Power restored"
            logger -t upssched-cmd "$message"
            ;;
        mute_beeper)
             message="(2) minute limit exceeded, muting beeper"
             upscmd -u ${UPS_USERNAME} -p ${UPS_PASSWORD} ${UPS_LINK} beeper.mute
             ;;
        onbatt_shutdown)
            message="Triggering shutdown after (10) minutes on battery"
            logger -t upssched-cmd "$message"
            /sbin/upsmon -c fsd
            ;;
        replace_batt)
            message="Quick self-test indicates battery requires replacement"
            logger -t upssched-cmd "$message"
            ;;
        *)
            logger -t upssched-cmd "Unrecognized command: $1"
            ;;
    esac
    ```
---