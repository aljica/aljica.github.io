---
layout: post
title: The Linux Programming Interface Chapter 5 File I/O Further Details
categories: tlpi
---

[5.1 Atomicity and Race Conditions](#5-1-atomicity-and-race-conditions)

# 5 1 Atomicity and Race Conditions

* System calls are atomic
    * This means that they are uninterrupted; this avoids race conditions.

![Image](/docs/assets/images/5.1-atomicity.png)