---
title: Arch Lab
date: 2026-05-14 11:15:00
tags: 
  - CSAPP
  - Assembly
  - HCL
categories:
  - Codes
  - Courses
---

# Arch Lab
## Part A
### Exp 1
When running rsum_list (rsum.ys), I accidentally set the pos of stack to 200 (normally 0x200). Then when pushing the callee-saved registers and the PCs to record call functions, the stack grows to 0x88, where is the code of `addq %rbx, %rax`. So it changes the code itself just like what we do in attack lab. However this change halt the program normally.
```txt
Stopped in 29 steps at PC = 0x8c.  Status 'HLT', CC Z=1 S=0 O=0
Changes to registers:
%rbx:   0x0000000000000000      0x0000000000000c00
%rsp:   0x0000000000000000      0x0000000000000088

Changes to memory:
0x0088: 0x000000903fb03060      0x0000000000000088
0x0090: 0x0000000000000000      0x00000000000000b0
0x0098: 0x0000000000000000      0x0000000000000088
0x00a0: 0x0000000000000000      0x000000000000000a
0x00a8: 0x0000000000000000      0x0000000000000088
0x00b8: 0x0000000000000000      0x000000000000005b
0x00c0: 0x0000000000000000      0x0000000000000013
```
### Exp 2
When last line is not newline, we get an error:
```txt
Error on line 49: Missing end-of-line on final line

Line 49, Byte 0x0200:     .pos 0x200
```

### Exp 3
When mistype `ret` in the beginning of execution:
```txt
	.pos 0x0
    irmovq stack, %rsp
    call main
    ret
```
We will get:
```txt
root@52f6d287284f:/work/_labs/04 arch lab/archlab-handout/sim/misc# ./yis copy.yo 44 
Stopped in 44 steps at PC = 0x13.  Status 'AOK', CC Z=1 S=0 O=0
Changes to registers:
%rax:   0x0000000000000000      0x0000000000000cba
%rsp:   0x0000000000000000      0x0000000000000200
%rsi:   0x0000000000000000      0x0000000000000048
%rdi:   0x0000000000000000      0x0000000000000030
%r10:   0x0000000000000000      0xffffffffffffffff

Changes to memory:
0x0030: 0x0000000000000111      0x000000000000000a
0x0038: 0x0000000000000222      0x00000000000000b0
0x0040: 0x0000000000000333      0x0000000000000c00
0x01f0: 0x0000000000000000      0x000000000000006f
0x01f8: 0x0000000000000000      0x0000000000000013
root@52f6d287284f:/work/_labs/04 arch lab/archlab-handout/sim/misc# ./yis copy.yo 45
Stopped in 45 steps at PC = 0x0.  Status 'AOK', CC Z=1 S=0 O=0
Changes to registers:
%rax:   0x0000000000000000      0x0000000000000cba
%rsp:   0x0000000000000000      0x0000000000000208
%rsi:   0x0000000000000000      0x0000000000000048
%rdi:   0x0000000000000000      0x0000000000000030
%r10:   0x0000000000000000      0xffffffffffffffff

Changes to memory:
0x0030: 0x0000000000000111      0x000000000000000a
0x0038: 0x0000000000000222      0x00000000000000b0
0x0040: 0x0000000000000333      0x0000000000000c00
0x01f0: 0x0000000000000000      0x000000000000006f
0x01f8: 0x0000000000000000      0x0000000000000013
```
Noticed that when returned from the first part, our program will restart.

## Part B
Need to investigate the implementation of codes

## Part C
Due to `%rax` was initialized to 0, and we can rearrange the instructions to avoid load/use hazard. And `iaddq $-1, %rdx` will set the CC automatically.
```ncopy-ys-editable
# You can modify this portion
	# Loop header
	andq %rdx,%rdx		# len <= 0?
    jg Loop
    ret
Loop:
	mrmovq (%rdi), %r10	# read val from src...
	iaddq $8, %rdi		# src++
	andq %r10, %r10		# val <= 0?
	rmmovq %r10, (%rsi)	# ...and store it to dst
	jle Npos		# if so, goto Npos:
    iaddq $1, %rax      # count++
Npos:
	iaddq $8, %rsi		# dst++
	iaddq $-1, %rdx		# len--
	jg Loop			# if so, goto Loop:
```
We got:
```txt
Average CPE     10.47
Score   0.5/60.0
```

Then we tried to separate the loop into 8 branches:
```ncopy-ys-editable
# You can modify this portion
    iaddq $-8, %rdx             # 0 1 2 3 4 5 6 7
    jge Loop
DealRemainder:
    iaddq $4, %rdx              # -8 -7 -6 -5
    jge four2seven              # -4 -3 -2 -1
    iaddq $3, %rdx              # -4 -3
    jg two2three                # -1 0
    je one
    jmp zero
four2seven:                     # 0 1 2 3
    iaddq $-2, %rdx             # -2 -1 0 1
    jl four2five                # 0 1
    jg seven
    jmp six
two2three:                      # 1 2
    iaddq $-1, %rdx
    jg three
    jmp two
four2five:
    iaddq $1, %rdx
    jl four
    jmp five
Loop:
    mrmovq 0x38(%rdi), %rbx
    mrmovq 0x30(%rdi), %r14
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
    andq %rbx, %rbx
    rmmovq %rbx, 0x38(%rsi)
    jle Mv7
    iaddq $1, %rax
Mv7:
    andq %r14, %r14
    rmmovq %r14, 0x30(%rsi)
    jle Mv6
    iaddq $1, %rax
Mv6:
    andq %r13, %r13
    rmmovq %r13, 0x28(%rsi)
    jle Mv5
    iaddq $1, %rax
Mv5:
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle Mv4
    iaddq $1, %rax
Mv4:
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle Mv3
    iaddq $1, %rax
Mv3:
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle Mv2
    iaddq $1, %rax
Mv2:
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle Mv1
    iaddq $1, %rax
Mv1:
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle Mv0
    iaddq $1, %rax
Mv0:
    iaddq $64, %rdi         # src += 8;
    iaddq $64, %rsi         # dst += 8;
    iaddq $-8, %rdx
    jge Loop
    jmp DealRemainder
seven:
    mrmovq 48(%rdi), %r14
    andq %r14, %r14
    rmmovq %r14, 48(%rsi)
    jle M6
    iaddq $1, %rax
M6:
six:
    mrmovq 40(%rdi), %r13
    andq %r13, %r13
    rmmovq %r13, 40(%rsi)
    jle M5
    iaddq $1, %rax
M5:
five:
    mrmovq 0x20(%rdi), %r12
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle M4
    iaddq $1, %rax
M4:
four:
    mrmovq 0x18(%rdi), %r11
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle M3
    iaddq $1, %rax
M3:
three:
    mrmovq 0x10(%rdi), %r10
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle M2
    iaddq $1, %rax
M2:
two:
    mrmovq 0x8(%rdi), %r9
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle M1
    iaddq $1, %rax
M1:
one:
    mrmovq (%rdi), %r8
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle M0
    iaddq $1, %rax
M0:
zero:
```
We got:
```text
Average CPE     7.79
Score   54.3/60.0
```
Noticed that when we use same tags, the status of `PC` is unpredictable.

Then deal with bubbles:
```ncopy-ys-editable
# You can modify this portion
    iaddq $-8, %rdx             # 0 1 2 3 4 5 6 7
    jge Loop
DealRemainder:
    iaddq $4, %rdx              # -8 -7 -6 -5
    jge four2seven              # -4 -3 -2 -1
    iaddq $3, %rdx              # -4 -3
    jg two2three                # -1 0
    je one
    ret
four2seven:                     # 0 1 2 3
    iaddq $-2, %rdx             # -2 -1 0 1
    jl four2five                # 0 1
    jg seven
    jmp six
two2three:                      # 1 2
    iaddq $-1, %rdx
    jg three
    jmp two
four2five:
    iaddq $1, %rdx
    jge five
    jmp four
Loop:
    mrmovq 0x38(%rdi), %rbx
    mrmovq 0x30(%rdi), %r14
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
    andq %rbx, %rbx
    rmmovq %rbx, 0x38(%rsi)
    jle Mv7
    iaddq $1, %rax
Mv7:
    andq %r14, %r14
    rmmovq %r14, 0x30(%rsi)
    jle Mv6
    iaddq $1, %rax
Mv6:
    andq %r13, %r13
    rmmovq %r13, 0x28(%rsi)
    jle Mv5
    iaddq $1, %rax
Mv5:
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle Mv4
    iaddq $1, %rax
Mv4:
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle Mv3
    iaddq $1, %rax
Mv3:
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle Mv2
    iaddq $1, %rax
Mv2:
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle Mv1
    iaddq $1, %rax
Mv1:
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle Mv0
    iaddq $1, %rax
Mv0:
    iaddq $64, %rdi         # src += 8;
    iaddq $64, %rsi         # dst += 8;
    iaddq $-8, %rdx
    jge Loop
    jmp DealRemainder
seven:
    mrmovq 0x30(%rdi), %r14
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
    andq %r14, %r14
    rmmovq %r14, 0x30(%rsi)
    jle M6
    iaddq $1, %rax
six:
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
M6:
    andq %r13, %r13
    rmmovq %r13, 0x28(%rsi)
    jle M5
    iaddq $1, %rax
five:
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
M5:
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle M4
    iaddq $1, %rax
four:
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
M4:
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle M3
    iaddq $1, %rax
three:
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
M3:
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle M2
    iaddq $1, %rax
two:
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
M2:
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle M1
    iaddq $1, %rax
one:
    mrmovq (%rdi), %r8
M1:
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle M0
    iaddq $1, %rax
M0:
zero:
```
Then:
```txt
        ncopy
0       Program too long
```
We should decrease the program length. Noticed that there are multiple useless `mrmovq #(%rdi), %r##`. We change it to:

```ncopy-ys-editable
# You can modify this portion
    iaddq $-8, %rdx             # 0 1 2 3 4 5 6 7
    jge Loop
DealRemainder:
    iaddq $4, %rdx              # -8 -7 -6 -5
    mrmovq (%rdi), %r8
    jge four2seven              # -4 -3 -2 -1
    iaddq $3, %rdx              # -4 -3
    jg two2three                # -1 0
    je one
    ret
four2seven:                     # 0 1 2 3
    iaddq $-2, %rdx             # -2 -1 0 1
    jl four2five                # 0 1
    jg seven
    jmp six
two2three:                      # 1 2
    iaddq $-1, %rdx
    jg three
    jmp two
four2five:
    iaddq $1, %rdx
    jge five
    jmp four
Loop:
    mrmovq 0x38(%rdi), %rbx
    mrmovq 0x30(%rdi), %r14
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
    andq %rbx, %rbx
    rmmovq %rbx, 0x38(%rsi)
    jle Mv7
    iaddq $1, %rax
Mv7:
    andq %r14, %r14
    rmmovq %r14, 0x30(%rsi)
    jle Mv6
    iaddq $1, %rax
Mv6:
    andq %r13, %r13
    rmmovq %r13, 0x28(%rsi)
    jle Mv5
    iaddq $1, %rax
Mv5:
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle Mv4
    iaddq $1, %rax
Mv4:
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle Mv3
    iaddq $1, %rax
Mv3:
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle Mv2
    iaddq $1, %rax
Mv2:
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle Mv1
    iaddq $1, %rax
Mv1:
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle Mv0
    iaddq $1, %rax
Mv0:
    iaddq $64, %rdi         # src += 8;
    iaddq $64, %rsi         # dst += 8;
    iaddq $-8, %rdx
    jge Loop
    jmp DealRemainder
seven:
    mrmovq 0x30(%rdi), %r14
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    mrmovq (%rdi), %r8
    andq %r14, %r14
    rmmovq %r14, 0x30(%rsi)
    jle M6
    iaddq $1, %rax
six:
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
M6:
    andq %r13, %r13
    rmmovq %r13, 0x28(%rsi)
    jle M5
    iaddq $1, %rax
five:
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
M5:
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle M4
    iaddq $1, %rax
four:
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
M4:
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle M3
    iaddq $1, %rax
three:
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
M3:
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle M2
    iaddq $1, %rax
two:
    mrmovq 0x8(%rdi), %r9
M2:
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle M1
    iaddq $1, %rax
one:
M1:
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle M0
    iaddq $1, %rax
M0:
zero:
```
We got:
```txt
Average CPE     7.67
Score   56.6/60.0
```

Change order:
```
# You can modify this portion
    iaddq $-8, %rdx             # 0 1 2 3 4 5 6 7
    mrmovq (%rdi), %r8
    jge Loop
DealRemainder:
    iaddq $4, %rdx              # -8 -7 -6 -5
    jge four2seven              # -4 -3 -2 -1
    iaddq $3, %rdx              # -4 -3
    jg two2three                # -1 0
    je one
    ret
four2seven:                     # 0 1 2 3
    iaddq $-2, %rdx             # -2 -1 0 1
    jl four2five                # 0 1
    jg seven
    jmp six
two2three:                      # 1 2
    iaddq $-1, %rdx
    mrmovq 0x8(%rdi), %r9
    jle two
    mrmovq 0x10(%rdi), %r10
    jmp three
four2five:
    iaddq $1, %rdx
    jge five
    jmp four
Loop:
    mrmovq 0x38(%rdi), %rbx
    mrmovq 0x30(%rdi), %r14
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    andq %rbx, %rbx
    rmmovq %rbx, 0x38(%rsi)
    jle Mv7
    iaddq $1, %rax
Mv7:
    andq %r14, %r14
    rmmovq %r14, 0x30(%rsi)
    jle Mv6
    iaddq $1, %rax
Mv6:
    andq %r13, %r13
    rmmovq %r13, 0x28(%rsi)
    jle Mv5
    iaddq $1, %rax
Mv5:
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle Mv4
    iaddq $1, %rax
Mv4:
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle Mv3
    iaddq $1, %rax
Mv3:
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle Mv2
    iaddq $1, %rax
Mv2:
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle Mv1
    iaddq $1, %rax
Mv1:
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle Mv0
    iaddq $1, %rax
Mv0:
    iaddq $64, %rdi         # src += 8;
    iaddq $64, %rsi         # dst += 8;
    iaddq $-8, %rdx
    mrmovq (%rdi), %r8
    jge Loop
    jmp DealRemainder
seven:
    mrmovq 0x30(%rdi), %r14
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    andq %r14, %r14
    rmmovq %r14, 0x30(%rsi)
    jle M6
    iaddq $1, %rax
six:
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
M6:
    andq %r13, %r13
    rmmovq %r13, 0x28(%rsi)
    jle M5
    iaddq $1, %rax
five:
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
M5:
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle M4
    iaddq $1, %rax
four:
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
M4:
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle M3
    iaddq $1, %rax
three:
M3:
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle M2
    iaddq $1, %rax
two:
M2:
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle M1
    iaddq $1, %rax
one:
M1:
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle M0
    iaddq $1, %rax
M0:
zero:
```
We got:
```txt
Average CPE     7.60
Score   58.0/60.0
```

Change order and Jump to halt directly.
```
# You can modify this portion
    iaddq $-8, %rdx             # 0 1 2 3 4 5 6 7
    mrmovq (%rdi), %r8
    jge Loop
DealRemainder:
    iaddq $4, %rdx              # -8 -7 -6 -5
    jge four2seven              # -4 -3 -2 -1
    iaddq $3, %rdx              # -4 -3
    jg two2three                # -1 0
    je one
    jmp 0x31
four2seven:                     # 0 1 2 3
    iaddq $-2, %rdx             # -2 -1 0 1
    jl four2five                # 0 1
    jg seven
    jmp six
two2three:                      # 1 2
    iaddq $-1, %rdx
    mrmovq 0x8(%rdi), %r9
    jle two
    mrmovq 0x10(%rdi), %r10
    jmp three
four2five:
    iaddq $1, %rdx
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    jl four
    mrmovq 0x20(%rdi), %r12
    jmp five
Loop:
    mrmovq 0x38(%rdi), %rbx
    mrmovq 0x30(%rdi), %r14
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    andq %rbx, %rbx
    rmmovq %rbx, 0x38(%rsi)
    jle Mv7
    iaddq $1, %rax
Mv7:
    andq %r14, %r14
    rmmovq %r14, 0x30(%rsi)
    jle Mv6
    iaddq $1, %rax
Mv6:
    andq %r13, %r13
    rmmovq %r13, 0x28(%rsi)
    jle Mv5
    iaddq $1, %rax
Mv5:
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle Mv4
    iaddq $1, %rax
Mv4:
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle Mv3
    iaddq $1, %rax
Mv3:
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle Mv2
    iaddq $1, %rax
Mv2:
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle Mv1
    iaddq $1, %rax
Mv1:
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle Mv0
    iaddq $1, %rax
Mv0:
    iaddq $64, %rdi         # src += 8;
    iaddq $64, %rsi         # dst += 8;
    iaddq $-8, %rdx
    mrmovq (%rdi), %r8
    jge Loop
    jmp DealRemainder
seven:
    mrmovq 0x30(%rdi), %r14
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
    andq %r14, %r14
    rmmovq %r14, 0x30(%rsi)
    jle M6
    iaddq $1, %rax
six:
    mrmovq 0x28(%rdi), %r13
    mrmovq 0x20(%rdi), %r12
    mrmovq 0x18(%rdi), %r11
    mrmovq 0x10(%rdi), %r10
    mrmovq 0x8(%rdi), %r9
M6:
    andq %r13, %r13
    rmmovq %r13, 0x28(%rsi)
    jle M5
    iaddq $1, %rax
five:
M5:
    andq %r12, %r12
    rmmovq %r12, 0x20(%rsi)
    jle M4
    iaddq $1, %rax
four:
M4:
    andq %r11, %r11
    rmmovq %r11, 0x18(%rsi)
    jle M3
    iaddq $1, %rax
three:
M3:
    andq %r10, %r10
    rmmovq %r10, 0x10(%rsi)
    jle M2
    iaddq $1, %rax
two:
M2:
    andq %r9, %r9
    rmmovq %r9, 0x8(%rsi)
    jle M1
    iaddq $1, %rax
one:
M1:
    andq %r8, %r8
    rmmovq %r8, (%rsi)
    jle 0x31
    iaddq $1, %rax
zero:

```
We got:
```txt
Average CPE     7.36
Score   60.0/60.0
```
If just changing order, we can got only:
```
Average CPE     7.57
Score   58.6/60.0
```

