# Hack The Box Write-up: Responder

- **Machine:** Responder
- **IP Address:** 10.129.81.233
- **Platform:** Hack The Box
- **Difficulty:** Easy (Starting Point - Tier 4)

---

### Summary

"Responder" is a Tier 4 machine that introduces a fundamental network attack: LLMNR/NBT-NS poisoning. The objective is to use the tool `Responder` to intercept a name resolution request from the victim machine, capture a user's Net-NTLMv2 password hash, and crack the hash offline to gain the user's password.

### Background: What is LLMNR/NBT-NS Poisoning?

When a Windows computer tries to find another computer on a network, it first uses DNS. If DNS fails, it falls back to older protocols like LLMNR and NBT-NS, where it essentially shouts out to the whole local network, "Does anyone know where [hostname] is?". An attacker can listen for these shouts and reply, "Yes, that's me! Send me your credentials." The tool `Responder` automates this poisoning attack.

---

### Enumeration

The initial Nmap scan was performed to identify the target as a Windows machine.

**Command:**
`nmap -sV -sC -T4 10.129.81.233`

**Results:**
The scan revealed port 445 (SMB) was open, strongly suggesting the target is a Windows machine and likely vulnerable to name resolution poisoning.

---

### The Attack: Capturing a Hash with Responder

The attack was a multi-step process.

1.  **Start Responder:** On my Kali machine, I started Responder, telling it to listen on my HTB VPN network interface (typically `tun0`).
    `sudo responder -I tun0`

2.  **Trigger the Hash:** I needed to make the victim machine try to authenticate to my attacker machine. In this challenge, this was likely triggered automatically or by visiting a specific page/service.

3.  **Capture the Hash:** Responder successfully intercepted a request and captured the Net-NTLMv2 hash for a user on the machine.

---

### Cracking the Hash & Gaining Access

The captured hash is not the password itself. It must be cracked offline.

1.  I saved the captured hash into a file (e.g., `hash.txt`).
2.  I used a password cracking tool like **John the Ripper** with a common wordlist to find the password.
    `john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`

3.  Once the password was cracked, I used it to log into the machine via a discovered service (like SMB or WinRM) to capture the flag.

---

### Lessons Learned

This machine is a critical lesson in the dangers of legacy network protocols. LLMNR and NBT-NS should be disabled in corporate environments to prevent this type of attack. It also highlights the importance of strong, complex passwords that are resistant to offline dictionary attacks.
