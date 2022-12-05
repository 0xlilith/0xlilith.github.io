---
author: "lilith"
title: "HeapOverride Senpai's Castle"
date: "2021-08-07"
description: "Lilith Struggling with heap senpai's binary"
tags:
  - writeup
  - rev
---
![](images/img1.png)

hmmm smol pp moment ! now we go open it in a disassembler

![](images/img2.png)

lets read the main function!
first it does its regular job of loading the stack and moving argc and argv to the registers 
then it checks weither the value of argc is == 2 as it takes one 1 argument vector shown in img1.
If it fails the condition then it jumps to the address 0x11bd. less have a look at it.

![](images/img3.png)

brrr....as we can see it just prints out the string "Umm...no..."
now lets see what happens if it matches the condition i.e if we pass something in the argument vector.

![](images/img4.png)

okkkkkkkk wait i am feeling cold... i am back. Back to the whatever we are doing. We can see that the string 
that we entered is being passed to a function call fun.00011dc. Now lets see what that function does to our 
baby stringwqnwdn. uhhghhghg my string ;-;-;-;!!

![](images/img5.png)

brrrr this shit is too confusing lets decompile it else ill die trying
you dont want me to die right?....right? do you want me to die do you want me to DIE???
ok

![](images/img6.png)

much better but we still need to do some cleanups...wait here
I just changed the variable names to something readable brrr wtf is uVar3 and var4h._4_4

![](images/img7.png)

before we do anything else lets first see what that 0x4050 holds for us. 
shshbh "FZGSVdzohWNAAGSCW^z_HMf]THbJWUnT_URK" som crazy string.
They are first comparing the length of that crazy string to the length of our baby string. 
And if the condition matches we enter inside the loop.

lets make it moe simple [note: asuming crazy = 0x4050]

```c
13  j = crazy_string_len;
14  for(i = 0; i < crazy_string_len; i++){
15    if (crazy[i] ^ j ^ 55 ) != 
16      arg1[i]){ // this next line is on purpose cus i am trying to replicate the code line by line
17      return 0;
18    }
19    j--;
20  }
```

j stores the length of that crazy string. Then inside the for loop we have an if with some condition.
char at index i of crazy xor j xor 55 is != char at index i of our baby string.
waaaaaaw now we know whats actually happening inside the hood. We can just write a script to solve it

![](images/img8.png)

hmmm BIG pp moment !! we got the flag