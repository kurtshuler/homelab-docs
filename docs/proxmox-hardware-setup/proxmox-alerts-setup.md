---
title: Proxmox Alerts Setup
parent: Proxmox Hardware Setup
nav_order: 3
---

# Proxmox Alerts Setup
{: .no_toc }

Proxmox host setup
{: .label .label-yellow }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Introduction

This guide is adapted from Techno Tim's [Set up alerts in Proxmox before it's too late!](https://technotim.live/posts/proxmox-alerts/){:target="_blank"} article and video.

<iframe width="560" height="315" src="https://www.youtube.com/embed/85ME8i4Ry6A?si=ao8hFKoFk9i-JuSX" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Install dependencies

Do
{: .label .label-green}


### Mailutils and SASL authentication libraries

```shell
apt update
apt install -y libsasl2-modules mailutils
```

### Configure app passwords on your Google account

[Configure app passwords on your Google account](https://myaccount.google.com/apppasswords){:target="_blank"}

## Configure postfix with new gmail and password

Add your gmail and app password to postfix:

```shell
echo "smtp.gmail.com your-email@gmail.com:YourAppPassword" > /etc/postfix/sasl_passwd
```

Update permissions:

```shell
chmod 600 /etc/postfix/sasl_passwd
```

Hash the file:

```shell
postmap hash:/etc/postfix/sasl_passwd
```

Check to to be sure the db file was created:

```shell
cat /etc/postfix/sasl_passwd.db
```

Edit postfix config:

```shell
nano /etc/postfix/main.cf
```

Comment out line 26:

```shell
### relayhost =
```

Add this text at end of file:


```sh
# google mail configuration

relayhost = smtp.gmail.com:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options =
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/Entrust_Root_Certification_Authority.pem
smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
smtp_tls_session_cache_timeout = 3600s
```

Save the file. 

### Check the gmail email connection

Reload postfix

```shell
postfix reload
```

Send a test email

```shell
echo "This is a test message sent from postfix on my Proxmox Server" | mail -s "Test Email from Proxmox" shulerpve1@gmail.com
```

## Fix the "from" name in email

Install dependency:

```shell
apt update
apt install postfix-pcre
```

Edit config:

```shell
nano /etc/postfix/smtp_header_checks
```

Add the following text:

```shell
/^From:.*/ REPLACE From: pve1-alert <pve1-alert@gmail.com>
```

Hash the file:

```shell
postmap hash:/etc/postfix/smtp_header_checks
```

Check the contents of the file:

```shell
cat /etc/postfix/smtp_header_checks.db
```

### Add module to postfix and test
Add the module to our postfix config:

```shell
nano /etc/postfix/main.cf
```

Add to the end of the file:

```shell
smtp_header_checks = pcre:/etc/postfix/smtp_header_checks
```

Reload postfix service:

```shell
postfix reload
```

Send another test email:

```shell
echo "This is a second test message sent from postfix on my Proxmox Server" | mail -s "Second Test Email from Proxmox" shulerpve1@gmail.com
```