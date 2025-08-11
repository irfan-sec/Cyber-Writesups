# Hack The Box Write-up: Archetype

- **Machine:** Archetype
- **IP Address:** 10.129.6.102
- **Platform:** Hack The Box
- **Difficulty:** Easy (Starting Point - Tier 2)

---

## Summary

"Archetype" is a Tier 2 machine that demonstrates a classic **vulnerability chaining** attack. The path to compromise involves enumerating an anonymous SMB share to discover database credentials, which are then used to perform a SQL Injection (SQLi) attack to bypass a web application's login page.

---

## Enumeration

Initial reconnaissance started with our standard Nmap scan to identify open ports and services.

**Command:**
`nmap -sV -sC -T4 -Pn 10.129.6.102`

**Results:**
The Nmap scan revealed several key open ports, including port 1433 (Microsoft SQL Server) and port 445 (SMB), indicating a Windows target.

---

## Gaining Access (SMB -> SQLi Chain)

The attack was a two-stage process, chaining two vulnerabilities together.

### Step 1: SMB Enumeration
An open SMB port was the first point of interest. I listed the available shares using `smbclient` with a blank password.
`smbclient -L //10.129.6.102/ -N`

This revealed a share named 'backups'. I connected to it anonymously:
`smbclient //10.129.6.102/backups -N`

Inside this share, I found a configuration file that contained the password for the 'archetype' user for the SQL database.

### Step 2: SQL Injection (SQLi)
The target machine was also hosting a web server with a login page. Using the password I discovered, I confirmed the login page was vulnerable to SQL Injection.

A classic SQLi payload in the password field, such as `' OR 1=1 -- `, successfully bypassed the authentication and granted me access to the application.

---

## Privilege Escalation & Flag Capture

Once logged into the application, I was able to find the user flag. Further enumeration of the system led to discovering a path to escalate privileges and capture the root flag.

---

## Lessons Learned

This machine is a perfect example of **defense-in-depth**. A single misconfiguration (the anonymous SMB share) can leak credentials that allow a completely different service (the web application) to be compromised. All services must be secured, even those not directly facing the internet.
