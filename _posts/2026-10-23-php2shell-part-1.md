---
title: "PHP2Shell Part 1: Understanding the Vuln - 'Theory'"
date: 2026-10-23 13:20:00 +0800
categories: [Security, Vulnerability Analysis]
tags: [php, php2shell, cve-2026-109618, cve-2026-109619, rce, use-after-free, opcache, web-security]
image:
    path: /images/php2shell-images/php2shell.jpg
    alt: PHP2Shell Vuln
description: "Part 1 of a 2-part series breaking down PHP2Shell (CVE-2026-109618 & CVE-2026-109619), the pair of critical PHP core RCE bugs patched on 21 Oct 2026. This part is all theory. Part 2 we pop it in a lab."
---

## Intro

So on 21 October 2026 the PHP team shipped an emergency out-of-band patch, and the whole infosec timeline lost its mind. The reason has a name now: **PHP2Shell**.

This is a pair of critical bugs that live inside the PHP engine itself, not in some random plugin or theme. That distinction is the scary part. It means the vulnerable code runs underneath WordPress, Drupal, Joomla, Laravel, basically anything built on PHP, even a bare "Hello World" page. Wiz Research, who found it, put the blast radius at around 70% of all websites on the planet.

This is a two part series. Part 1 (this one) is the theory: what the bugs are, how the disclosure went down, and why everyone is sweating. Part 2 is the fun part where we actually exploit it inside my own lab that dropped for it. Let's get into it.

## TL;DR

| Attribute        | Detail                                                    |
| ---------------- | --------------------------------------------------------- |
| Name             | PHP2Shell                                                 |
| CVE IDs          | CVE-2026-109618, CVE-2026-109619                          |
| Severity         | CVSS v4.0 9.2 (both), Critical                            |
| Type             | Unauthenticated Remote Code Execution + memory disclosure |
| Affected         | Every PHP release going back to 5.4                       |
| Patched versions | 8.5.12, 8.4.27, 8.3.36, 8.2.35                            |
| Discovered by    | Wiz Research, with Calif and OpenAI                       |
| Patch date       | 21 October 2026                                           |

> Only the 8.2 through 8.5 branches got a fix. Anything on 8.1 or older is end of life, so there is no patch coming. If you are stuck there, the only move is upgrading the whole branch.
{: .prompt-danger }

## How it blew up

The messy part is that this was supposed to be a coordinated disclosure on 22 October. It leaked a day early and everything accelerated.

- **19 Aug 2026:** Wiz Research finds the first bug.
- **20 to 21 Aug:** They confirm unauthenticated RCE against Drupal, then WordPress.
- **24 Aug:** Wiz reports the whole thing to the PHP security team.
- **21 Sep:** PHP acknowledges and starts building a coordinated fix.
- **20 Oct:** Details leak. A since-deleted Hacker News post and screenshots of an exploit tool literally called "PHP2Shell" (credited to the firm Calif) spread online. CERT-EU drops advisory 2026-018. PHP posts an advance notice of an emergency release.
- **21 Oct:** PHP ships the patches early (around 13:34 UTC, ahead of the announced 14:00). Public disclosure follows. Honeypots at watchTowr, plus Aikido and Patchstack, start seeing exploitation attempts in the wild the same day.

Plot twist for the timeline nerds: Amazon's threat intel team says they caught a probe for this on **17 October**, four days before any public disclosure, coming from an anonymized IP. So somebody had info before the rest of us.

## What is actually broken

Here is the root cause in plain language before we touch the specific bugs.

PHP processes a web request, and sometimes a request dies badly in the middle, most importantly when it hits an **out-of-memory (OOM)** error. When that happens, PHP tears down the request's memory. The problem is that some internal bookkeeping does not get cleaned up properly. It gets left behind in a corrupted or stale state.

Now remember how PHP works in production: a single worker process handles many requests one after another (mod_php, PHP-FPM, and friends). So the *next* request that lands on that same worker can come along and reuse that stale leftover state. That reuse is the whole ballgame. Depending on which piece of state got left dangling, an attacker can either read memory they should never see, or corrupt memory into code execution.

This is a classic family of memory-safety issues: **use-after-free** and **type confusion**. The twist that makes PHP2Shell special is that it happens *across requests*, triggered by something as simple as forcing the server to run out of memory.

> These bugs sit *below* the application layer. Your own code can be 100% clean and you are still exposed, because the flaw is in the runtime your code sits on top of.
{: .prompt-warning }

## The four bugs under two CVEs

Wiz actually reported four separate bugs, which the PHP team grouped under the two CVEs. Here is the conceptual breakdown of each.

### CVE-2026-109618

**1. Upload-progress unserializer use-after-free.** PHP's session module can decode session data during upload-progress handling, very early in the request lifecycle. An OOM failure mid-way can leave the unserializer's global bookkeeping pointing at freed memory. A later request on the same worker then treats attacker-controlled memory as legit unserializer state, which turns into a memory-corruption primitive strong enough for RCE. Wiz demonstrated pre-auth RCE on WordPress, Drupal, and Joomla with this.

**2. Remote process-memory disclosure.** This is the Heartbleed-style one. During multipart request parsing, PHP can leave a session ID pointer aimed at freed memory after an OOM abort. A follow-up request gets the server to URL-encode that reclaimed heap memory straight into the `PHPSESSID` response cookie. The wild part: it works even if the app never calls `session_start()`, so it is completely application-agnostic. That leaked memory can contain passwords, tokens, API keys, and certs belonging to other requests.

### CVE-2026-109619

**3. Fatal-error backtrace use-after-free.** PHP holds onto the backtrace of the most recent fatal error in a global. An OOM error can wipe the request heap without clearing that global, so it is left dangling. If the same worker logs a diagnostic early in the next request, it follows that dangling pointer into attacker-reclaimed memory. Wiz built this into pre-auth RCE against Drupal.

**4. OPcache type confusion.** OPcache caches compiled PHP in shared memory across workers using index maps. An allocation failure at exactly the wrong moment leaves the local and shared state out of sync, so a slot gets reused for an object of the wrong type. That cross-worker type confusion corrupts shared OPcache state, and Wiz chained it into unauthenticated RCE on WordPress.

> Patching your app code does nothing here. Config hardening does not fully save you either. The PHP maintainers were clear: updating the PHP runtime is the only complete fix.
{: .prompt-info }

## Who is in the danger zone

Every SAPI that serves web requests is potentially affected: PHP-FPM, Apache's mod_php, php-cgi, FrankenPHP, LiteSpeed. The PHP team flagged **mod_php as the highest risk** because the attack can execute before your application code even starts running.

And the exposure is huge. Scans right after disclosure counted roughly 2.24 million internet-facing PHP hosts, with well over half sitting on versions that have no fix coming. Dark web chatter was already calling it open season.

## Blue-team quick wins

If you run PHP in prod, in priority order:

1. **Patch now** to 8.5.12, 8.4.27, 8.3.36, or 8.2.35. On an EOL branch, upgrade the branch.
2. If you genuinely cannot patch this second, many CDNs and WAFs (Cloudflare, Fastly, Akamai and others) shipped virtual patches. Lean on those as a temporary bandage, not a cure.
3. **Hunt for compromise** going back a few months. Look for web shells, modified or unexpected PHP files, new admin accounts, and weird outbound connections.
4. **Watch the logs** for the tell-tale signs: odd multipart requests followed by memory-limit errors or repeated HTTP 500s, unusually large or malformed `PHPSESSID` response cookies, and unexplained PHP worker churn.

## The part nobody can ignore

One last thing before we close Part 1. The vulnerable code sat in PHP for over a decade untouched. What changed in 2026 was not the code, it was the attacker. Wiz did this research with frontier AI in the loop (Wiz Atlas, Gemini 4 Argon, and GPT-5.6 Sol), and they are openly framing PHP2Shell as the Heartbleed or Log4Shell moment of the AI era. The bug was always exploitable in theory, AI just made it practical to pull off and scale. That is a whole conversation on its own, and maybe a Part 3 later.

## Coming up in Part 2

Theory's done. Next post we go hands-on inside TryHackMe's [PHP2Shell room](https://tryhackme.com/room/cvephp2shell), spin up the vulnerable target, and walk the solve end to end to grab the flag. Lab-only, authorized environment, the way CTF writeups are meant to be.

Heads up, it's a Premium/Max room, so you'll need a subscription to launch it.

Stay patched, stay curious.

`T4R0T`

![Happy Hacking](https://i.pinimg.com/originals/ca/e4/a7/cae4a766bf2fdbed1e1cc48da3d12225.gif)

> Full walkthrough dropping in Part 2. Patch your own servers before you go poking at anyone's.
{: .prompt-tip }
