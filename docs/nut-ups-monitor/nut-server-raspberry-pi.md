---
title: NUT Server - Raspberry Pi
parent: NUT UPS Monitor
nav_order: 2
---

# <i class="fas fa-power-off"></i> NUT Server - Raspberry Pi
{: .no_toc }

<i class="fas fa-power-off"></i> NUT UPS setup
{: .label .label-rasp }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Sources

There are many online walkthroughs on how to install NUT Server on a Raspberry Pi. Here is the best:

1. I used the [NUTandRpi script](https://github.com/dzomaya/NUTandRpi){:target="_blank"} to install NUT Server to my Raspberry Pi. 
   1. You can also install NUT server manually for [any OS](https://networkupstools.org/docs/user-manual.chunked/_installation_instructions.html#Installing_packages){:target="_blank"}. 

2. I used Techno Tim's [NUTandRpi script instructions](https://technotim.live/posts/nut-server-script/){:target="_blank"} along with the instructions on the script GitHub page.
  
    Techno Tim's walkthrough video:

    <iframe width="560" height="315" src="https://www.youtube.com/embed/HgKeD4320c0?si=R14OKtKQVtaj1woG" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## <i class="fas fa-cogs"></i> Configuration files summary table

After installing and experimenting, I became confused on what files are where and what they do. I have documented my setup below to help.

| File              | Purpose                                                  | File Directory      | On Server? | On Client? |
|-------------------|-----------------------------------------------------------|---------------------|------------|------------|
| `ups.conf`        | Defines UPS hardware and driver                           | `/etc/nut/ups.conf` | ✅          | ❌          |
| `upsd.conf`       | Configures `upsd` network listener                         | `/etc/nut/upsd.conf`| ✅          | ❌          |
| `upsd.users`      | Defines user accounts and permissions for `upsd`          | `/etc/nut/upsd.users`| ✅         | ❌          |
| `upsmon.conf`     | Configures UPS monitoring and shutdown logic              | `/etc/nut/upsmon.conf`| ✅        | ✅          |
| `nut.conf`        | Declares system role (`standalone`, `netserver`, `netclient`) | `/etc/nut/nut.conf`| ✅          | ✅          |
| `hosts.conf`      | Lists UPS devices for web UI tools (`upsstats.cgi`, etc.) | `/etc/nut/hosts.conf`| ✅         | ❌ (unless running web UI) |
| `upsset.conf`     | Defines host access for the `upsset.cgi` config tool      | `/etc/nut/upsset.conf`| ✅        | ❌          |
| `upssched.conf`   | Configures timed/custom actions during UPS events         | `/etc/nut/upssched.conf`| ✅      | ✅          |
| `upssched-cmd`    | User-defined script triggered by `upssched`               | (user-defined, e.g. `/usr/local/bin/upssched-cmd`) | ✅ | ✅ |


---

## <i class="fas fa-plug"></i> Raspberry Pi Server Configuration Files

My NUT server configuration is simple because I want my Raspberry Pi to run until it dies if it loses power. My `upsmon.conf` only monitors and doesn't require any triggers to execute for shutdown of the NUT server. Therefore, I don't need `upssched.conf` to define triggers and actions or the `upssched-cmd` script to implement them.

### `ups.conf`
- **Purpose**: Defines one or more UPS devices and their drivers.
- **Used by**: `upsdrvctl`
- **Location**: `/etc/nut/ups.conf`
- **My file**:
```config
[ups]
        driver = "usbhid-ups"
        port = "auto"
        vendorid = "0463"
        productid = "FFFF"
        product = "Ellipse ECO"
        serial = "000000000"
        vendor = "EATON"
        bus = "001"
```

---

### `upsd.conf`
- **Purpose**: Configures how the NUT daemon (`upsd`) listens for connections.
- **Used by**: `upsd`
- **Location**: `/etc/nut/upsd.conf`
- **My file**:
```config
LISTEN 0.0.0.0 3493
```

---

### `upsd.users`
- **Purpose**: Defines users and their permissions (e.g., for monitoring or shutdown).
- **Used by**: `upsd`
- **Location**: `/etc/nut/upsd.users`
- **My file**:

```config
# This host's own upsmon, running as the primary.
# Must match the username in this host's upsmon.conf MONITOR line.
[admin]
  password = secret
  upsmon master

# The Synology NAS client. DSM connects as a secondary and
# authenticates as monuser. That name is hardcoded in DSM.
[monuser]
  password = secret
  upsmon slave

# The client user on the other systems (Proxmox machines)
[upsmon]
  password = secret
  upsmon slave
```

{: .warning }
> **Corrected 2026-08-25.** Earlier versions of this page showed `[monuser]` containing a line reading `admin master`. That is not valid `upsd.users` syntax. The only directives this file accepts are `password`, `actions`, `instcmds`, and `upsmon primary|secondary` (the legacy spellings `master|slave` still work in NUT 2.8).
>
> The consequence of the bad line was that `[monuser]` had **no** upsmon privilege and `[admin]` did not exist at all, so every LOGIN was refused. Anonymous reads kept working the whole time, which is why nothing looked wrong: the `upsstats.cgi` page, Home Assistant, and the Synology status display all showed correct live data. What was silently broken was the primary/secondary registration that coordinated shutdown depends on. This went unnoticed from March 2025 until August 2026.

{: .note }
> **Verifying this file is correct.** A misconfiguration here has no visible symptom outside the logs, so check it explicitly.
>
> On the client side, a bad username or privilege shows up in `journalctl -u nut-monitor` as:
>
> ```
> Login on UPS [ups@localhost] failed - got [ERR ACCESS-DENIED]
> ```
>
> On the server side, `upsd` logs successful logins but **not** failed ones, so the absence of a success line is the only signal. Confirm every client with:
>
> ```sh
> sudo journalctl -u nut-server | grep "logged into"
> ```
>
> You should get one line per client, for example:
>
> ```
> User admin@127.0.0.1      logged into UPS [ups]
> User upsmon@192.168.0.110 logged into UPS [ups]
> User upsmon@192.168.0.120 logged into UPS [ups]
> User monuser@192.168.0.5  logged into UPS [ups]
> ```

---

## <i class="fas fa-stethoscope"></i> Monitoring Configuration Files

### `upsmon.conf`
- **Purpose**: Monitors UPS status and triggers system shutdown.
- **Used by**: `upsmon`
- **Location**: `/etc/nut/upsmon.conf`
- **My file** (this host's own monitor):
  ```sh
  MONITOR ups@localhost 1 admin secret master
  ```

{: .note }
> The username on this line (`admin`) must exist in `upsd.users` with `upsmon master`. If the two files disagree, `upsmon` will still poll UPS status successfully and report `Communications with UPS established`, while its LOGIN is refused. Do not read that message as proof the configuration is good.

{: .note }
> There is deliberately no `SHUTDOWNCMD` in this file. This Pi is the monitor for everything else, so I want it to stay up as long as the battery lasts rather than shut itself down. `upsmon` will log `Warning: no shutdown command defined!` on every start. That warning is expected here.
>
> The trade-off is that the Pi loses power uncleanly when the battery finally runs out, which is a known way to kill an SD card. Add a `SHUTDOWNCMD` if you would rather protect the card than maximise monitoring time.

---

### `nut.conf`
- **Purpose**: Defines the system role: `standalone`, `netserver`, or `netclient`.
- **Used by**: All NUT services
- **Location**: `/etc/nut/nut.conf`
- **My file**:
  ```sh
  MODE=netserver   # on server
  ```

---

## <i class="fas fa-globe"></i> Web Interface Configuration Files

### `hosts.conf`
- **Purpose**: Lists UPS devices for use in the web interface (e.g., `upsstats.cgi`, `upsset.cgi`).
- **Used by**: Web CGI tools
- **Location**: `/etc/nut/hosts.conf`
- **Example**:
  ```config
  MONITOR ups@localhost "Sotelo Computer Cabinet UPS"
  ```

{: .note }
> The CGI tools reach `upsd` over loopback, as this `MONITOR` line shows. The web interface does **not** need to be on a different IP address or network interface from the NUT server, and both can serve from the same address. Verified 2026-08-25 serving `https://192.168.0.10/cgi-bin/nut/upsstats.cgi` with `upsd` on the same host.

---

### `upsset.conf`
- **Purpose**: Defines access permissions for the `upsset.cgi` tool.
- **Used by**: `upsset.cgi`
- **Location**: `/etc/nut/upsset.conf`
- **Example**:
  ```sh
  I_HAVE_SECURED_MY_CGI_DIRECTORY
  ```
