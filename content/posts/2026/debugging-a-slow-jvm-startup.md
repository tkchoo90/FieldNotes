---
title: "Debugging a Slow JVM Startup"
date: 2026-10-28T09:00:00+08:00
draft: false
tags: ["java", "performance", "debugging"]
summary: "A sample write-up: how I tracked down a 14-second startup, with code, logs, and a table of measurements."
---

This is a sample post showing the formatting a technical write-up typically needs.

## The problem

A service took **14 seconds** to start in staging but only 3 seconds locally.

## Investigation

I enabled class-loading and GC logs:

```bash
java -Xlog:class+load:file=classload.log -Xlog:gc*:file=gc.log -jar app.jar
```

Then looked for the slowest phases:

```java
long start = System.nanoTime();
context.refresh();
log.info("Context refresh took {} ms", (System.nanoTime() - start) / 1_000_000);
```

## Findings

| Phase | Local | Staging |
|-------|-------|---------|
| Class loading | 0.8s | 1.1s |
| Bean init | 1.5s | 2.0s |
| DNS lookup | 0.1s | 10.5s |

The DNS lookup was the culprit: a reverse lookup against an unreachable resolver.

## Takeaways

- Measure before guessing.
- Compare environments, not just code.
- Write down what you ruled out, too.

> Tip: keep a running log of experiments while debugging. It makes the write-up almost free.
