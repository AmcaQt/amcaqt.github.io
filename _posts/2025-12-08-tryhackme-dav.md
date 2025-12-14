---
layout: post
title: 'TryHackMe: Dav'
author: AmcaQt
categories: [TryHackMe]
tags: [Web, Enumeration, Boot2Root]
render_with_liquid: false
media_subpath: /images/thm_dav
image:
    path: room_card.jpeg
date: 2025-12-05 10:39 +0800
---

![TryHackMe Room Link](room_image.PNG){: width="600" height="150" .shadow}
_<https://tryhackme.com/room/bsidesgtdav>_

## Description

The **Dav** is a boot2root machine for FIT and bsides guatemala CTF

## Gain a Flag

### Enumeration

#### Nmap Scan

![Nmap Scan](nmap-Scan.PNG)

From the scan we see that only one open ports:

- 80/tcp http

with an info of apache version 2.4.18

![Main Page](main-page.PNG)

from the index page, we see default Apache Page with no further info, then let's do further enumeraion with [Gobuster](https://www.kali.org/tools/gobuster/), using this command 

```
gobuster dir -u MACHINE_IP -w /usr/share/dirb/wordlists/common.txt
```

from this scanning we found 6 directory, one of it is **/webdav**, if we go to the directory it will ask for credentials. Luckily i do some researching on Webdav vulnerability [Webdav Vuln](https://xforeveryman.blogspot.com/2012/01/helper-webdav-xampp-173-default.html).

Default cred for webdav referring on the documentation is :-

User : wampp
Pass : xampp

![clue_1](clue_1.PNG)

when we login we found and open directory that only have 1 file named **password.dav**, that mean we on the right track. 

#### What the User Flag ? 

What we got so far :-

```
Username = wampp
Password = xampp

http server: Apache 2.4.18
```

from the info that we have, we can login to site using [Cadaver](https://www.kali.org/tools/cadaver/).

P/S : Cadaver is CLI WebDav client for Unix

![access_1](access_1.PNG)

Cadaver is actually look-a-like SSH, not the same, but look-a-like, means we can upload, download, move and copy. By using `put` we can upload our reverse shell file

```
put reverse_shell.php
```

![access_2](access_2.PNG)wdaw

remember that `/webdav` directory ? the reverse_shell file is there, so now open a listener on Kali `nc -lvnp 1234` then open the file on webdav directory, you'll get your shell.

![access_3](access_3.PNG)

The flag is on `/home/merlin`

![flag](user.jpg)

#### What the root Flag ?
awda
for the root flag, it is simple, first check the permission using `sudo -l`, noticed that no password on `/bin/cat`. So simple use this command 

```
sudo /bin/cat /root/root.txt
```

then you'll get the flag

![root](root.jpg)

## Conclusion

This lab is simple Boot2Root, good for beginner to learn how to exploit from apache server from being normal user into root

## Happy Hacking

![](https://i.pinimg.com/originals/e5/0a/79/e50a79b81d32b7c87383317c9ee62c22.gif)