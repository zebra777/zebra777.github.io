---
layout: page
title: Cheat Sheet
type: cheatsheet
---   

<center><em>75 6e 64 65 72 20 63 6f 6e 73 74 72 75 63 74 69 6f 6e 20 2e 2e 2e</em></center> 


---  
   


## Port and service scanning  
---
  
Rustscan <a href="https://github.com/RustScan/RustScan" target="_blank">&#x1f517; Link</a>  
`rustscan ip-address --ulimit 5000 -- -sC -sV -oN nmap/initial | tee r./nmap/rust_initial.txt`
      
`rustscan ip-address --ulimit 5000 -- -A nmap/initial | tee ./nmap/rust_-A.txt`    

Nmap  
`nmap -A -T4 ip-address `
    
`nmap -Pn -p- ip-address -oN .nmap/portsonly`   
---> `cat ./nmap/ports|grep open | awk -F/ '{print $1}' ORS=','`
    
`nmap -sC -sV -oN .nmap/ports`  



## Web Enumeration
---
  

dirserach<a href="https://github.com/maurosoria/dirsearch" target="_blank">&#x1f517; Link</a>        
`python3 /git/dirsearch/dirsearch.py -u ip-address -e html,php`    
  
Gobuster
`gobuster dir -u http://ip-address -w wordlists -x extentions -o output.txt`  

wfuzz (Subdomain fuzzering)
`wfuzz -c -f sub-fighter -w wordlist -u "http://hostname" -H "Host: FUZZ.hostname" --hc 404 --hw 968`  
(--hc, --hw = exclude commands; for the first scan not needed)
  


# Image Enumeration
--- 


Stegcracker<a href="https://github.com/Paradoxis/StegCracker" target="_blank">&#x1f517; Link</a>    
`stegcracker image wordlist`
  



# Listener
---
  

netcat listener
nc -lnvp port

Pwncat<a href="https://github.com/calebstewart/pwncat" target="_blank">&#x1f517; Link</a>     
  
`pwncat --config /git/pwncat/data/pwncatrc -l -p 9901`    
  
  


# Privilege escalation
---
  

<ins>Password Hunting</ins>  
Looking for Word Password in Files on system:
    
`grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null`
    
`grep --color=auto -rnw '/' -ie "PASSWORD=" --color=always 2> /dev/null`
    
`grep --color=auto -rnw '/' -ie "PASS" --color=always 2> /dev/null`
    
  
`locate password | more`
  
`locate pass | more`
  
`locate pwd | more`
  
  
<ins>SSH Keys</ins>  

`find / -name authorized_keys 2> /dev/null`  
  
`find / -name id_rsa 2> /dev/null`
  
<ins>User enumeration</ins> 
  
Hostname    
`whoami` ; `id`
  
Sudo Run Check    
`sudo -l`
  
User-list  
`cat /etc/passwd` ; `cat /etc/passwd | cur -d : -f 1`
  
Group list    
`cat /etc/group`

Switch user without password  
`sudo su -`

History check  
`history`





  
---