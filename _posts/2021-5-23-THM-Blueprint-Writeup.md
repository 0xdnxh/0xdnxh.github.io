---
layout: post
title: THM Blueprint writeup
categories: THM
---
## Introduction
<img src="/images/THM/Blueprint/banner.PNG" width="800" height="150"/>

In this blog post you will find a writeup for the [Blueprint](https://tryhackme.com/room/blueprint) room on TryHackMe. It's a Windows machine running a vulnerable version of osCommerce, an online store solution. Since this room isn't guided like the other ones, you will find my own steps.

## Enumeration
First we fire up nmap to get more info on the machine and find out open ports. The command that I used is ```nmap -sV -sC -Pn 10.10.7.76``` , the option -sV is used to determine service/version info, -sC to use the default script of nmap and -Pn to disable host discovery. The command outputs the following:

<img src="/images/THM/Blueprint/nmap_output.PNG" width="800" height="800"/>

From this output we understand that the following interesting ports are open:
<ul>
  <li>80</li>
  <li>8080</li>
  <li>443</li>
</ul>

Navigating to http://10.10.7.76 , we get a Server error:

<img src="/images/THM/Blueprint/server_error.PNG" width="600" height="100"/>

However, navigating to https://10.10.7.76 or 10.10.7.76:8080 we get the following page:

<img src="/images/THM/Blueprint/oscommerce_page.PNG" width="500" height="150"/>

Interesting! So now we know there's an **osCommerce version 2.3.4** application running. Navigating inside the directory, we find two folders: catalog/ and docs/

<img src="/images/THM/Blueprint/indexof_oscommerce.PNG" width="500" height="175"/>

### Fuzzing
Let's try to find which directories exist using dirb

<img src="/images/THM/Blueprint/dirboutput.PNG" width="500" height="250"/>

Nothing really interesting. Let' go back to focusing on finding an exploit related to the version.

## Exploitation
Trying to find out if there are any exploit for osCommerce 2.3.4 using searchsploit, we come across some results that could potentially be helpful :

<img src="/images/THM/Blueprint/searchsploit_oscommerce.PNG" width="600" height="200"/>

The last one **php/webapps/44374.py** looks interesting, it's a Remote code execution, we can find the complete code [here](https://www.exploit-db.com/exploits/44374). Let's download it and take a look at it. 

<img src="/images/THM/Blueprint/exploit_original.PNG" width="700" height="400"/>

It looks like we could modify some PHP code in the **http://ipaddress/oscommerce-2.3.4.1/catalog/install/install.php** page and execute it. In the code, the payload tries to execute the **system("ls")** command to list the files on a Linux target. Let's try to replace it with a Windows PHP reverse shell. I used [this one](https://raw.githubusercontent.com/Dhayalanb/windows-php-reverse-shell/master/Reverse%20Shell.php) and modified it accordingly so now it looks like this :

<img src="/images/THM/Blueprint/exploit_modified.PNG" width="700" height="500"/>

Let's fire up Metasploit and use **/multi/handler** to set up a listener on port 1234, accordingly to my modified script.

<img src="/images/THM/Blueprint/multihandler.PNG" width="600" height="200"/>

Let's run the exploit :

<img src="/images/THM/Blueprint/launching_script.PNG" width="600" height="100"/>

We load **configure.php** as indicated in the output and TADAAA we receive a shell on our listener :)

<img src="/images/THM/Blueprint/metasploit_shell.PNG" width="600" height="90"/>

A shell is good but a Meterpreter session is better as it will allow us to run tools. We use the command ```sessions -u 7``` where 7 is our session ID :

<img src="/images/THM/Blueprint/upgrade.PNG" width="600" height="300"/>

## Cracking NTLM hash

Great ! So now we have a Meterpreter sessions we can interact with. The question in the THM room tells us to get the NTLM hash for user "Lab". Using hashdump we are able to retrieve hashes on the system :

<img src="/images/THM/Blueprint/hashdump.PNG" width="600" height="150"/>

The NTLM hash is in the following format : ```Username:SID:LMhash:NThash```. To decrypt it, I use Hashcat which is a password cracking tool. The command I use is ```hashcat -m 1000 myhash SecLists/Passwords/Common-Credentials/common-passwords-win.txt``` where:
<ul>
  <li>-m 1000 : specifies the type of hash to crack, 1000 is for NTLM. You can find the complete list for hash types [here](https://hashcat.net/wiki/doku.php?id=example_hashes)</li>
  <li>myhash : the file where I stored the necessary part of the NTLM hash for user Lab, which is the last part. It looks like this :
  </li>
  <img src="/images/THM/Blueprint/cat_myhash.PNG" width="300" height="90"/>
  <li>SecLists/Passwords/Common-Credentials/common-passwords-win.txt : the wordlist containing commong Windows passwords to compare to. It can be found [here](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/common-passwords-win.txt)</li>
</ul>

Unfortunately, this doesn't work. I decide instead to go for something easier : [CrackStation](https://crackstation.net/), where you will just provide the hash and the website cracks it for you. Convenient, no ? It uses a dictionary that can be found [here](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm).
Okay, so we got the cracked password that I will not be sharing here for obvious reasons. That will be the answer to the question in the THM room. 

## Finding the root flag

Now let's go back to our Meterpreter session and find the **root.txt** flag. We can use the command ```dir /s *root*``` in the C:\Users\Administrator folder, it returns the exact location for our file :

<img src="/images/THM/Blueprint/searchflag.PNG" width="450" height="250"/>

And we got our root.txt :

<img src="/images/THM/Blueprint/flag.PNG" width="500" height="90"/>