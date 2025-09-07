# Hack The Box Write-up: Explosion

- **Machine:** Explosion
- **IP Address:** 10.129.61.85
- **Platform:** Hack The Box
- **Difficulty:** Very Easy (Tier 4)

---

### Summary

"Explosion" is the Tier 4 Starting Point machine on Hack The Box. This Windows-based challenge introduces Remote Desktop Protocol (RDP) enumeration and exploitation. The machine demonstrates the security risks of exposing RDP services with weak or default credentials, a common misconfiguration in Windows environments.

---

### Enumeration

The initial reconnaissance began with a comprehensive Nmap scan to identify all open services and their versions.

**Command:**
`nmap -sV -sC -T4 10.129.61.85`

**Results:**
The scan revealed several open ports, most notably:
- Port 135 (RPC)
- Port 139 (NetBIOS)
- Port 445 (SMB)
- Port 3389 (RDP - Remote Desktop Protocol)

Port 3389 running Microsoft Terminal Services (RDP) became the primary target for this challenge.

---

### Gaining Access (RDP Exploitation)

With RDP exposed, the next step was to attempt authentication with common default credentials.

1. I used the `xfreerdp` tool to connect to the Windows machine:
   `xfreerdp /v:10.129.61.85 /u:Administrator`

2. When prompted for a password, I tried common defaults. The password field was left blank, and surprisingly, the connection was successful with no password required.

3. This granted me immediate access to the Windows desktop as the Administrator user, making privilege escalation unnecessary.

---

### Capturing the Flag

Once logged into the Windows desktop via RDP:

1. I navigated to the Desktop to look for the flag file.

2. Found and opened the `flag.txt` file located on the Administrator's desktop.

3. The file contained the flag needed to complete the challenge.

**Alternative Command Line Approach:**
If needed, the flag could also be retrieved via command prompt:
```cmd
dir C:\Users\Administrator\Desktop\
type C:\Users\Administrator\Desktop\flag.txt
```

---

### Lessons Learned

This machine highlights several critical security issues commonly found in Windows environments:

1. **RDP Security:** Exposing RDP (port 3389) to the internet without proper authentication is extremely dangerous. RDP should always be protected with strong passwords and ideally restricted to VPN access only.

2. **Default Credentials:** Many Windows systems are deployed with default or empty Administrator passwords, creating an immediate security vulnerability.

3. **Network Segmentation:** Critical services like RDP should not be directly accessible from external networks without proper security controls.

4. **Account Lockout Policies:** Implementing account lockout policies can help prevent brute force attacks against RDP services.