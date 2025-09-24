# Hack The Box Write-up: Meow

- **Machine:** Meow
- **IP Address:** 10.129.1.17
- **Platform:** Hack The Box
- **Difficulty:** Very Easy

---

### Summary

"Meow" is the first machine in the Hack The Box Starting Point series, designed as an introduction to basic network enumeration and exploitation. This "Very Easy" challenge demonstrates the critical security risks associated with legacy protocols like Telnet and the dangers of passwordless root access.

---

### Enumeration

The reconnaissance phase began with a comprehensive port scan to identify open services and their configurations.

**Command:**
`nmap -sV -sC -T4 10.129.1.17`

**Results:**
The scan revealed that port 23 was open, running the Telnet service. Telnet is an unencrypted, legacy remote access protocol that transmits all data, including credentials, in plain text.

**Key Findings:**
- Port 23: Telnet service running
- No encryption (clear text protocol)
- Potentially weak authentication

**Additional Enumeration:**
To gather more information about the service, I also performed:
- Banner grabbing to identify service version
- OS fingerprinting to determine target system type

---

### Gaining Access (Telnet Exploitation)

Based on the enumeration results, Telnet was identified as the primary attack vector.

1. **Initial Connection:**
   I connected to the Telnet service using the built-in client:
   `telnet 10.129.1.17`

2. **Authentication Testing:**
   The service prompted for a login. I attempted authentication using common default credentials:
   - Username: `root`
   - Password: (blank/empty)

3. **Successful Access:**
   The login was successful without requiring a password, immediately granting me root-level access to the system. This represents a critical security misconfiguration.

4. **System Access Verification:**
   After successful authentication, I verified my access level:
   ```bash
   whoami
   id
   pwd
   ```

---

### Capturing the Flag

Once authenticated with root privileges, I proceeded to locate and retrieve the flag:

1. **Directory Exploration:**
   I explored the file system to locate the flag:
   ```bash
   ls -la
   find / -name "*flag*" -type f 2>/dev/null
   ```

2. **Flag Discovery:**
   Found the flag file in the current directory:
   ```bash
   ls -la
   ```

3. **Flag Retrieval:**
   Read the contents of the flag file:
   ```bash
   cat flag.txt
   ```

   This revealed the flag needed to complete the challenge.

---

### Lessons Learned

This introductory machine highlights several fundamental security principles:

1. **Legacy Protocol Risks:** Telnet transmits all data in clear text, making it vulnerable to network sniffing and man-in-the-middle attacks. SSH should be used instead.

2. **Authentication Security:** Systems should never allow passwordless access, especially for privileged accounts like root.

3. **Privilege Principle:** Following the principle of least privilege, root access should be restricted and require strong authentication.

4. **Network Security:** Insecure services like Telnet should not be exposed to networks, especially the internet.

5. **Default Configurations:** Default system configurations must be hardened before deployment to production environments.

6. **Access Controls:** Implement proper access controls and authentication mechanisms for all remote access services.
