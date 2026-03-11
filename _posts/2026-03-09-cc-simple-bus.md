---
layout: post
title: "Simple Event Bus in CC Printer — Here's how its working"
date: 2026-03-09
author: "MadRajib Lab"
tags: [C, Event Bus]
description: "Working of a simple event bus in Elegoo Centauri 3D printer."
---

Recently I bought a Elegoo Centauri Carbon 3D printer. Elegoo have chosen a custom firmware for the printer rather than using klipper and fluidd.
And because I like to understand how it works, I decided to dig into the firmware to understand its working.
So I jumped into the source code and started wondering around.

After tracing the call stack visually my attention landed in a module same `simple_bus.c`.
Digging around a bit I got to know that it is some sort of an event bus for handling service request and msg packets.

In this post I will try to explain what I have learned so far about the simple bus and what improvement I decided to add to it.

## The Data Structures

Here there are couple of structures used to implement the bus. The very first structure is `simple_bus_t`
```c
typedef struct simple_bus
{   
    /* suscriber list */
    list_head_t subscribe_name_list;

    /* async msg list */
    list_head_t async_msg_list;
    pthread_mutex_t async_msg_list_lock;
    pthread_cond_t async_msg_list_cond;

    /* services list */
    list_head_t service_list;
    pthread_mutex_t service_list_lock;

    pthread_mutex_t lock;
    pthread_t pid;
} simple_bus_t;
```