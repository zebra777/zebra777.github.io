---
layout: post
title:  "EasyPeasy on Tryhackme"
date:   2020-10-21 13:12:00 +0200
categories: writeup thm
permalink: "/:categories"
---
![alt text](/images/easypeasy.png)

**[THM EasyPeasy Link][Easypeasylink]**

# EasyPeasy WriteUp
## by Zebra

>
*Platorform: Tryhackme   
*Difficulty: Easy  
*OS		  :  Linux

------------------------ 
# ***Under Construction!!!***



### <ins>Preparation Steps</ins>

> 
```bash	
mkdir EasyPeasy #New Room folder
cd EasyPeasy   	#Move into folder
mkdir nmap  	#Nmap directory
mkdir gobuster  #Gobuser directory
```

### <ins>Scanning and Enumeration</ins>

*#Portscanning by Rustscan*

>
```bash
rustscan -b 500 -t 1500 10.10.144.140 -- -A | tee ./nmap/rust_initial.txt
```

*#Open Ports*  
>
80/tcp,  	#Webserver  
6498/tcp,   #ssh  
65542/tcp,  #Webserver2  

### <ins></ins>

[Easypeasylink]: https://tryhackme.com/room/easypeasyctf
