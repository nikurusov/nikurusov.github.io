---
layout: post
title: "Startup is Pain... Let’s Talk JVM Warmup"
date: 2025-05-05
tags: [jvm, exceptions, java, scala]
---
**All large JVM projects face the same issues: slow startup and sluggish throughput right after startup.**

In the JVM world, this is known as the [warmup stage](https://docs.azul.com/prime/analyzing-tuning-**warmup**). The **warmup stage** is the period of time during which the JVM starts the application and can be slow because of **runtime compilation (JIT), runtime class loading, JVM initialization, etc**.  
The JVM **interprets code before compiling** — that's the main problem. Without smart compiler optimizations, line-by-line code execution is just not fast.

So there are a couple of techniques to reduce the influence of, or even eliminate, the warmup stage:

1. **By-hand warmup**
    
2. **AOT compilation**
    
3. **Questionable future**
   
   
## By-hand warmup

It's useful for applications **that want to fix slow throughput after startup**: you just emulate load on your application, and that will trigger compilation processes so your code gets compiled.  

**But there are pitfalls.**


### *You need to emulate real load*

Input parameters should cover all cases. Methods you warm up must be real and used in the application flow. 
👉**The JVM recompiles methods if it sees they aren’t used as expected.**

### *Hard to emulate all logic*

**Complex business cases, integrations**. In business cases, you need to cover at least what gets called first, **so recompilation of methods underneath won’t cause recompilation above**.

Typical cases for integrations are **serializers and deserializers**: HTTP, DB, GRPC, JSON, etc. You `might` need to warm up the DB connection, but you **must** warm up serialization and deserialization.

### *It only makes sense if you can delay startup of your application*

This technique helps **only** with **throughput after startup**, **not** with **startup itself** — it may even make it worse.  
So it works for apps **where you want MAX performance before the app is used**.

**Example**:  
Web app → set readiness probe to `true` **after warmup**, e.g. 30s of load.  
But this doesn’t scale with **a ton of microservices** — you can’t reload them or add them on the fly.  
❌ So horizontal scaling becomes a problem.


**⚡ The most common and quickest way to get started. But to make it work well — use profilers!**


## AOT compilation

> In computer science, **ahead-of-time compilation** (**AOT compilation**) is the act of compiling an (often) higher-level programming language into an (often) lower-level language before execution of a program, usually at build-time, to reduce the amount of work needed to be performed at run time.

In the JVM, we have implementations of AOT: GraalVM Native Image, Kotlin Native, and Scala Native.

The biggest player here is the GraalVM AOT implementation (based on GitHub repo activity and overall production usage).

### GraalVM in their own words:

> _Native Image is a technology to compile Java code ahead-of-time to a binary — a_ **_native executable_**_. A native executable includes only the code required at run time — the application classes, standard-library classes, the language runtime, and statically-linked native code from the JDK.  
> An executable file produced by Native Image has several important advantages:
> - Uses a fraction of the resources required by the Java Virtual Machine, so it’s cheaper to run  
> - Starts in milliseconds  
> - Delivers peak performance immediately, with no warmup 
> - Can be packaged into a lightweight container image for fast and efficient deployment  
> - Presents a reduced attack surface

Those benefits are **real**, except for _"**peak performance** immediately, **with no warmup**"_ — we’ll get to that.

So, my **list of downsides** of GraalVM Native Image:

- It requires no-code changes and more engineering effort to add support for this tech in your project. Yeah, your project is unlikely to start without some additional config files for Native Image — but it’s not too hard, unless you use some company-god-delivered library with proxies, reflection, JNI, runtime class loading. In that case, GraalVM offers some tools to handle no-code adaptation, but in my project it didn’t work out (Hello `Netty`).
    
    
- 🐢 Build time. AOT compilers take time to build the executable, and Native Image is slower compared to others. Native Image needs to bundle all your Java runtime features into one executable — this takes time. So it could slow down your CI/CD pipeline. Locally you can use the JVM, but in CI/CD you must use Native Image to test that your app works as expected.
    
- Not all libraries support Native Image — they don’t have config files for it, but you can do it yourself.
    
- Peak performance of Native Image is close to peak performance of JIT C2, but C2 can be faster.
    


**⚡ Still, it’s a great tech and a must-use if you’re starting a big JVM project from scratch.**




## Questionable future

These technologies might blow our minds in the future, but for now they’re either not so popular or not mature enough in dev:

- [Cloud JVM](https://www.azul.com/glossary/cloud-native-jvm/)
    
- [CRaC](https://docs.azul.com/core/crac/crac-introduction)
    

The most interesting one for me is CRaC — at least it’s free to use :)


### CRaC

> The CRaC (Coordinated Restore at Checkpoint) Project researches coordination of Java programs with mechanisms to checkpoint (make an image of, snapshot) a Java instance while it is executing. Restoring from the image could be a solution to some of the problems with the start-up and warm-up times. The primary aim of the [Project](https://openjdk.org/projects/index.html) is to develop a new standard mechanism-agnostic API to notify Java programs about the checkpoint and restore events. Other research activities will include, but will not be limited to, integration with existing checkpoint/restore mechanisms and development of new ones, changes to JVM and JDK to make images smaller and ensure they are correct.

So, this technology provides mechanisms to save app state and restore from the saved app state. That state includes everything about the JVM itself, so you don’t need to init the JVM again at startup — it just loads everything from the checkpoint.

I use this technology for my university diploma, and **I have mixed feelings about i**t:

- You need to close resources before the snapshot and reopen them after restore. Sounds easy, but with third-party libraries it’s often tricky or even impossible. You have to check if your libraries give you any hooks for that.
    
- There’s no tooling to check what was or wasn’t loaded during the snapshot. If some class wasn’t touched before the checkpoint, you’ll get a `java.lang.NoClassDefFoundError` later. One way to avoid that — just force-load everything you _might_ need before snapshot.
    
- 🐧 CRaC only works on Linux for now. That’s a pain if your dev machine is macOS or Windows. They promise support for other OSes in the future, but for now — yeah, annoying.
    
- 🔐 The snapshot includes everything: heap, native memory, compiled code, config — literally your whole app state. Which means it can also include secrets or user data. So you have to be careful: either scrub memory before snapshot or lock down access to snapshot files.
    
- 🚫 If you’re on something other than x86_64 — like ARM64 — good luck. Most tools and images are built for x86_64. Running through emulators (like Rosetta on macOS) didn’t really work for me — low-level stuff just breaks.

**⚡Not for now...**

## Wrap-up

The most popular solutions were described! I **will show examples with techniques above in next blogs.** 

**Stay tuned — and have a good time!**
