---
title: Stanford CS110L Notes 1
description: Stanford CS110L Notes intro | lec01 | lec02
slug: rust-cs110l-notes-1
date: 2023-06-12 10:15:00+0000
image: cover.png
categories:
    - Study
tags:
    - Rust
    - Course Notes
    - CS110L
    - Technology
---

> Rust —— A language empowering everyone to build reliable and efficient software.

## Intro
### Official Course Website

[CS 110L: Safety in Systems Programming](https://reberhardt.com/cs110l/spring-2020/)

### My Homework Repository

[GitHub - UnicoCN/learn-rust](https://github.com/UnicoCN/learn-rust/tree/main)

## Lec01: Why Rust

- Why not C/C++?
    - Security Issue
        - buffer overflow
        - integer overflow(signed → unsigned)
- Why not GC’ed language?
    - downsides of GC
        - Expensive
        - Disruptive
        - Non-deterministic
        - Precludes manual optimization
    - can not solve memory leak completely

## Lec02: Memory Safety

- Why is it so easy to screw up in C?
    - Dangling Pointers
    - Double Frees
    - Iterator Invalidation
    - Memory Leaks
- Ownership Rules
    - Each value in Rust has a variable that’s called its `owner`
    - There can only be one owner at a time
    - When the owner goes out of scope, the value will dropped
- Lifetime

> Written by Jiacheng Hu, at Zhejiang University, Hangzhou, China.
