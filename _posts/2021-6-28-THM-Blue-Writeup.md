---
layout: post
title: THM Blue writeup
categories: THM
---
## Introduction
<img src="/images/THM/Blue/banner.PNG" width="800" height="150"/> 

In this blog post you will find a writeup for the [Blue](https://tryhackme.com/room/blue) room on TryHackMe. This is a Windows machine running a vulnerable version of SMB, a network file sharing protocol. 

## Enumeration
**IP** : 10.10.53.170  
First we fire up nmap to get more info on the machine and find out open ports. The command that I used is ```nmap -sV -sC -Pn -p- 10.10.53.170``` , the option -sV is used to determine service/version info, -sC to use the default script of nmap and -Pn to disable host discovery. I also included ```-p-``` to make sure to scan all the ports. The command outputs the following:

<img src="/images/THM/Blue/nmap_output.PNG" width="800" height="800"/>

From this output we understand that the following interesting ports are open:
<ul>
  <li>135</li>
  <li>139</li>
  <li>445</li>
</ul>

This indicates that the machine is exposing its SMB service which is known for having multiple vulnerabilities in the past. We can already see that the Hostname is **JON-PC** but let's use the nmap script **smb-os-discovery** to get more information about the OS and the SMB service:
<img src="/images/THM/Blue/smbosdiscovery.PNG" width="800" height="800"/>

Nice, we got more information about the OS version and we confirmed the computer and workgroup names. Now let's use **smb-protocols** to get more information about the version:
<img src="/images/THM/Blue/smbprotocols.PNG" width="800" height="800"/>

Let's also use the **smb-enum-shares** nmap script to list the shares and see if we can have anonymous access. Unfortunately w can't, so it's obvious that if we want to connect to any of the present shares we're going to need valid credentials.
<img src="/images/THM/Blue/smbenumshares.PNG" width="800" height="800"/> 

Okay so now using the information we gathered let's see if we can find a vulnerability for SMBv1 on Windows. Doing some research on internet, we find out that this SMB version is vulnerable to EternalBlue. 

## EternalBlue

What is EternalBlue? EternalBlue is the name given both to the vulnerability and to the exploit that was developed for it. Originally, the NSA discovered the bug in the protocol and once they found it, they developed EternalBlue to exploit the vulnerability. However, in 2017, the exploit got leaked by the Shadow Brokers hacker group and was released into the wild. 
<img src="/images/THM/Blue/EternalBlueschema.PNG" width="800" height="800"/> 


The [vulnerability](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144) itself allows remote attackers to execute arbitrary code on a system by sending specially crafted messages to the SMBv1 server. It was patched and listed on Microsoft’s security bulletin as [MS17-010](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010).

What's more interesting is that, if one device is infected by malware via EternalBlue, this puts other devices that are connected to the same network is at risk. This is what happened with [WannaCry](https://www.csoonline.com/article/3227906/what-is-wannacry-ransomware-how-does-it-infect-and-who-was-responsible.html), a crypto-ransomware, whicwas one of the first and most well-known malware to use this exploit to spread.

### Gaining access
Now that we gathered some information about the vulnerability, let's try to exploit it. Let's start Metasploit and search for MS17-010:

<img src="/images/THM/Blue/metasploitms17010.PNG" width="800" height="800"/>

The exploit that stands out for our Windows 7 target is the **EternalBlue SMB Remote Windows Kernel Pool Corruption**, which is used by the module **exploit/windows/smb/ms17_010_eternalblue** so let's go ahead and use that one.

<img src="/images/THM/Blue/metasploitoptions.PNG" width="800" height="800"/>

### Shell


### Meterpreter session
Disclaimer: In this part I don't follow the room questions, I'm directly trying to get a Meterpreter session.

We use the payload **windows/x64/meterpreter/reverse_tcp** and after setting the RHOSTS and LHOST options using the corresponding IP addresses, we run the exploit. As we can see, we get a Meterpreter session:
<img src="/images/THM/Blue/runexploit.PNG" width="800" height="800"/>

We can verify that we are on the machine using **sysinfo** and that our UID is **NT AUTHORITY\SYSTEM**
<img src="/images/THM/Blue/sysinfogetuid.PNG" width="800" height="800"/>

We can now use **ps** to list all the processes:
<img src="/images/THM/Blue/pscommand.PNG" width="800" height="800"/>>

## Cracking NTLM hash

using hashdump we are able to retrieve hashes on the system :
<img src="/images/THM/Blue/hashdump.PNG" width="800" height="800"/>

As we can see, the NTLM hash corresponding to our user Jon is there. The NTLM hash is in the following format : ```Username:SID:LMhash:NThash```. To decrypt it, I ran Hashcat which is a password cracking tool using the command ```hashcat -m 1000 jonhash SecLists/Passwords/Common-Credentials/common-passwords-win.txt``` You can find the details of the options in my [Blueprint writeup](https://0xdnxh.github.io/THM-Blueprint-Writeup/) in the "Cracking NTLM hash" section.
After a few seconds, Hashcat retrives the password in cleartext:
<img src="/images/THM/Blue/hashcat.PNG" width="800" height="800"/>

### Finding flags

The last part of this room invites us to find 3 flags planted on the machine using some hints :)

## First flag

*Hint : This flag can be found at the system root.*

## Second flag

*Hint : This flag can be found at the location where passwords are stored within Windows.*

## Third flag

*Hint : This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.*





