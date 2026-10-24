---
name: social-safe-reading
description: Use BEFORE an AI agent reads, scrolls or researches anything on a social network (LinkedIn, Facebook, Instagram, X, Reddit, TikTok, YouTube) through the user's browser, a browser extension, a CDP/Playwright browser or any scraper. Rules that keep the user's account from being restricted, blocked or banned.
---

# Reading social networks without getting the account locked

Every major network forbids automated reading in its terms, even read-only and even while logged in.
These rules keep an agent's reading close to what a person does, so the account stays safe.
The risk is low when they are followed, but it is never zero.

## Core rules (all networks)

1. **The user's own logged-in browser tab first**, read through an extension the user controls. Never a
   headless browser, never Playwright/Puppeteer or a CDP-driven browser on a high-risk network
   (LinkedIn, Facebook, Instagram, TikTok): they expose `navigator.webdriver` and headless fingerprints.
2. **One target per task.** One profile, one thread, one page. Do not follow links to other profiles.
3. **Human pace.** Scroll one screen, read, wait 5-10 seconds. About 15-20 posts or 5 minutes per target.
4. **Read only.** Never like, comment, follow, connect, message, vote, share or click any action button.
   Never type a password, never submit a form.
5. **Text on screen only.** No downloads, no bulk export, no saving media, no opening every post.
6. **Stop at once** on any checkpoint, captcha, "unusual activity", "temporarily blocked", "action blocked",
   "rate limit exceeded" or sign-in prompt. Tell the user in one line. Do not retry the same day.
7. **Safest option first:** the user scrolls, the agent only reads what is already loaded on screen.
8. **Prefer an official route** when one exists (a public API, an RSS feed, a public page without login).
9. Keep a daily total across all agents low; spread reads over the day.

## Per network

| Network | What the terms say | Extra risk | Extra rule |
|---|---|---|---|
| LinkedIn | User Agreement 8.2: no software, scripts, bots, crawlers or browser add-ons to scrape or copy the services, including profiles | Strictest. Page scans for thousands of known extensions and fingerprints the browser (reported April 2026). Scraper users reported restricted within ~48 h | Max ~80 profile views a day in total, far below for agents. Never the automated browser |
| Facebook | Terms: no access or collection of data by automated means without prior permission, "regardless of whether ... while logged-in" | Meta blocks fast or repeated actions ("You're temporarily blocked", "going too fast"); checkpoints can lock the account | Groups and profiles: one at a time, slow. Read-only even in groups the user is a member of |
| Instagram | Terms of Use: no accessing or collecting information in an automated way without express permission, also when logged in | "Action blocked" and temporary limits on fast scrolling/opening many profiles (unverified exact numbers) | Do not open many profiles or stories in a row. No automated browser |
| X | Terms: no scraping without X's express written permission; no access other than through published interfaces; no working around technical limits | Daily read limits per account and "Rate limit exceeded" (numbers change and are not published reliably) | Stop on the first rate-limit notice. Prefer search pages the user opens himself |
| Reddit | User Agreement: no access, search or collection by automated means except as permitted; crawling only per robots.txt; scraping without consent prohibited | Lower account risk for reading; public pages and `.json` / RSS readable without login | Prefer public pages without login. Never vote or post |
| TikTok | Terms of Service: no automated scripts to collect information from or interact with the services | Aggressive bot detection; logged-in automation can flag the account | Read in the user's tab only; no automated browser |
| YouTube | Terms: no access by automated means (robots, botnets, scrapers), except public search engines per robots.txt | Lower account risk; the official YouTube Data API exists | Use the official API or the user's tab; never automate the logged-in account |

## Honest risk to state when asked

Following these rules keeps reading close to a person's, so the risk is low but not zero: on paper any extension
or script that reads the page breaks the terms of every network above, and networks change their detection
without notice.

## Sources (fetched 20.10.2026)

- LinkedIn User Agreement 8.2: https://www.linkedin.com/legal/user-agreement
- LinkedIn extension scanning (April 2026): https://www.bleepingcomputer.com/news/security/linkedin-secretely-scans-for-6-000-plus-chrome-extensions-collects-data
- Facebook Terms of Service: https://www.facebook.com/legal/terms
- Instagram Terms of Use: https://help.instagram.com/581066165581870
- X Terms of Service: https://x.com/en/tos
- Reddit User Agreement: https://redditinc.com/policies/user-agreement
- TikTok Terms of Service: https://www.tiktok.com/legal/page/row/terms-of-service/en
- YouTube Terms of Service: https://www.youtube.com/static?template=terms
- Unverified: exact action/rate limits for Facebook, Instagram, X and TikTok (not published by the networks;
  numbers found online change often). Treat any limit notice as the ceiling.
