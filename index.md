---
title: Home
layout: home
nav_order: 1
description: "Kurt's Homelab Setup Docs."
permalink: /
---

# Kurt's Homelab Setup Docs
{: .fs-9 }

I created this to document the steps I took to create my homelab. It might also be useful to others.
{: .fs-6 .fw-300 }

---

This site documents in detail how I set up my homelab. My first tranche of documentation concerns setting up my servers that host the VMs, and optimizing my experience with services like:

- **Hardware GPU transcode:** [<i class="fas fa-microchip" style="color: #2B66AF"></i> GPU hardware passthrough ]({% link docs/proxmox-gpu-passthrough/index.md %}) from my Proxmox VE host to individual VMs to accelerate my Docker-based Plex hardware transcode
- **Alerts** that tell me when there are issues with [<i class="fab fa-mixer" style="color: #D6762C"></i> Proxmox]({% link docs/proxmox-hardware-setup/proxmox-alerts-setup.md %}), my [power supply and <i class="fas fa-battery-three-quarters" style="color: gray"></i> UPS]({% link docs/nut-ups-monitor/index.md  %}), or my [<i class="fas fa-server fa-rotate-90" style="color: black"></i> Synology NAS shared folders]({% link docs/mount-synology-nfs-share/startup-scripts-to-check-mounts.md %}).
- **Backup:** [<i class="fab fa-mixer" style="color: gray"></i> Proxmox Backup Server (PBS)]({% link docs/proxmox-backup-server-on-synology-nas/index.md %}) running on my Synology NAS.

---

{: .note-title}
> PROPS to Anand @ SimpleHomeLab!
>
> I first got involve in homelabbing about a year ago when I stumbled upon Anand's [SimpleHomeLab website](https://www.simplehomelab.com/){:target="_blank"} and started tinkering with Docker. His site and videos are the most cogent explantion of all of this. And Anand is very active and helpful on the SimpleHomeLab Discord server, as are the great members there. Highly recommended. Thank you Anand and the SimpleHomeLab gang!
>
> - GitHub: [https://github.com/SimpleHomelab](https://github.com/SimpleHomelab){:target="_blank"} 
> - Website: [https://www.simplehomelab.com/](https://www.simplehomelab.com/){:target="_blank"} 
> - YouTube: [https://www.youtube.com/@Simple-Homelab](https://www.youtube.com/@Simple-Homelab){:target="_blank"} 

----

In the future I will add documentation for:

- ***Ultimate Docker Media Stack (UDMS)*** setup using [Anand's SimpleHomelab Deployarr app](https://www.simplehomelab.com/deployarr/){:target="_blank"}
    
    {: .important }
    >
    > Anand's paid script is not required, but it does save tons of time. All the code required to create the stack is available at [Anand's SimpleHomelab website](https://www.simplehomelab.com/deployarr/){:target="_blank"}. I recommend this site because it requires you to learn about the technologies as you use them.
- ***Unifi UDM Pro*** network setup
- ***Home Assistant OS (HAOS)*** setup within a Proxmox LXC
- ***Paperless-NGX*** setup