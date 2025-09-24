# Hack The Box Write-up: Dancing

- **Machine:** Dancing
- **IP Address:** 10.129.16.169
- **Platform:** Hack The Box
- **Difficulty:** Very Easy (Tier 2)

---

### Summary

"Dancing" is the Tier 2 Starting Point machine on Hack The Box. This challenge is a fantastic introduction to the enumeration of a new, critical service: SMB (Server Message Block). The path to completion involves listing public shares, connecting to one anonymously, and retrieving the flag.

---

### Enumeration

The reconnaissance phase began with a comprehensive port scan to identify open services and their configurations.

**Command:**
`nmap -sV -sC -T4 10.129.16.169`

**Results:**
The Nmap scan revealed that port 445 was open, running the `microsoft-ds` service, which indicates SMB (Server Message Block). SMB is a network protocol used for sharing files, printers, and other resources between devices on a network.

**Key Findings:**
- Port 445: SMB service running
- Windows-based target system indicated
- Potential for anonymous share access

**Additional SMB Enumeration:**
I also performed additional SMB-specific enumeration:
```bash
# Check for SMB version and signing requirements
nmap --script smb-protocols -p 445 10.129.16.169
nmap --script smb-security-mode -p 445 10.129.16.169
```

---

### Gaining Access (SMB Enumeration)

With SMB identified as the target service, I proceeded to enumerate available shares and test for anonymous access.

1. **Share Enumeration:**
   I listed the available shares on the target without authentication:
   `smbclient -L 10.129.16.169`
   
   Alternative enumeration methods:
   ```bash
   # Using smbclient with null session
   smbclient -L //10.129.16.169 -N
   
   # Using enum4linux for comprehensive enumeration
   enum4linux 10.129.16.169
   ```

2. **Share Discovery:**
   The output revealed several shares, including:
   - Administrative shares (C$, ADMIN$, IPC$)
   - A custom share named 'Workshares' that appeared accessible

3. **Anonymous Access Testing:**
   I attempted to connect to the 'Workshares' share without credentials:
   `smbclient //10.129.16.169/Workshares`
   
   The connection was successful, indicating misconfigured share permissions that allow anonymous access.

4. **Share Navigation:**
   Once connected, I was presented with an SMB prompt:
   ```
   smb: \>
   ```
   
   I then explored the available commands and directory structure:
   ```bash
   help                    # List available commands
   ls                      # List files and directories
   pwd                     # Show current directory
   ```

---

### Capturing the Flag

After gaining anonymous access to the SMB share, I proceeded to locate and retrieve the flag:

1. **Directory Exploration:**
   I systematically explored the share contents:
   ```bash
   smb: \> ls
   smb: \> cd [directory_name]
   smb: \> ls -la
   ```

2. **Flag Discovery:**
   Through directory traversal, I located a file that contained the flag.

3. **File Download:**
   I downloaded the flag file to my local machine:
   ```bash
   smb: \> get flag.txt
   smb: \> quit
   ```

4. **Local Verification:**
   After exiting the SMB session, I verified the flag content:
   ```bash
   cat flag.txt
   ```

   This revealed the flag needed to complete the challenge.

---

### Lessons Learned

This machine demonstrates critical SMB security principles:

1. **Share Permissions:** SMB shares should never allow anonymous access. Proper authentication and authorization controls must be implemented.

2. **Default Configurations:** Many SMB implementations have insecure default configurations that should be hardened before deployment.

3. **Network Segmentation:** SMB services should be restricted to internal networks and not exposed to the internet.

4. **Access Controls:** Implement the principle of least privilege for all network shares, ensuring users only have access to resources they need.

5. **Regular Auditing:** Regularly audit SMB share permissions and access logs to detect unauthorized access attempts.

6. **SMB Signing:** Enable SMB signing to prevent man-in-the-middle attacks and ensure data integrity.

7. **Alternative Protocols:** Consider more secure alternatives like SFTP or HTTPS-based file sharing for sensitive data transfer.
