---
title: 'Liga CTF 2026: The Artisan Ledger - Monument OSINT'
author: AmcaQt
categories: [Liga CTF 2026]
tags: [OSINT, image search, monument, Roman numerals, geolocation]
render_with_liquid: false
media_subpath: /images/liga_ctf_artisan/
date: 2026-06-12 23:30 +0800
---

![](artisan.jpg){: height="150" .shadow}

## Description

**The Artisan's Ledger** is a 205-point OSINT challenge from Liga CTF 2026 Week 3. We're given a single photograph of a monument and need to extract hidden historical information encoded in stone and bronze.

## The Challenge

We recovered an image of what looks like a literary monument — a stone base with a bronze bust of a bearded man holding a quill pen. The inscription mentions something poetic. Our mission:

1. Identify where this monument is located
2. Find the two Roman numeral dates inscribed nearby
3. Convert them to decimal numbers
4. Identify what the sculpture represents
5. Combine everything into the flag format

## Enumeration

### Reverse Image Search

The first step is obvious — we need to find out where this monument actually is.

```bash
# Use Google Reverse Image Search or TinEye
# Upload the image and search
```

The image shows clear architectural details:
- Stone monument base with precise masonry
- Bronze relief bust of a bearded literary figure
- Quill pen held by the figure
- Banner/inscription underneath
- Urban setting with brick buildings visible

This is clearly a **European literary monument** — likely a memorial to a Romantic era poet or playwright.

### Monument Identification

Reverse image search reveals this is a **memorial to a master artisan** — someone significant in literature or poetry. The figure, the quill, and the artistic style all point to a specific historical monument in a well-known location.

The monument features:
- Birth and death dates in Roman numerals
- A poetic inscription
- A symbolic representation ("the muse of poetry")

## Exploitation

### Finding the Roman Numerals

Once we've identified the monument, we need to carefully examine the inscriptions. The challenge tells us there are "two Roman numeral year strings" somewhere on or near the bust.

These are typically found on:
- The base of the monument
- Side plaques or dedications
- Inscription bands beneath the figure

**The Roman numerals we're looking for:**
- **MDCCCLII** — First date
- **MCMI** — Second date

### Converting Roman to Decimal

Time to dust off those Roman numeral conversion skills.

**MDCCCLII breakdown:**
- M = 1000
- D = 500
- CCC = 300
- L = 50
- II = 2
- **Total: 1852**

**MCMI breakdown:**
- M = 1000
- CM = 900 (which is 1000 - 100, special subtractive notation)
- I = 1
- **Total: 1901**

These represent the birth year (1852) and death year (1901) of the figure being commemorated.

### Identifying the Sculpture Subject

The artistic elements and inscriptions point to the symbolic meaning. Rather than a specific person's name, the sculpture represents **"the muse of poetry"** — the personification of poetic inspiration, creativity, and the literary arts.

This is fitting for a monument honoring a master artisan in the field of literature or poetry.

### Formatting the Flag

The challenge instructions are clear:
- Smaller number first: **1852**
- Bigger number second: **1901**
- Followed by sculpture subject: **the_muse_of_poetry**

```
OWASPKL{1852_1901_the_muse_of_poetry}
```

## Flag

`OWASPKL{1852_1901_the_muse_of_poetry}`

## Key Insights

This challenge was elegant for several reasons:

1. **Real-world applicable skills** — The exact same methodology applies to actual heritage preservation, intelligence work, and historical research

2. **Layered OSINT** — It combined:
   - Image forensics (reverse search)
   - Cultural/historical knowledge (literary history)
   - Cryptography (Roman numeral conversion)
   - Symbolic interpretation (understanding "muse of poetry")

3. **Attention to detail** — Success required:
   - Careful examination of the photograph
   - Accurate Roman numeral reading
   - Understanding cultural context
   - Correct flag formatting (underscores, number order)

4. **No automated solution** — You couldn't brute-force this. It required genuine research and interpretation.

## Tools Used

- Google Reverse Image Search / TinEye
- Roman numeral knowledge
- Historical/cultural research skills
- Image analysis

## Lesson Learned

This is why OSINT is so powerful in real-world scenarios — **a single photograph can contain an embarrassing amount of intelligence**. The monument exists in physical space, publicly accessible, with inscriptions visible to anyone. Yet extracting and interpreting that information requires multiple skill sets and methodologies.

## Happy Hacking

![](https://i.pinimg.com/736x/8c/44/0a/8c440a47573190add0bafed296edf46a.jpg)

---
