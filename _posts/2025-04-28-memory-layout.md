---
layout: post
title: "Cache Lines, Padding, and the Struct Layout That Doubled My Performance"
date: 2025-04-28
author: "Alex Mercer"
tags: [C, Performance]
description: "A 2× speedup hidden in 8 bytes of reordering. How struct field order can make or break your hot path."
---

A colleague once told me struct layout was a micro-optimization not worth thinking about.
Then we reordered six fields in a hot struct and shaved 40ms off a 80ms request cycle. *That's* when I started paying attention.

## What Is a Cache Line?

Modern CPUs don't fetch memory one byte at a time. They fetch 64-byte chunks called **cache lines**.
When your code reads a single `int`, the CPU loads the surrounding 63 bytes too, placing them in the L1 cache.

If your next access falls in the same cache line — free. If not — another trip to RAM, easily 200+ cycles.

## The Problem: False Sharing and Wasted Space

Consider this struct:

```c
typedef struct {
    char     flag;       /* 1 byte  */
    double   value;      /* 8 bytes */
    int      count;      /* 4 bytes */
    char     name[3];    /* 3 bytes */
} record_t;
```

What's its size? You might guess 16. Run `sizeof(record_t)` and you'll likely get **24**.
The compiler inserted 7 bytes of padding after `flag` to align `double` to an 8-byte boundary.

## The Fix: Sort Fields by Alignment

The rule is simple: **largest fields first, smallest last.**

```c
typedef struct {
    double   value;      /* 8 bytes — 8-byte aligned, no padding */
    int      count;      /* 4 bytes — 4-byte aligned, no padding */
    char     flag;       /* 1 byte  */
    char     name[3];    /* 3 bytes */
    /* total: 16 bytes — 8 bytes saved */
} record_t;
```

Same data, 8 fewer bytes. In a hot loop over millions of records, those 8 bytes per element
dramatically reduce cache pressure.

## Verify with pahole

The `pahole` tool (part of `dwarves`) shows you exactly where padding hides:

```sh
$ pahole ./your_binary
struct record_t {
    char     flag;            /*     0     1 */
    /* XXX 7 bytes hole */
    double   value;           /*     8     8 */
    int      count;           /*    16     4 */
    char     name[3];         /*    20     3 */
    /* size: 24, cachelines: 1, members: 4 */
    /* sum members: 16, holes: 1, hole bytes: 7 */
};
```

## When to Break the Rule

Sometimes you *want* padding between fields — specifically to prevent **false sharing**
between fields written by different threads. In that case, align hot fields to cache line
boundaries with `__attribute__((aligned(64)))` or C11's `_Alignas(64)`.

```c
typedef struct {
    _Alignas(64) atomic_int reader_count;  /* own cache line */
    _Alignas(64) atomic_int writer_count;  /* own cache line */
} counters_t;
```

Two threads can now hammer their respective counters without invalidating each other's
cache lines. This technique is called **cache line padding** and it's one of the first
things to try in high-contention concurrent code.
