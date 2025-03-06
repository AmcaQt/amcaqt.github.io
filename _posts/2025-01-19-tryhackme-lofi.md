---
title: 'TryHackMe: Lofi'
author: AmcaQt
categories: [TryHackMe]
tags: [web, Enumeration, LFI]
render_with_liquid: false
media_subpath: /images/thm_lofi/
image:
  path: room_card.png
date: 2024-09-16 00:00 +0000
---

![Tryhackme Room Link](room_image.PNG){: width="600" height="150" .shadow }
_<https://tryhackme.com/r/room/lofi>_

## Description

Lo-Fi was a very simple room where we exploited a `Local File Inclusion (LFI)` vulnerability to read the flag.

## Enumaration

In this challenge, we skip the Nmap scan. Since the room description already asks us to visit a web page, and we should test for local file inclusion here.

First we open the website direct by the IP Address given ( Machine IP ), looks like a normal website to listen a Lofi Music

![WebPage](1.PNG)

## Discovering LFI

By clicking the function on bottom right on `Discography` box, noticed that it direct to url `http://MACHINE_IP/?page=relax.php`, that means we can abuse the URL by manipulating the GET request using the following payload

![Local File Inclusion](clue-1.PNG)

```
http://MACHINE_IP/?page=relax.php ~> Normal link to show content on filename relax.php
http://MACHINE_IP/?page=../../../etc/passwd ~> Shown it was not found
http://MACHINE_IP/?page=../../../../../../etc/passwd ~> Shown /etc/passwd content
```

Now we found Vulnerability of [LFI](https://www.invicti.com/learn/local-file-inclusion-lfi/)

## Gaining Flag

So that we found Entry Point of LFI, maybe we can try view the file direct from the payload ? 

Change the payload from `../../../../../../etc/passwd` into `../../../../../../flag.txt`, then you'll get the flag.

![Flag](FLAG.jpg)

## Conclusion

Yet another fun room with my favourite Vulnerability, Love it and hope to play more room with Local File Inclusion Vulnerability

## Happy Hacking

![](https://i.pinimg.com/originals/7b/41/c1/7b41c1965ed2d95cb4c8894efe299af1.gif)