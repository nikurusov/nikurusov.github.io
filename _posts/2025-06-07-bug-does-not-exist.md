---
layout: post
title: "Bug that does not exist"
date: 2025-06-07
tags: [browser, curl, web, programming, bug]
---

**TL;DR:** Browser and clients act differently. Always Google your problem first—maybe the issue you're trying to solve isn't really a problem.

It's a classic story: you spend hours trying to fix something that doesn't actually exist...

Recently at work, I ran into an issue with HTTP streaming.

**Example:**

1. User sends a `/download` request
2. Server responds with a stream and a **Content-Length** header
3. The user doesn't receive the full body because the server interrupted the request
4. User keeps waiting for the rest of the body

Check it with Python, the client won't wait for the body—the issue doesn't appear. But **curl** and **browser** does different.

**What's going on?**

Browsers rely on the **Content-Length** header, even when it doesn't seem logical.  
So, if your picture is 15 KB but you specify a higher **Content-Length**, **the stream will never finish**. If you specify a lower value, you'll get a corrupted, cut-off image.

By the way, at first, I thought it was a library bug. I forked the repo and started writing tests (it was the Scala Tapir library), but couldn't reproduce the issue. That's when I realized what was really going on. :)
