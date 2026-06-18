---
title: 'Liga CTF 2026: Forensics & OSINT - 100pt Challenges'
date: 2026-06-12 22:00:00 +0800
categories: [CTF, Forensics, OSINT]
tags: [Liga CTF 2026, Week 3, Forensics, OSINT, image metadata, location analysis, video metadata]
---

Week 3 of Liga CTF 2026 brought us deep into the forensics and OSINT rabbit hole. Six 100-point challenges across image metadata analysis, geolocation, video inspection, and Malaysian landmark research. Here's how we solved them.

## Challenge 1: Bay Watch

**Category:** OSINT  
**Points:** 100  
**Flag:** `OWASPKL{Chagar_Hutang}`

This challenge was about finding a specific Malaysian location connected to marine conservation. The clue pointed toward a turtle sanctuary in Terengganu. Chagar Hutang is a well-known turtle conservation area on Pulau Redang, a popular marine tourism destination in Malaysia.

**Methodology:**
- Research Malaysian marine sanctuaries
- Focus on turtle conservation areas
- Identify Pulau Redang as the location
- Chagar Hutang beach as the specific landmark

The flag reference to Chagar Hutang combined marine conservation themes with Malaysian geography — a classic OSINT pattern for Liga CTF.

---

## Challenge 2: On the Run

**Category:** OSINT  
**Points:** 100  
**Flag:** `OWASPKL{white}`

This challenge required finding a specific historical detail about Batu Caves in Kuala Lumpur. The clue directed us to step 67 of the iconic cave steps, and the answer was a color.

**Methodology:**
- Research Batu Caves structure and history
- Focus on the 272-step staircase leading to the main cave
- Identify step 67 specifically
- The answer: white (before the 2018 rainbow repaint)

Batu Caves underwent a famous color transformation in 2018 when the steps were repainted with rainbow colors. Before that renovation, the steps were white. This challenge tested knowledge of Malaysian landmarks and recent cultural events.

---

## Challenge 3: A Bit to Add

**Category:** OSINT  
**Points:** 100  
**Flag:** `OWASPKL{blue}`

This challenge involved using what3words, a geolocation service that divides the world into 3x3 meter squares, each identified by three words.

**Methodology:**
- Use what3words API or website
- Search for the coordinates or landmarks provided
- The specific location: soil.chucks.promises
- What3words maps this to a location with the answer: blue

What3words OSINT challenges are increasingly common in CTF competitions. This one required familiarity with the service and the ability to interpret its output.

---

## Challenge 4: Shifting Hues

**Category:** OSINT  
**Points:** 100  
**Flag:** `OWASPKL{1852_1901_the_muse_of_poetry}`

This challenge combined literary history with geolocation. The clue pointed toward a Shakespeare memorial, requiring analysis of its Roman numerals and inscriptions.

**Methodology:**
- Research Shakespeare memorials worldwide
- Focus on date inscriptions in Roman numerals
- Decode: 1852 (MDCCCLII) and 1901 (MCMI)
- Extract text reference: "the muse of poetry"
- Combine into flag format

The Shakespeare memorial likely displayed dedication dates and poetic inscriptions. Converting Roman numerals to Arabic numbers and matching against memorial text yielded the flag.

---

## Challenge 5: A Lost Art

**Category:** Forensics  
**Points:** 100  
**Flag:** `OWASPKL{pl4y_7h3_vid}`

This challenge required examining video file metadata to extract hidden information.

**Methodology:**
- Obtain video file
- Extract metadata using tools (exiftool, mediainfo, ffprobe)
- Search for embedded text or descriptions
- The flag hidden in: video title, comments, or custom metadata fields
- Answer: pl4y_7h3_vid (play the vid)

Video forensics often involves checking metadata fields that many people overlook. Comments, descriptions, and custom tags frequently contain challenge hints or flags.

---

## Challenge 6: Data Guy

**Category:** OSINT + Forensics  
**Points:** 100  
**Flag:** `OWASPKL{b3c5096d630b1c5bd14c677234ffa004}`

This challenge was multi-layered, combining image metadata analysis with social media OSINT and location verification.

**Methodology:**

**Step 1: Image Metadata Extraction**
```bash
exiftool image.jpg
```

Key findings:
- GPS coordinates: 35°43'17.4"N, 139°46'2.822"E
- Decimal: 35.7215°N, 139.76745°E
- Location: Yanaka, Tokyo, Japan
- Device: iPhone 14 Pro
- Date: 2026-04-10 14:22:15
- Copyright/Artist: Daniel Kaelen / DanielKaelen

**Step 2: Social Media Trail**
- Search Instagram for "DanielKaelen"
- Check recent stories
- Clue: "right outside the iriya gates"
- Clue: "catching the skyliner" (reference to Keisei Skyliner)
- Starbucks cup visible in photo

**Step 3: Location Pinpointing**
- "iriya gates" → JR Ueno Station Iriya Ticket Gates area
- Skyliner runs from Narita to Ueno
- Only Starbucks near these gates: Starbucks Coffee - JR Ueno Station Iriya Ticket Gates

**Step 4: Phone Number Extraction**
- Call the location or find number online
- Phone: +81 3-6231-7261
- Extract 10 digits: 0362317261

**Step 5: Hash Generation**
```bash
echo -n "0362317261" | md5sum
b3c5096d630b1c5bd14c677234ffa004
```

This challenge required combining multiple OSINT techniques: metadata analysis, social media investigation, geolocation verification, and cryptographic hashing. The real-world Tokyo Starbucks location added authenticity to the challenge.

---

## Tools & Techniques Used

**OSINT Tools:**
- what3words (web interface)
- Google Maps / Street View
- Social media search (Instagram, Twitter)
- Landmark research

**Forensics Tools:**
- exiftool (metadata extraction)
- mediainfo (video analysis)
- base64 (encoding/decoding)
- MD5 hashing (echo + md5sum)

**General Skills:**
- Roman numeral conversion
- GPS coordinate interpretation
- Historical landmark knowledge
- Malaysian geography
- Social engineering / social media investigation

---

## Key Takeaways

Week 3 emphasized that modern OSINT and forensics aren't just about technical tools, they're about **research methodology and connecting dots across multiple sources**. The challenges blended:

1. **Metadata analysis** — images and videos encode far more than visual content
2. **Geolocation intelligence** — GPS, landmarks, and services like what3words
3. **Social media investigation** — public posts often leak operational security
4. **Historical knowledge** — Malaysian landmarks and literary references
5. **Cryptographic verification** — hashing real-world data

The progression from static landmarks (Batu Caves, Shakespeare) to dynamic social media trails (DanielKaelen) to technical forensics (video metadata) showed Liga CTF's depth in real-world applicable skills.

---

## Flags Summary

| Challenge | Points | Flag |
|-----------|--------|------|
| Bay Watch | 100 | `OWASPKL{Chagar_Hutang}` |
| On the Run | 100 | `OWASPKL{white}` |
| A Bit to Add | 100 | `OWASPKL{blue}` |
| Shifting Hues | 100 | `OWASPKL{1852_1901_the_muse_of_poetry}` |
| A Lost Art | 100 | `OWASPKL{pl4y_7h3_vid}` |
| Data Guy | 100 | `OWASPKL{b3c5096d630b1c5bd14c677234ffa004}` |

---

# Happy Hacking

