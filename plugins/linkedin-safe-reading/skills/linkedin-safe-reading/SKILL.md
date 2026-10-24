---
name: linkedin-safe-reading
description: Use BEFORE an AI agent reads, scrolls or researches anything on LinkedIn (a profile, activity page, posts, job pages) through the user's browser, a browser extension, Playwright or any scraper. Rules that keep the user's LinkedIn account from being restricted or locked.
---

# Reading LinkedIn without getting the account locked

These rules apply to every agent that reads LinkedIn on the user's behalf.

## Why

- LinkedIn User Agreement section 8.2 forbids scripts, bots, crawlers and browser add-ons that scrape or copy
  profiles, even read-only. https://www.linkedin.com/legal/user-agreement
- Since April 2026 LinkedIn's page scans for 6,000+ known extensions and fingerprints the browser
  (navigator.webdriver, headless canvas/WebGL, activity rate). Users of known scrapers report restrictions
  within about 48 hours.
  https://www.bleepingcomputer.com/news/security/linkedin-secretely-scans-for-6-000-plus-chrome-extensions-collects-data
- Commonly cited safe ceiling: about 80-100 profile views a day on a free account.
  https://www.datablist.com/how-to/linkedin-account-restriction

## Rules

1. **The user's own, already logged-in browser tab only.** Never a headless browser, never Playwright or a
   remote-debugging (CDP) browser for LinkedIn reading: they carry the webdriver flag.
2. **One profile per task.** Do not open other profiles linked from it.
3. **Human pace:** scroll one screen, read, wait 5-10 seconds. At most about 15-20 posts or about 5 minutes per
   profile. Do not open every post.
4. **Read only.** Never like, comment, connect, message, follow or click any action button. Never type a password.
5. **Text on screen only.** No downloads, no bulk export, no saving images.
6. **Stop at once** on a checkpoint, captcha, "unusual activity" notice or sign-in prompt, and tell the user in
   one line.
7. **Safest option:** the user scrolls and the agent only reads what is already loaded. Offer it first when the
   task is not urgent.
8. The daily total across all agents stays far below 80 profile views.

## Honest risk to state when asked

Low for one profile at human pace in the user's own browser, but not zero: on paper any extension that reads the
page breaks section 8.2, and it cannot be proven from outside whether a given extension is on LinkedIn's scanned
list.
