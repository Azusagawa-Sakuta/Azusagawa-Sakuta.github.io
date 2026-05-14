---
title: Attack Lab
date: 2026-05-14 10:56:00
tags: 
  - CSAPP
  - Assembly
categories:
  - Codes
  - Courses
---

# Attack Lab

Use gdb to get the stack pointer (%rsp)
Use regex and translate all possible mov / pop / nop / equal nop instructions
When using ROP attack, we should always remember that we did not use any call functions (cuz it's hard to find offset), but the `ret` instruction will return to the address of what popped from stack.

It may help you in finding assembly gadget in [farmer.asm](https://github.com/Azusagawa-Sakuta/CSAPP/blob/main/lab/03%20attack%20lab/target1/farmer.asm)
