# HTB Writeups — OSCP Prep Journey

This repository contains writeups and penetration testing reports for HackTheBox Starting Point machines, created as part of my OSCP preparation.

## Boxes Completed

| # | Machine | OS | Difficulty | Topics | Writeup | Medium |
|---|---------|-----|------------|--------|---------|--------|
| 1 | Archetype | Windows | Beginner | SMB, MSSQL, xp_cmdshell, WinPEAS | [archetype.md](./archetype.md) | [Medium](https://medium.com/@soulhex777/htb-starting-point-archetype-walkthrough-oscp-prep-43fec17700f6) |
| 2 | Oopsie | Linux | Beginner | Web Enumeration, Cookie Manipulation, File Upload, SUID PATH Hijack | [oopsie.md](./oopsie.md) | [Medium](https://medium.com/@soulhex777/htb-starting-point-oopsie-walkthrough-oscp-prep-b95c1f838745) |
| 3 | Vaccine | Linux | Beginner | FTP Anonymous Login, SQL Injection, ZIP Cracking, PostgreSQL RCE, vi GTFOBin | [vaccine.md](./vaccine.md) | [Medium](https://medium.com/@soulhex777/htb-starting-point-vaccine-walkthrough-oscp-prep-ce3e1c6fd519) |

## Methodology

Each writeup follows an OSCP-style reporting format:

1. **Reconnaissance** — Nmap scans, service enumeration
2. **Initial Access** — Exploitation of identified vulnerabilities
3. **Post-Exploitation** — Credential hunting, lateral movement
4. **Privilege Escalation** — Local enumeration, exploit chain
5. **Proof** — whoami / hostname / flag screenshots
6. **Findings** — CVSSv3-rated findings with remediation

## Tools Used

- Nmap, Gobuster, Burp Suite
- sqlmap, zip2john, John the Ripper
- Netcat, Python3 pty shell
- WinPEAS, LinPEAS
- GTFOBins

## About

BTech graduate | Cybersecurity enthusiast | OSCP candidate

Documenting every machine to build a solid pentest methodology.
