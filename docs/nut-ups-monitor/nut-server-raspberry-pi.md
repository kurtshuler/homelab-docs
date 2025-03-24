---
title: NUT Server - Raspberry Pi
parent: NUT UPS Monitor
nav_order: 2
---

# NUT Server - Raspberry Pi
{: .no_toc }

NUT UPS setup
{: .label .label-purple }

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

## 📑 Configuration files summary table

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

## 🖥️ Raspberry Pi Server Configuration Files

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
    # This is the NUT SERVER user on the Blackberry Pi host and 
    # the NUT CLIENT user on the Synology NAS
    [monuser]
      password = secret
      admin master

    # This is the CLIENT user on the other systems (Proxmox machines)
    [upsmon]
      password = secret
      upsmon slave
```
---

## 🖥️ Monitoring Configuration Files

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

---

## 🌐 Web Interface Configuration Files

### `hosts.conf`
- **Purpose**: Lists UPS devices for use in the web interface (e.g., `upsstats.cgi`, `upsset.cgi`).
- **Used by**: Web CGI tools
- **Location**: `/etc/nut/hosts.conf`
- **Example**:
  ```config
  MONITOR ups@localhost "Sotelo Computer Cabinet UPS"
  ```

---

### `upsset.conf`
- **Purpose**: Defines access permissions for the `upsset.cgi` tool.
- **Used by**: `upsset.cgi`
- **Location**: `/etc/nut/upsset.conf`
- **Example**:
  ```sh
  I_HAVE_SECURED_MY_CGI_DIRECTORY
  ```