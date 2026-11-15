---
title: 'InIt CTF: The Fractured Key'
author: AmcaQt
categories: [InIt CTF]
tags: [Crypto, AES, PRNG]
render_with_liquid: false
media_subpath: /images/init-ctf-fractured-key/
image:
  path: challenge_card.png
---

## Description

We intercepted a secure communication between two operatives. We managed to get the cipher text and a strange script they were using to generate their keys. It looks like they tried to be clever by hiding the true flag among fakes. The true flag follows the standard `InIt{...}` format, but don't trust everything you decrypt.

500 points, category OffScript. Handout: `intercepted.zip`

![Challenge card](challenge_card.png)

## Question & Flag

### What's in the handout?

```
$ unzip intercepted.zip
Archive:  intercepted.zip
  inflating: intercepted.txt
  inflating: key_gen.py
```

![Unzipping the handout](unzip.png)

`intercepted.txt` is three hex-encoded AES ciphertexts, one per line. `key_gen.py` is the "strange script" they used to generate their keys:

```python
import os
import random
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad

# We need a 32-byte key for AES-256
def generate_master_key():
    # To make it extra secure, we base it on a random seed
    seed = random.randint(0, 255)
    random.seed(seed)
    
    key = bytearray()
    for _ in range(32):
        key.append(random.randint(0, 255))
    return bytes(key)

def encrypt(data: bytes) -> bytes:
    key = generate_master_key()
    iv = os.urandom(16)
    cipher = AES.new(key, AES.MODE_CBC, iv=iv)
    return iv + cipher.encrypt(pad(data, AES.block_size))
```

### Spotting the bug

It looks like a proper AES-256 key, 32 random bytes. But look at where the randomness actually comes from:

```python
seed = random.randint(0, 255)
random.seed(seed)
```

The "master key" is entirely decided by one byte, 0 to 255, fed into Python's `random.seed()`. Python's `random` module is a Mersenne Twister, not a cryptographically secure generator, so the same seed always produces the exact same 32 bytes afterward. That means the real key space isn't 2²⁵⁶, it's just **256**. Instead of brute-forcing AES-256, which is never happening, we only need to brute-force which one of 256 seeds was used.

The IV is also shipped in the clear, prepended to the ciphertext by `encrypt()`, so once we land on the right seed, decryption is just normal AES-CBC.

### Brute-forcing the seed

Took `key_gen.py`'s own function and made the seed an argument instead of random, then just looped all 256 of them. A wrong key fails PKCS7 unpadding almost every time, so "unpadding succeeded and the result is printable ASCII" is enough to tell us we hit the right seed.

```python
import random
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

def generate_master_key(seed):
    random.seed(seed)
    key = bytearray()
    for _ in range(32):
        key.append(random.randint(0, 255))
    return bytes(key)

def crack(line):
    data = bytes.fromhex(line.strip())
    iv, ct = data[:16], data[16:]
    for seed in range(256):
        key = generate_master_key(seed)
        cipher = AES.new(key, AES.MODE_CBC, iv=iv)
        try:
            pt = unpad(cipher.decrypt(ct), AES.block_size)
        except ValueError:
            continue
        if all(32 <= b < 127 for b in pt):
            print(f"seed {seed}: {pt.decode()}")

with open("intercepted.txt") as f:
    for line in f:
        if line.strip():
            crack(line)
```

```
$ python3 solve.py
seed 42: InIt{w0w_y0u_f0und_1t_but_1ts_f4k3}
seed 201: InIt{ps3ud0_r4nd0m_1s_n0t_r4nd0m_k3y_sp4c3_t00_sm4ll}
seed 137: InIt{h4h4_n1c3_try_k33p_l00k1ng}
```

![solve.py output, three decrypted lines](solve_output.png)

### Don't trust everything you decrypt

All three lines decrypt cleanly, which is exactly the warning in the description. Two of them are decoys there to bait a rushed submission:

- `InIt{w0w_y0u_f0und_1t_but_1ts_f4k3}` — taunts you for finding it
- `InIt{h4h4_n1c3_try_k33p_l00k1ng}` — taunts you for trying

The third is the only one that actually describes the bug we just exploited, predictable PRNG seed collapsing the key space, so that's the real flag.

```
InIt{ps3ud0_r4nd0m_1s_n0t_r4nd0m_k3y_sp4c3_t00_sm4ll}
```

## Conclusion

Textbook lesson in why `random.seed()` and the standard `random` module have no business anywhere near key generation, use `secrets` or `os.urandom` directly instead. Fun one too, since the decoys actually punish you for not reading your own exploit's output carefully.

# Happy Hacking
![Gif](https://i.pinimg.com/originals/65/39/5e/65395e7d4d8ea15941c538b54e293f60.gif)
