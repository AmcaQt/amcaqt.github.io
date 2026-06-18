---
title: 'Liga CTF 2026: The Ghost in the Logs'
author: AmcaQt
categories: [Liga CTF 2026]
tags: [forensics, windows, evtx, powershell, base64, log analysis]
render_with_liquid: false
media_subpath: /images/liga_ctf_ghost_logs/
date: 2026-06-07 00:00 +0000
image: 
    path: room_card.png
date: 2026-06-07 12:26 +0800
---

## Description

The **Ghost in the Logs** is a Forensics challenge from Liga CTF 2026 Week 3. We were given a Windows Event Log file and tasked to find a rogue account created by an attacker who tried to hide their tracks using PowerShell obfuscation.

## Parsing the Log File

We got a `.evtx` file which is a Windows Event Log format. To read it on Linux, we used Chainsaw.

```bash
./chainsaw dump ghost_in_log.evtx > ghost_logs.txt
```

## Finding the Suspicious Commands

Since the tip mentioned Base64, we grep for `-EncodedCommand` which is how PowerShell hides encoded scripts.

```bash
grep -A2 "EncodedCommand" ghost_logs.txt
```

![Two EncodedCommand entries found in the logs](osint-1.png)
_Two suspicious PowerShell EncodedCommand entries buried in the noise_

Found two encoded commands. The attacker created a ton of fake `LocalAdmin_1` through `LocalAdmin_10` accounts just to bury this in the logs.

## Decoding the Commands

PowerShell EncodedCommand uses UTF-16LE so we need iconv to decode it properly.

```bash
grep "EncodedCommand" ghost_logs.txt | grep -o 'EncodedCommand [A-Za-z0-9+/=]*' | awk '{print $2}' | while read b64; do
    echo "$b64" | base64 -d | iconv -f UTF-16LE -t UTF-8
done
```

![Decoded commands revealing the flag parts](osint-2.png)
_Both commands decoded, flag split across password and description fields_

The two decoded commands:

```powershell
New-LocalUser -Name "SVC_BackupAdmin" -Password (ConvertTo-SecureString "l0g_us3r_cr34t3d}" -AsPlainText -Force)

Set-LocalUser -Name "SVC_BackupAdmin" -Description "OWASPKL{w1nd0ws_ev3nt"
```

The rogue account is `SVC_BackupAdmin` and the flag was split across two fields, the description and the password.

Combine both parts and we get the full flag.

## Flag

`OWASPKL{w1nd0ws_ev3ntl0g_us3r_cr34t3d}`

## Conclusion

Fun forensics challenge. The noise from all the fake LocalAdmin accounts was meant to distract but grepping for EncodedCommand cuts straight through it. Always check process creation events (Event ID 4688) when hunting for PowerShell abuse.

## Happy Hacking

![](https://i.pinimg.com/originals/17/3f/6e/173f6e2c27e4821abdf5d81054d91572.gif)