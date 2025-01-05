---
title: 'TryHackMe: Cheese CTF'
author: AmcaQt
categories: [TryHackMe]
tags: [web, enumeration, RCE, LFI, Privilege Escalation]
render_with_liquid: false
media_subpath: /images/thm_cheese_ctf/
image:
  path: room_card.png
date: 2024-09-16 00:00 +0000
---

![Tryhackme Room Link](room_image.png){: width="600" height="150" .shadow }
_<https://tryhackme.com/r/room/cheesectfv10>_

## Description

Cheese CTF was a straightforward room where we used SQL injection to bypass a login page and discovered an endpoint vulnerable to `LFI`. By utilizing PHP filters chain to turn the `LFI` into `RCE`, we gained a foothold on the machine. After that, we exploited a writable `authorized_keys` file to pivot to another user. As this user, we fixed a syntax error in a timer and used `sudo` privileges to start it, which allowed us to run a service that created a SUID binary. By exploiting this SUID binary, we were able to escalate our privileges to the `root` user.

## Enumaration

By checking the most common ports, we can see that a custom web application is running on port `80`.

![WebPage](page.PNG)

## Getting Shell

### Discovering LFI

Clicking the login button on the index page redirects us to `/login.php`, where we encounter a login form.

After trying a couple of simple SQL injection payloads, we are able to bypass the login using the payload `' or ''='` as the username. and `'` as password

After bypassing the login, we are redirected to `http://MACHINE_IP/secret-script.php?file=supersecretadminpanel.html`, where the `file` parameter is particularly interesting. Additionally, by checking the other links on the page, we notice the application also uses the PHP filters in the `file` parameter.

![Clue 1](clue-1.PNG)

Since the application seems to accept PHP filters, we can try using the `convert.base64-encode` filter to read the source code of the PHP pages.

By examining the source code of `secret-script.php`, we see that it simply takes the `file` parameter and calls `include` with it.

![Clue 2](clue-2.PNG)

### RCE With PHP Filters Chain

We can achieve RCE using PHP filters chain
> Refferer here : https://www.synacktiv.com/en/publications/php-filters-chain-what-is-it-and-how-to-use-it

First, we need to generate a payload using the [`php_filter_chain_generator`](https://github.com/synacktiv/php_filter_chain_generator)

![Clue 3](clue-3.PNG)

by this we can generate new payload, to gain reverse shell

![Reverse Shell](revshell.PNG)

## Shell as comte

While checking for files writable by the `www-data` user, we discover `/home/comte/.ssh/authorized_keys`

To get a shell as the `comte` user, we can simply add an SSH key to the `authorized_keys` file. 

Since the authorized_keys on the machine is empty we have to generate the SSH Key

Then, write the public key to the `/home/comte/.ssh/authorized_keys` file.

Now, using the private key, we can SSH into the system and obtain a shell as the `comte` user, allowing us to read the user flag.

![User Flag](user.jpg)

## Shell As root

As usual, Checking the sudo privileges for the `comte` user, we see that we can reload the configuration files for `systemd` and manually start the `exploit.timer`, which will in turn trigger the execution of `exploit.service`.

![LPE](lpe.PNG)

Let's take a look at what is behind the service `/etc/systemd/system/exploit.service`. It copies `xxd` to `/opt` and turns it into a `SUID` binary. Since it is then still in the possession of the root user, we can use it to read any files.

![LPE](lpe-2.PNG)

However, we receive an error message when we start the service. The set timer in exploit.timer is malformed. We remember that we have write access to this. Let's take a look at it.

![LPE](lpe-1.PNG)

The value for OnBootSec is not set, which triggers the error.

We correct this by setting `OnUnitActiveSec=0` and `OnBootSec=0`. This triggers the service immediately after execution.

```
[Unit]
Description=Exploit Timer

[Timer]
OnUnitActiveSec=0
OnBootSec=0

[Install]
WantedBy=timers.target
```

We run the service as follows, which copies the `xxd` to `/opt` and sets the SUID bit, recognisable by the red background.

![LPE](lpe-3.PNG)

Next, we specify which file we want to read. The first thing we are interested in is the root flag. As mentioned, we can find out how to do this in GTFOBins. And we are able to read the root flag.

```
LFILE=/root/root.txt
./xxd "$LFILE" | xxd -r
```

![Root Flag](root.jpg)

## Conclusion

for the concluion, It was fun room plus I love the concept of abusing vulnerability of `LFi` that lead to `RCE` and I learn a new thing about PHP FIlter CHain.

## Happy Hacking

![](https://i.pinimg.com/originals/24/5d/82/245d82247faed3e636991bb8140d1981.gif)
