## Welcome to Penetration Testing: HTB Archetype

Want to get started with penetration testing, red teaming, and offensive security? Check out resources like [Try Hack Me](https://tryhackme.com/), [Hack The Box](https://www.hackthebox.eu/), or the Heath Adam's [Practical Ethical Hacking course](https://academy.tcm-sec.com/p/practical-ethical-hacking-the-complete-course)! These are all great for beginners like you and I, though some fundamental knowledge should be understood first. Computer networking is key, and concepts like the OSI model will help you to understand how data is moving from point A to point B. Basic Linux usage as well as understanding the basic concepts of Microsoft's Active Directory technology stack will help tremendously. 

This short walkthrough will take you through the first of the Hack The Box (HTB) machines in the *Starting Point* section, **Archetype**. They already do provide clues, but when learning I like to hear the same concepts explained by different people. It helps reinforce what I'm learning and sometimes I just click with one presenter over another. I hope this helps you in the beginning of your own journey!

### First steps

After getting your VPN connection established to the Starting Point environment, the first thing we need to do is enumerate! This means looking around the perimeter to see what windows, doors, and sewer grates might be open for us to get in initially. Let's start with the trusty nmap scan. This will show us open ports, services, and other information like versions, OSes, and sometimes obvious vulnerabilities.

HTB provides an awesome little nmap command to assign all open ports on the 10.10.10.27 Archetype box to a variable called `ports` that they shoot into nmap again as the ports argument. This is a bit complicated to unpack as a beginner, so honestly, you can just use the following simple command:

`nmap -A -T 5 -p- 10.10.10.27`

Check out this resource called [Explain Shell](https://explainshell.com/explain?cmd=nmap%20-A%20-T%205%20-p-%2010.10.10.27) to help with explaining flags without digging through man pages. Basically `-A` does basically what you want in a typical initial scan, `-p-` will scan all ~65k ports, and `-T 5` will speed up the scan since we do not need to worry about detection.

### nmap Results
You should see something similar to this output:
![nmap scan 1](/images/scan01.png)
![nmap scan 2](/images/scan02.png)



We see that it's on Windows Server 2019 and running MS SQL 2017

We see the following ports open: 	
135, 139, 445, 1433, 1434, 3389, 5985, 47001, 49664, 49665, 49666, 49667, 49668, 49669

Of those, 445 and 3389 are the most striking as those are the common ports assigned to the SMB and RDP protocols, both of which are notorious for exploits and misconfigurations we may be able to use.


Since SMB is most commonly associated with Windows network shares, let's see what's being shared with us! ;)
Using Samba's smbclient tool we can list what's available. Note that backslashes are used to escape characters in BASH, so we need to escape the escape character. When listing UNC paths in Windows they are denoted by a double backslash, so each one needs an extra backslash. It does look pretty silly...

`smbclient -N -L \\\\10.10.10.27\\`

![smb 1](/images/smb01.png)

Ooh, what is this backups share that's open to everyone? Backups should not typically be shared to the public, so let's hope there's something juicy in there.
Point smbclient to that share. Then, use `dir` to list out the folder contents since this is a Windows machine. This command is similar to the `ls` command in Unix-land.

![smb 2](/images/smb02.png)

The only file there is `prod.dtsConfig`. Nice. Someone backed up their SQL Server Integration Services configuration to a public share. Looking online we can see that it usually contains ["metadata such as the server name, database names, and other connection properties to configure SSIS packages"](https://fileinfo.com/extension/dtsconfig). Perfect. Use the `get` command to download it and lets inspect it.

![config 1](/images/config01.png)


Cleartext credentials are a beautiful sight for us. We see `User ID=ARCHETYPE\sql_svc` and `Password=M3g4c0rp123`. Those are the credentials for the local sql_svc user on the ARCHETYPE machine. Let's see what we can do from here! 
