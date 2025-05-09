---
layout: post
title: "Google blocks Yandex: A Tale of Two robots.txt Files"
date: 2025-05-09
tags: [google, yandex, seo, webindexing, robots, searchengines]
---

While setting up my personal blog, I checked the `robots.txt` file — **and was surprised to see that Google is quietly blocking Yandex, and only Yandex, from indexing some parts of its site.** That’s not just a technical detail — it’s a sign of how search engines compete behind the scenes.

For those unfamiliar, `robots.txt` is a small file websites use to tell search engines what they can and can't access. Most of the time, it's simple and boring — but sometimes it reveals something much bigger.

## Google Blocks Yandex — Explicitly

Google’s `robots.txt` includes a very specific directive:

```
User-agent: Yandex
Disallow: /search
Disallow: /about/careers/applications/jobs/results
```

This means:

- **Yandex is blocked from crawling Google Search results**
    
- **Yandex cannot index Google Careers job listings**
    

But here’s the kicker: **Bing, DuckDuckGo, and Yahoo are _not_ blocked.**

### Why target Yandex?

- Perhaps Yandex previously scraped search results or structured job listings.
    
- Maybe Google wants to **block a direct competitor in CIS markets**.
    
- Or it could be a **strategic move to restrict access** to its most valuable data sources.
    

## Yandex: Caution, Consistency, Control

Yandex takes a different approach. Its `robots.txt` shows a broad and consistent defensive policy:

```
User-agent: *
Disallow: /search
Disallow: /news
```

Yandex **blocks all crawlers**, including its own, from indexing its search and news sections. No special targeting. No exceptions. Just uniform rules across the board — a clear contrast to Google’s selective blocking.

## A Curious Timeline

Here’s where it gets interesting:  
**Google only added the Yandex block in April 2025.**

🔎 [April 28, 2025 — no Yandex block](https://web.archive.org/web/20250428032950/https://www.google.com/robots.txt)  
🔒 [April 29, 2025 — Yandex block appears](https://web.archive.org/web/20250429040230/https://www.google.com/robots.txt)

It wasn’t always there. It was **a deliberate update**, recently and surgically introduced.

## Final Thoughts

From a technical and ethical standpoint, **Yandex’s approach feels more transparent**. It applies the same rules to everyone. Google, on the other hand, targets just one rival.

---
**What do you think — is this fair play or gatekeeping?**
