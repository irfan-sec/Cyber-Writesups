# Hack The Box Write-up: Fawn

- **Machine:** Fawn
- **IP Address:** 10.129.14.1
- **Platform:** Hack The Box
- **Difficulty:** Very Easy

---

### Summary

"Fawn" is a "Very Easy" Starting Point machine on Hack The Box that introduces File Transfer Protocol (FTP) enumeration and exploitation. This challenge demonstrates the security risks associated with anonymous FTP access, which is a common misconfiguration that can lead to sensitive data exposure.

---

### Enumeration

The reconnaissance phase began with a comprehensive port scan to identify open services.

**Command:**
`nmap -sV -sC -T4 10.129.14.1`

**Results:**
The scan revealed that port 21 was open, running an FTP (File Transfer Protocol) service. FTP is commonly used for transferring files between systems and often allows anonymous access in misconfigured environments.

**Key Findings:**
- Port 21: FTP service running
- Anonymous login potentially enabled
- No encryption (clear text protocol)

---

### Gaining Access (FTP Exploitation)

With FTP identified as the target service, I proceeded to test for anonymous access.

1. **FTP Connection:**
   I connected to the FTP service using the built-in FTP client:
   `ftp 10.129.14.1`

2. **Anonymous Authentication:**
   When prompted for credentials, I attempted anonymous login:
   - Username: `anonymous`
   - Password: (blank/empty)

3. **Successful Access:**
   The anonymous login was successful, granting me access to the FTP server without authentication.

4. **Directory Exploration:**
   Once connected, I explored the available directories and files:
   ```bash
   ls -la
   pwd
   ```

---

### Capturing the Flag

After gaining access to the FTP server, I located and retrieved the flag:

1. **File Discovery:**
   Listed all files in the current directory to identify the flag file:
   `ls -la`

2. **Flag Download:**
   Downloaded the flag file to my local machine:
   `get flag.txt`

3. **Local Verification:**
   After the download completed, I exited the FTP session and read the flag:
   ```bash
   quit
   cat flag.txt
   ```

   This revealed the flag needed to complete the challenge.

---

### Lessons Learned

This machine highlights several critical FTP security principles:

1. **Anonymous Access Risks:** Allowing anonymous FTP access can expose sensitive files to unauthorized users. This should be disabled unless absolutely necessary.

2. **Protocol Security:** FTP transmits data in clear text, making it vulnerable to network sniffing. SFTP or FTPS should be used for secure file transfers.

3. **Access Controls:** FTP servers should implement proper authentication mechanisms and restrict access to authorized users only.

4. **File Permissions:** Even with restricted access, files on FTP servers should have appropriate permissions to prevent unauthorized access to sensitive data.

5. **Regular Auditing:** FTP configurations should be regularly reviewed to ensure no sensitive files are inadvertently exposed.

6. **Alternative Solutions:** Consider more secure alternatives like SCP, SFTP, or HTTPS-based file transfer methods for sensitive data.
