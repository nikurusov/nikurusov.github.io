---
layout: post
title: "Exceptions are not free"
date: 2025-05-02
tags: [jvm, exceptions, java, scala]
---

**Did you know that creating exceptions can quietly hurt your performance?**

Many people use exceptions as a control flow tool, and it helps to:

- Keep the happy path clean and in place  
- Move error handling to where it fits better  
- Reduce code verbosity  
- Make the code look cleaner and more readable  

But **exceptions are not free in terms of memory and performance**.

To create an exception, the JVM needs to generate a stack trace — and this is far from free. Both CPU cost and memory usage grow with the depth of the call stack.

I write in Scala, and there's a common pattern to mark control-flow exceptions with `NoStackTrace` — a trait for exceptions which, for efficiency reasons, do not fill in the stack trace.

[Scala doc: `NoStackTrace`](https://www.scala-lang.org/api/3.x/scala/util/control/NoStackTrace.html)

This is primarily a JVM-related topic, but I believe any language where exceptions capture stack traces will face similar issues.

[Great read: The hidden performance costs of instantiating Throwables](http://normanmaurer.me/blog/2013/11/09/The-hidden-performance-costs-of-instantiating-Throwables)

> Does your memory profiler show exceptions hogging space? Here’s why.
