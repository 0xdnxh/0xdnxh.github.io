---
layout: post
title: THM Bounty Hacker writeup
categories: THM
---
## Introduction
<img src="/images/THM/BountyHacker/banner.PNG" width="800" height="90"/>

In this blog post you will find a writeup for the [Bounty Hacker](https://tryhackme.com/room/cowboyhacker) room on TryHackMe. It's a vulnerable Linux machine and since this room has a few questions to answer, you will find my answers as well as some additional information I gathered along the way.

## Enumeration
**IP**: 10.10.37.218  
First of all, let's run nmap to get more information about the machine and discover open ports. The command that I used is ```nmap -sC -sV 10.10.37.218``` , the option -sV is used to determine service/version info, -sC to use the default script of nmap and -Pn to disable host discovery. The command outputs the following: 

<img src="/images/THM/BountyHacker/nmap_output.PNG" width="600" height="600"/>

From this output we understand that the following ports are open:
<ul>
  <li>21 : running FTP, we notice that Anonymous FTP login is allowed</li>
  <li>22 : SSH </li>
  <li>80</li>
</ul>
We also see that the OS is Linux. 

## FTP anonymous login
Let's try to log in FTP using anonymous:anonymous as credentials and look around by listing system files :

<img src="/images/THM/BountyHacker/ftploginls.PNG" width="400" height="250"/>

We can see that there are two files **locks.txt** and **task.txt**, let's go ahead and download them :

<img src="/images/THM/BountyHacker/tasklocks_download.PNG" width="400" height="220"/>

Task.txt looks like it was written by a person named **lin** who might be a user on the system:

<img src="/images/THM/BountyHacker/tasktxt.PNG" width="300" height="150"/>

However, Locks.txt looks like it's a list of passwords :

<img src="/images/THM/BountyHacker/lockstxt.PNG" width="200" height="250"/>

So maybe we could use these passwords to bruteforce SSH using the username **lin** .

## SSH Login/Getting user shell

For this part, I'm using Hydra which is a login cracker that supports many protocols to attack. The command I use is ```hydra -l lin -P locks.txt 10.10.37.218 -t 4 ssh``` where :
<ul>
  <li>-l : specifies the username to use, in our case lin</li>
  <li>-P : specifies the wordlist of passwords to use </li>
  <li>-t : number of threads</li>
  <li>ssh : the protocol we are trying to bruteforce</li>
</ul>
You can find more on using Hydra to bruteforce the SSH login [here](https://linuxconfig.org/ssh-password-testing-with-hydra-on-kali-linux) .

Running the command gives us the following output :

<img src="/images/THM/BountyHacker/hydraoutput.PNG" width="800" height="230"/>

Great, so now we have valid credentials to connect to SSH ! Using them enables us to log into the system and list system files and we can find the **user.txt** file :

<img src="/images/THM/BountyHacker/sshconnect.PNG" width="500" height="350"/>

## Privilege escalation
Let's see which commands our user lin is allowed to run on the machine using ```sudo -l```

<img src="/images/THM/BountyHacker/sudoloutput.PNG" width="550" height="100"/>

We see that we're allowed to run /bin/tar as root. This could be a way for us to escalate privileges and pop that root shell. One great resource for privesc is [GTFOBins](https://gtfobins.github.io/). Looking at the page dedicated to **tar**: 

<img src="/images/THM/BountyHacker/tar.PNG" width="550" height="90"/>

Great ! So we can run that command **sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh** and get a root shell :

<img src="/images/THM/BountyHacker/root.PNG" width="550" height="90"/>

And we got our root.txt :

<img src="/images/THM/BountyHacker/rootflag.PNG" width="180" height="140"/>

