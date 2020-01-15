---
title: Binary Exploitation - Modern Stack Buffer Overflow Countermeasures
date: 2020-01-16 10:22:07
tags: Security
---

# Address Space Layout Randomization (ALSR)
ALSR is the process by which the addresses for particular sections of a program (the stack in the Linux kernel) are randomized to prevent exploits from targeting a particular address. This can be demonstrated using a simple program.
```c
#include <stdio.h>

int main(int argc, char* argv)
{
        int x = 0;
        printf("%x\n",(unsigned int)&x);
        return 0;
}
```

Running this a couple of times will yield the following result:
```bash
ffe61c68
ffb43b28
ffa70a08
ffe70398
ffda4b18
ffc00538
ff9a6478
ffdfaf88
ffda0258
ffb25508
```
We can see that the results each have 5 random bytes within them that would need to be known if attempting to smash the stack.

# Position Independent Executable (PIE)
PIE/PIC (Position Independent Code) is used to describe a program that can be run from anywhere in memory regardless of its absolute address. This differs from absolute code, in that position-independent code can be executed at any memory address without modifications. This is usually for loading in shared libraries at different addresses in programs virtual memory spaces.

The primary purpose of PIE is not for security reasons, it acts as a way to prevent exploits from knowing the exact address of specific instructions when used with ASLR. We can show this using the following program:

```c
#include <stdio.h>

int main(int argc, char* argv)
{
        printf("%x\n",(unsigned int)&printf);
        return 0;
}
```
Gives us the output:
```
f7e052d0
f7df62d0
f7df82d0
f7dee2d0
f7deb2d0
f7dd52d0
f7e1e2d0
f7d4b2d0
f7de22d0
f7dec2d0
```
This makes attacks using `ret-to-lib` methods extremely difficult.

# Stack Variable Reordering
If we look at following code:
```c
#include <stdio.h>

int main(int argc, char* argv)
{
        int y[5];
        int x = 3;
        y[0] = 10;
        return 0;
}
```
We can see that if we overflow `y`, we can be able to overwrite `x`. This would be true in older systems, however if we inspect the ASM for newer systems.
```
  0x000005ad <+48>:	mov    DWORD PTR [ebp-0x24],0x3
  0x000005b4 <+55>:	mov    DWORD PTR [ebp-0x20],0xa
```

We find that arrays and buffers are deemed to be high risk data structures. Therefore, variables will be ordered such that the high risk structures are placed towards the top of the stack frame and the lower risk structures are placed towards the bottom, this means that any buffer overflow will not overwrite lower risk variables.

# Stack Canary
A stack canary is a random value that is stored in the stack at the beginning of the function and is checked for integrity at the end. If the stack canary has been overwritten, the program will throw stack check error and output to the console that stack smashing has been detected.

We can see the value being stored here:
```
   0x565555a1 <+36>:	mov    ecx,DWORD PTR gs:0x14
   0x565555a8 <+43>:	mov    DWORD PTR [ebp-0xc],ecx
```
The stack canary is extracted from `gs:0x14` and stored into `[ebp-0xc]`.
We then check that this value doesn't change at the end of the function.
```
   0x565555df <+98>:	mov    ebx,DWORD PTR [ebp-0xc]
   0x565555e2 <+101>:	xor    ebx,DWORD PTR gs:0x14
   0x565555e9 <+108>:	je     0x565555f0 <main+115>
   0x565555eb <+110>:	call   0x56555670 <__stack_chk_fail_local>
```
We compare the value from the stack and the original value from `gs:0x10`  using the `xor` instruction and then we will either jump to `__stack_chk_fail_local` or continue, depending on whether the value matches or not.

Some example canaries:
```
927fb500
19e92b00
19c5ee00
1e301100
564b7500
41443700
b546f100
9622da00
```

Notice that for a 32-bit device, the canary has only 24 random bits, this means that there are only ~16 million unique combinations. The last bit is a null byte to acts as a string terminator to prevent exploits using `strmp` from propagating through the canary.