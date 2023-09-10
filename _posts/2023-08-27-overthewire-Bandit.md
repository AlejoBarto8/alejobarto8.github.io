---
layout: post
title:  "Over The Wire - Bandit Game"
date:   2023-09-04
desc: "Learn & Practice security concepts - Novice - Over The Wire Plataform"
keywords: "Linux,Bash,Easy"
categories: [LINUX]
tags: [Linux,Training,Easy]
icon: icon-htb
---


**[Over the Wire](https://overthewire.org/wargames/){:target="_blank"}** can help you to learn and practice security concepts in the form of fun-filled games. I'm going to start the path as indicated on the website, from the most basic game and from there go up the complexity.
<br/>

The [Bandit wargame](https://overthewire.org/wargames/bandit/){:target="_blank"} is aimed at absolute beginners. It will teach the basics needed to be able to play other wargames.
This game is organized in levels and I must try to "beat" or "finish" them. Each time I finish a level I get the information on how to start the next one. It's time to start, maybe to people with very advanced skills the first games can be very simple, but they are very beneficial to remember and practice commands, or even create small scripts, for beginners is very interesting this proposal of the **Over The Wire community**.
<br /><br />

<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/><br/>

For games on this platform I must establish a secure connection through the `SSH protocol` to the **Over The Wire** server.
<br/><br/>

> The **Secure Shell Protocol (SSH)** is a cryptographic network protocol for operating network services securely over an unsecured network. Its most notable applications are remote login and command-line execution.

<br />
## Level 0
----------

<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

In the first level, I just need to get a connection to the server through SSH. I have the user and password, **`bandit0:bandit0`**, and the port offered by the service, **2220**.
I start with the **novice level**, `Bandit`, and then scale the degree of difficulty, but the idea is not only to meet the goal of winning each challenge, but to try in various ways to achieve it to acquire as much knowledge as possible, here we go!
<br/><br/>

I investigate with the `man` command a little about **SSH** and I can now establish a valid connection to the server.

```bash
man sh
# -p port
# Port to connect to on the remote host.  This can be specified on a per-host basis in the configuration file
```

```bash
ssh bandit0@bandit.labs.overthewire.org -p2220
```
<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

Since I was able to connect to the server, I can start the game. I will try to publish all the levels in one post, I hope it won't be too long.
To advance to **level 1**, I have to explore in the home directory of the user **`bandit0`** to find the password to connect as **`bandit1`**, I am told it is in a `readme` file.
<br /><br />

<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> Tip: In the case that the shell that I get presents problems with the shortcuts, I modify the TERM environment variable to use an xterm terminal and that's it, I continue working more efficiently.

```bash
export TERM=xterm
```

I connect again to the server and I can browse the directories and get the password for **`bandit1`**. Since I was able to get the information, I can continue with the game and learn new commands and tricks!
<br />

```bash
ssh bandit0@bandit.labs.overthewire.org -p2220
```
<br />

<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

```bash
ssh bandit1@bandit.labs.overthewire.org -p2220
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 1
----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **`cat`**: concatenate files and print on the standard output.

For this level I have to try to open a file whose name is a special character, `-`, I try several ways and in the first ones the command waits for some parameter, so I investigate a little and I manage to get the content of the file.

```bash
cat < -
cat ./-
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

Now that I have the **`bandit2`** user password, I advance to the next level of the Bandit game.

```bash
ssh bandit2@bandit.labs.overthewire.org -p2220
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />


## Level 2
----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

To open a file that has blank spaces, you can escape the space or wrap the file name in quotation marks, and then I can get the text stored in the file.

```bash
cat spac*
cat *this*
cat *name
cat $(pwd)/*
cat "spaces in this filename"
cat spaces\ in\ this\ filename
cat ./spaces\ in\ this\ filename
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">

I can now access the next level with the password and username **`bandit3`**

```bash
ssh bandit3@bandit.labs.overthewire.org -p2220
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />

## Level 3
----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **`ls`**: List information about the FILEs (the current directory by default). Sort entries alphabetically if none of -cftu‐vSUX nor --sort is specified.

> **`find`**: search for files in a directory hierarchy.

The password to advance to the next level is in a `hidden file`, if I first look in the help of the **`ls`** command, I see that there is the **`-a`** parameter to not ignore hidden entries. This way I can now use **`cat`** to open the hidden file.

```bash
ls --help
# -a, --all                  do not ignore entries starting with .

find . -type f -printf "%f\t%p\t%u\t%g\t%m\n" | column -t
find \-type f -name .hidden | xargs cat
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

Since I was able to get what I was looking for, I advance to the next level of the game as the user **`bandit4`**.

```bash
ssh bandit4@bandit.labs.overthewire.org -p2220
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

## Level 4
----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **`file`**: determine file type

To complete this level, I first search with `find` for files in the user's Home folder, and then I can use the `find` command to find out what types of files they are. I can then open the one I am interested in, whose content is humanly readable.

```bash
find . -type f -printf "%f\t%p\t%u\t%g\t%m\n" | grep file0 | column -t
find . -name -file*
find . \-type f -name -file* | xargs file

file inhere/*

find . \-type f -name -file07 | xargs cat
cat inhere/-file07
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

I can now level up with as user **`bandit5`**

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 5
----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **`sed`**: is a stream editor, is used to perform basic text transformations on an input stream (a file or input from a pipeline). While in some ways similar to an editor which permits scripted edits (such as ed), `sed` works by making only one pass over the input(s), and is consequently more efficient. But it is sed's ability to filter text in a pipeline which particularly distinguishes it from other types of editors.

At this level I must search for a file with certain characteristics, if I search with `man` the parameters that the `find` command has, I look for those that can help me to search quickly thanks to them.

```bash
man find
# /readable
# Matches files which are readable by the current user.  This takes into account access control lists and  other
# permissions  artefacts which the -perm test ignores.

# /size
# File uses less than, more than or exactly n units of space, rounding up.

# /executable
# Matches files which are executable and directories which are searchable (in a file name resolution  sense)  by the  current  user.
```

I can now search efficiently and quickly with `find`

```bash
find \-type f -readable
find \-type f -readable ! -executable
find \-type f -readable ! -executable -size 1033c
find \-type f -readable ! -executable -size 1033c | xargs cat
# Ugly output format

find \-type f -readable ! -executable -size 1033c | xargs cat | xargs   # :) MUch Better!
find \-type f -readable ! -executable -size 1033c | xargs cat | sed 's/^ *//'
find \-type f -readable ! -executable -size 1033c | xargs cat | sed '/^\s*$/d'
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

I found the file, and I can know the **`bandit6`** password to be able to connect via ssh.

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 6
----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

The theme of this level is similar to the previous one, I search with `man` the necessary help on `find` parameters to optimize the search according to the information of the characteristics that a particular file has.

```bash
man find
# \owner        :( Nothing Interesting
# \-user uname  File is owned by user uname (numeric user ID allowed).
# \-group gname File belongs to group gname (numeric group ID allowed).

```

Now I can search very quickly in the whole file system, if I want to, and I find what I was looking for. I use `|` to make the output of the `find` command the input of the `cat` command and get the `bandit7` password.

```bash
find \-type f -user bandit7
find \-type f -user bandit7 -group bandit6 2>/dev/null
find \-type f -user bandit7 -group bandit6 -size 33c 2>/dev/null | xargs cat
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

So I can connect to the server as **`bandit7`**. 

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

# Level 7
---------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **`grep`**: searches for PATTERNS in each  FILE.   PATTERNS is   one  or  more  patterns  separated  by  newline characters, and `grep` prints each line that matches a pattern. Typically  PATTERNS should be quoted when `grep` is used in a shell command.

> **`awk`**: Gawk  is the GNU Project's implementation of the AWK programming language.  It conforms to the definition of  the language in the POSIX 1003.1 standard.  This version in turn is based on the description in The AWK Programming Language, by Aho, Kernighan, and Weinberger.  Gawk provides the  additional  features found  in  the  current version of Brian Kernighan's awk and numerous GNU-specific extensions.

At this level I can use the `grep` command, which allows me to filter according to the pattern I indicate, in this case from the text stored in a file, to find a particular word. But I can create different onelines to get what I want.

```bash
cat data.txt | wc -l

cat data.txt | grep millionth
cat data.txt | grep millionth | awk '{print $2}'
cat data.txt | grep millionth -n
cat data.txt | grep millionth -n | awk '{print $2}'
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

Now that I have the **`bandit8`** password, I connect to the server to continue the path. 

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 8
----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **`sort`**: Write sorted concatenation of all FILE(s) to standard output. With no FILE, or when FILE is -, read  standard  input.

> **`unique`**: Filter adjacent matching lines from INPUT (or standard input), writing to OUTPUT (or standard output). With no options, matching lines are merged to the first occurrence.

In order to move forward, I have to find the password of the following user in a text file, with the particularity that it is the only line that is not repeated. With the `sort` and `unique` commands it can be solved quickly.

```bash
sort data.txt
sort data.txt | uniq
sort data.txt | uniq -c
sort data.txt | uniq -c | grep -v 10
sort data.txt | uniq -u
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

Now that I have the password of the user **`bandit9`**, I connect by ssh to the server and continue advancing in level!

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 9
----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **`strings`**: For each file given, GNU strings prints the printable character sequences that are at least 4 characters long (or the number given with the options below) and are followed by an unprintable character.

> **`tail`**: Print the last 10 lines of each FILE to standard output. With more than one FILE, precede each with a header giving the file name.

Now I need to find a particular string in a text file. The `strings` command can help me to get all those understandable strings, the advantage I have is that I also know that I know that is has a certain amount of `=` before, and with `grep` I can filter to get it.

```bash
strings data.txt
strings data.txt | grep ======
strings data.txt | grep ====== | tail -n 1
strings data.txt | grep ====== | tail -n 1 | awk 'NF{print $NF}'
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

Now that I have **`bandit10`** password, I can move forward in my goal of completing this game for rookies!

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 10
-----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **`base64`**: encode or decode FILE, or standard input, to standard output in base 64 format.

> **`tr`**: translate, squeeze, and/or delete characters from standard input, writing to standard output.

Now I have to decode the content of a file that is in **base 64** format, for that I use the `base64` command and I can get the information I need to continue moving forward.

```bash
base64 -d data.txt
base64 -d data.txt | tr ' ' '\n'
base64 -d data.txt | tr ' ' '\n' | tail -n 1
base64 -d data.txt | sed 's/ /\n/g'
base64 -d data.txt | sed 's/ /\n/g' | tail -n 1
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

Now that I have **`bandit11`**'s credentials, I move on to the next challenge!

```bash
ssh bandit11@bandit.labs.overthewire.org -p2220
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 11
-----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **ROT13 (rotate by 13 places):** is a simple letter substitution cipher that replaces a letter with the 13th letter after it in the latin alphabet. ROT13 is a special case of the Caesar cipher which was developed in ancient Rome. 

Now I have to decipher the content of a file that is encrypted in **Rot13**, for that I can use the `tr` command and try to get the plaintext, for this type of encryption is simple because it is old but for the modern ones I should investigate if it could be possible.

```bash
cat data.txt | tr '[A-Za-z]' '[T-ZA-St-za-s]'
cat data.txt | tr '[A-Za-z]' '[C-ZA-Bc-za-b]'
cat data.txt | tr '[A-Za-z]' '[K-ZA-Jk-za-j]'
cat data.txt | tr '[A-Za-z]' '[N-ZA-Mn-za-m]'
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

I can continue advancing, now as the user **`bandit12`**!

```bash
ssh bandit12@bandit.labs.overthewire.org -p2220
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 12
-----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

> **`xxd`**: creates a hex dump of a given file or standard input.  It can also convert a hex dump back to its original binary form.  Like uuencode(1) and uudecode(1) it allows the transmission of binary data in a `mail-safe' ASCII  representation, but has the advantage of decoding to standard output. Moreover, it can be used to perform binary file patching.

> **`gunzip`**: reduces the size of the named files using Lempel-Ziv coding (LZ77). Whenever possible, each file is replaced by one with the extension .gz, while keeping the same ownership modes, access and modification times.

> **`bzip2`**: compresses  files  using  the Burrows-Wheeler block sorting text compression algorithm, and Huffman coding.   Compression is generally considerably better than that achieved by more conventional LZ77/LZ78-based compressors, and approaches the performance of the PPM family of statistical compressors.

> **`tar`**: saves many files together into a single tape or disk archive, and can restore individual files from the archive.

For this level I will follow the recommendation to copy the file to a folder that I create under the user `bandit12` in the **`tmp`** folder. If I use `file` to see the type of file I am messing with, I see that it is clear text but when I open it I see that it is in **hexadecimal** format, which I can reverse it with **`xxd`**. And then I use `file` to see the file type and go using the necessary commands to open/decompress the file until I get the information I need.

```bash
/tmp/oldbtest
cp data.txt !$
cd !$
file data.txt

mv data data.gz
gunzip -d data.gz
mv data data.bz
bzip2 -d data.bz
gunzip -d data.gz
tar -xf data
tar -xf data5.bin
tar -xf data.tar
bzip2 -d data6.bz
tar -xf data6.tar
mv data8.bin data8.gz
gunzip -d data8.gz
cat data8
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

It is obvious that I must create a script to optimize this!! So I can resort to the great **S4vitar's course** on his [hack4u](https://hack4u.io/){:target="_blank"} website and learn how to do it, there are also videos on his [YouTube channel](https://www.youtube.com/@s4vitar){:target="_blank"}.

With everything I learned I can create an efficient and fast script.

```bash
 xxd -r data.txt > data
file data
mv data data.gzip
```

> **`getpassword.sh`** script

```bash
#!/bin/bash

name_compress_file=$(7z l data.gzip | grep 'Name' -A 2 | tail -n 1 | awk 'NF{print $NF}')
7z x data.gzip > /dev/null 2>&1

while true; do
  7z l $name_compress_file > /dev/null 2>&1

  if [ "$(echo $?)" == "0" ]; then
    name_compress_file_two=$(7z l $name_compress_file | grep 'Name' -A 2 | tail -n 1 | awk 'NF{print $NF}')
    7z x $name_compress_file > /dev/null 2>&1 && name_compress_file=$name_compress_file_two
  else
    cat $name_compress_file; rm data* 2>/dev/null
    exit 1
  fi
done

```

Now I can move on, I already have the password of the user **`bandit13`**.

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 13
-----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

To continue advancing level, now we do not have a file where to look for the password, directly we share a private `id_rsa` key with which we can connect to the local machine as the user `bandit14`.

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

We can do it from the shell that we already have or from the shell of our host machine.

```bash
ssh -i sshkey.private bandit14@localhost
```

```bash
ssh -i id_rsa bandit14@bandit.labs.overthewire.org -p2220
```

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

I can now go to the next level as user **`bandit14`**!

## Level 14
-----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

Now I have to send the password string to the port of the victim machine on port 30000, I can do it with `nc` or `telnet` and get the password of the user **`bandit15`**.

```bash
telnet localhost 30000
nc localhost 30000
echo -------- | nc localhost 30000
```
<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

I can now access the remote server as the user **`bandit15`**!

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

## Level 15
-----------

<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />


```bash
```
<br />
<img src="{{ site.img_path }}/overthewirebandit/OTH_bandit_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br />

I can now access the remote server as the user **`bandit16`**!


## Resources
------------

[Secure Shell (SSH) - Wikipedia](https://en.wikipedia.org/wiki/Secure_Shell){:target="_blank"}

[How to use SSH on wikiHow](https://www.wikihow.com/Use-SSH){:target="_blank"}

[Dashed Filename](https://www.webservertalk.com/dashed-filename){:target="_blank"}

[Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/special-chars.html){:target="_blank"}
