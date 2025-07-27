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

As always, the first step was a thorough port scan with Nmap to identify open services.

**Command:**
`nmap -sV -sC -T4 10.129.16.169`

**Results:**
The Nmap scan revealed that port 445 was open, running the `microsoft-ds` service, which indicates SMB.

---

### Gaining Access (SMB Enumeration)

With an open SMB port, the next logical step was to enumerate the available shares. I used the `smbclient` tool for this.

1.  First, I listed the available shares on the target, which does not require a password.
    `smbclient -L 10.129.16.169`

2.  The output showed a few shares, with one named 'Workshares' looking particularly interesting. I attempted to connect to this share, again without a password.
    `smbclient //10.129.16.169/Workshares`

3.  The connection was successful, granting me anonymous access to the share's contents.

---

### Capturing the Flag

Once connected to the share, I was at an `smb: \>` prompt.

1.  I used the `ls` command to list the files within the share.
2.  This revealed a file containing the flag.
3.  I used the `get flag.txt` command to download the flag to my local machine, successfully completing the challenge.

---

### Lessons Learned

This machine is a critical lesson in the dangers of misconfigured SMB shares. Allowing anonymous, unauthenticated access to any file share, no matter how seemingly unimportant, can lead to a full system compromise. Network shares should always be protected with strong passwords and proper access controls.
