---
title: Bomb Lab
date: 2026-05-14 10:11:00
tags: 
  - CSAPP
  - C
  - Assembly
categories:
  - Codes
  - Courses
---

# Bomb Lab

This may be the most famous lab across all CSAPP labs, which requires us to defuse a bomb by reading the assembly (technically reverse engineering).

All thoughts are in comments of the `bombf.asm` in my [GitHub Repo](https://github.com/Azusagawa-Sakuta/CSAPP/blob/main/lab/02%20bomb%20lab/bomb/bombf.asm).

In this lab, I used my MacBook to defuse bombs, which is uncompatible with lab environment. Dockers with x86 platform can help run programs and read inputs, but I cannot debug with gdb on it.

The error message is: "Couldn't read debug register: Input/output error."

Depressedly, I encountered this bug in around 2025, but it has been still an [open issue](https://github.com/docker/for-mac/issues/6921) right now. Also, I have tried the advice provided in the link above, but Rosetta is too slow and troublesome for my device. In that, I have mannually read the plain asm code and finish this lab in a more complicated way, but I think that strongly improved my assembly code.

There should be another way to debug, such as VM/Dual OS. But due to my poor ROM space, I gave up Dual OS. And due to the poor running speed of VM(x86 arch) on arm, I gave up VM, too. If your RAM/ROM either is big enough, you can technically try these ways and enjoy the travel of debugging. But there is a plenty of time to debug on future labs (I can debug the obj files I complied), I chose to read the assembly and strengthen my insight of obj files itself.

If you are like me or you want to complete the lab in above way, it will be helpful for you to run `objdump -d bomb > bomb.asm` to disassemble and get the data in .rodata section (storing string literal and jump table).

Good Luck!
