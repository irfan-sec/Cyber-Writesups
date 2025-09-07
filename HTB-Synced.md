# Hack The Box Write-up: Synced

- **Machine:** Synced
- **IP Address:** 10.129.96.155
- **Platform:** Hack The Box
- **Difficulty:** Very Easy (Tier 7)

---

### Summary

"Synced" is the Tier 7 Starting Point machine on Hack The Box. This challenge focuses on Rsync service enumeration and exploitation. The machine demonstrates the security risks of misconfigured Rsync services that allow anonymous access to sensitive directories, potentially exposing confidential files and system information.

---

### Enumeration

The reconnaissance phase began with a comprehensive port scan to identify available services.

**Command:**
`nmap -sV -sC -T4 10.129.96.155`

**Results:**
The scan revealed that port 873 was open, running the Rsync service. Rsync is a file synchronization protocol commonly used for backing up and mirroring files between systems.

**Rsync Enumeration:**
With Rsync identified, I proceeded to enumerate available shares and modules.

1. **List Available Modules:**
   `rsync --list-only rsync://10.129.96.155/`
   
   This command revealed available Rsync modules that could be accessed anonymously.

2. **Module Discovery:**
   The enumeration showed a module named "public" that appeared to allow anonymous access.

---

### Gaining Access (Rsync Exploitation)

With anonymous Rsync access available, I proceeded to explore the accessible content.

1. **Browse Module Contents:**
   `rsync --list-only rsync://10.129.96.155/public/`
   
   This command listed the files and directories available in the "public" module.

2. **File Discovery:**
   The listing revealed several files and directories, including what appeared to be flag-related content.

3. **Download Files:**
   I synchronized the entire public module to my local machine:
   `rsync -av rsync://10.129.96.155/public/ ./synced_files/`
   
   This downloaded all accessible files for local examination.

---

### Capturing the Flag

After downloading the files from the Rsync service:

1. **Local File Examination:**
   I examined the downloaded files to locate the flag:
   ```bash
   ls -la synced_files/
   find synced_files/ -name "*flag*" -o -name "*Flag*"
   ```

2. **Flag Discovery:**
   Found the flag file within the synchronized directory structure.

3. **Flag Retrieval:**
   ```bash
   cat synced_files/flag.txt
   ```
   
   Successfully retrieved the flag to complete the challenge.

---

### Lessons Learned

This machine provides valuable insights into file synchronization service security:

1. **Rsync Security:** Rsync services should never allow anonymous access to sensitive directories. All Rsync modules should require proper authentication.

2. **Access Controls:** Implement strict access controls for file synchronization services. Use authentication mechanisms and restrict access to authorized users only.

3. **Data Classification:** Understand what data is being synchronized and ensure sensitive information is not accessible through public modules.

4. **Network Segmentation:** File synchronization services should be restricted to internal networks and not exposed to the internet without proper security controls.

5. **Regular Auditing:** Regularly audit Rsync configurations to ensure no sensitive data is inadvertently exposed through misconfigured modules.

6. **Alternative Solutions:** Consider more secure alternatives for file synchronization that provide better access controls and encryption, especially for sensitive data.

7. **Monitoring:** Implement logging and monitoring for all file synchronization activities to detect unauthorized access attempts.