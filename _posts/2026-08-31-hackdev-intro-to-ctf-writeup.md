---
title: HackDev - Intro to CTF Writeup
auhor: AmcaQt
date: 2026-08-31
categories: [HackDev]
tags: [Intro to CTF]
media_subpath: /images/hackdev_intro_to_ctf
image:
    path: forensics_clue.jpg
---

## Intro

Joined HackDev: Intro to CTF and walked away as **runner up** . This one was a solid beginner-friendly mix category which is crypto, web exploitation, forensics, reverse engineering, OSINT, and a final challenge that chained everything together. Here's the full breakdown of how every category went down.

![Runner Up](2nd.png)

---

## Cryptography

### Crypto.txt

**Description**
First crypto challenge dropped a block of Base64-looking text. Classic warm-up to see if you know your encodings.

**Enumeration**
Looked at the string, recognized the padding (`=` at the end) and the character set straight up Base64.

**Exploitation**
Ran it through a decoder:

```bash
echo "WyBUUkFOU01JU1NJT04gREVDT0RFRCBdCk5pY2Ugd29yaywgYWdlbnQuIFlvdSByZWFkIGl0LgpZb3VyIGZsYWc6IEhhY2tEZXZ7YjRzMzY0XzFzX2p1c3RfM25jMGQxbmd9" | base64 -d
```

Output:
```
[ TRANSMISSION DECODED ]
Nice work, agent. You read it.
Your flag: HackDev{b4s364_1s_just_3nc0d1ng}
```

**Flag:** `HackDev{b4s364_1s_just_3nc0d1ng}`

### Shift.txt

**Description**
Second one gave a message where every letter looked shifted classic Caesar cipher setup, and the challenge straight up told us the shift was 13.

**Enumeration**
"Rotated by 13 places" is a dead giveaway for ROT13, so no need to bruteforce the shift.

**Exploitation**
Quickly go to ![Dcode ROT 13](https://www.dcode.fr/rot-13-cipher)

Output:
```
[ SHIFT CIPHER CRACKED ]
Every letter was rotated by 13 places.
Your flag: HackDev{r0t13_c4es4r_sh1ft}
```

**Flag:** `HackDev{r0t13_c4es4r_sh1ft}`

---

## Web Exploitation

### web.txt

**Description**
Given a terminal connection to `web1.ctf.hackdev.my` on port 443. Curling it gave raw data but the hint straight up told us: a terminal can't render HTML, but the full HTTP response is bigger than what a browser shows you.

**Enumeration**
The hint was basically the solve path — stop looking at rendered output, look at the raw source instead.

**Exploitation**
Opened the site in an actual browser and did view-source. The flag was sitting in the page source the whole time, just not visible in the rendered page.

![](web1.png)

**Flag:** `HackDev{s0urc3_c0d3_h1d3s_s3cr3ts}`

### web2.txt (IDOR)

**URL:** `https://web2.ctf.hackdev.my/profile?id=30`

**Description**
FOr this challenge agak straight forward, Some resource was being fetched by ID with no access control check.

**Enumeration**
Try change the value of id , from 1 to 10, increment 1 by 1, found various profile

**Exploitation**
Wrote a script sebab macam makan masa nak sampai 30, turns out the flag dekat ID 22

```bash
import requests
import time

BASE_URL = "https://web2.ctf.hackdev.my/profile"
START_ID = 1
END_ID = 30          
FLAG_PATTERN = "HackDev{"  
DELAY = 0.2 #letak delay, kesian skit dekat server

def enumerate_ids():
    session = requests.Session() 

    for uid in range(START_ID, END_ID + 1):
        url = f"{BASE_URL}?id={uid}"
        try:
            resp = session.get(url, timeout=10)
        except requests.RequestException as e:
            print(f"[!] id={uid} -> request failed: {e}")
            continue

        # log every request status so you can spot weird ones (403, 500, etc)
        print(f"[id={uid}] status={resp.status_code} len={len(resp.text)}")

        if FLAG_PATTERN.lower() in resp.text.lower():
            print(f"    >>> FLAG FOUND at id={uid}!")
            print(f"    URL: {url}")
            
            with open("hits.txt", "a") as f:
                f.write(f"id={uid} | {url}\n")

        time.sleep(DELAY)

if __name__ == "__main__":
    enumerate_ids()

```

**Flag:** `HackDev{1d0r_3num3r4t3_th3_d1r3ct0ry}`

---

## Forensics

**Description**
Given an image file to dig through for hidden data.

![](forensics_clue.jpg)

**Enumeration**
Ran `exiftool` on the image to check the metadata for anything out of place.

**Exploitation**
Metadata contained a Base64-encoded string. Decoded it to get the flag.

```bash
exiftool image.jpg
# found a base64 string in metadata
echo "SGFja0RldnttM3Q0ZDR0NF9uM3Yzcl9sMTNzfQ==" | base64 -d
```

**Flag:** `HackDev{m3t4d4t4_n3v3r_l13s}`

---

## Reverse Engineering

**Description**
Challenge shipped a binary with a key-submission.

**Enumeration**
Ran `strings` on the binary to scan for anything human-readable hiding inside.

**Exploitation**
Found the key sitting in plaintext in the strings output. Submit on the url given and get the flag.

```bash
strings gate.exe | grep -i key
```

![](key-clue.png)

**Flag:** `HackDev{cr4ck3d_7h3_k3y_f0r_fr33_5ub5cr1p710n}`

---

## OSINT

**Description**
Challenge pointed us toward the HackDev website itself, with a clue mentioning "robots."

**Enumeration**
`robots.txt` was the obvious lead — checked it and found a disallowed path.

**Exploitation**
Followed the trail starting from `/vault`, then kept chaining through each path it led to until the last directory, found a base64 that turns out it was the flag.

```
https://hackdev.my/robots.txt

1st. /vault
2nd. /dead-drop
3rd. /operative
4th. /ghost-protocol
5th. /final-transmission

echo "SGFja0RldnswczFudF90cjQxbF9jMG1wbDN0M30=" | base64 -d

```

**Flag:** `HackDev{0s1nt_tr41l_c0mpl3t3}`

---

## Final Challenge

**Description**
The last challenge, hint said the order was reverse → forensic → crypto → web, so it chained multiple categories into one long solve path.

**Enumeration**
Started the same way as the RE challenge: ran `strings` on the given file and found a clue referencing "burned_evidence."

```RECOVERY NOTE: the real evidence is not here. download it at https://web2.ctf.hackdev.my/img/burned_evidence.jpg then read the hidden metadata (EXIF) of that image. the data about the picture, not the picture itself. what you find there is encoded.
```

**Exploitation**
That clue pointed to an image file. Downloaded it, ran `exiftool` again, and found a Base64-encoded string in the metadata. Decoded it to get the final flag.

```bash
strings final_file
exiftool burned_evidence.jpg
echo "aHR0cHM6Ly93ZWIyLmN0Zi5oYWNrZGV2Lm15L2pDWXpRVlFwZjNlY21OYzU=" | base64 -d
```

**Flag:** `HackDev{tr1pl3_thr34t_d0n3}`

---

## Lesson Learned

Love this webinar and mini CTF Challenge, learned a lot from the webinar, just realized that there are proper way to solve ctf chall than just "langgar" xD. Much appreciated to HackDev for hosting this Mini CTF and Webinar, realy good for beginner.

---

![Happy Hacking](https://i.pinimg.com/originals/ba/86/fc/ba86fcef7b6d8cb8d817430d3b1b1710.gif)

## Happy Hacking!
