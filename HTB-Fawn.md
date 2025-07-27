# Hack The Box Write-up: Meow

- **Machine:** Fawn
- **IP Address:** 10.129.14.1
- **Platform:** Hack The Box
- **Difficulty:** Very Easy

---

### Summary

Fawn was my second machine on Hack The Box. This box served as an excellent introduction to basic network enumeration using Nmap and demonstrated the security risks associated with leaving insecure services like Ftp exposed with default or weak credentials.

---

### Enumeration

The first step was to perform a port scan to identify open services on the target. I used Nmap for this purpose.

**Command:**
`nmap -sV -sC -T4 10.129.14.1`

**Results:**
The scan revealed that port 21 was open and running the Telnet service.


---

### Gaining Access & Privilege Escalation

Based on the enumeration results, Telnet was the clear attack vector.

1.  I connected to the service using the built-in ftp client:
    `ftp 10.129.14.1`

2.  The service prompted for a login. I tried the common default username **`anonymous`**.

3.  It did not require a password. I was immediately granted a root shell. No privilege escalation was necessary as I had gained initial access as the highest-level user.

---

### Capturing the Root Flag

Once logged in as the `anonymous` user, I listed the files in the home directory to find the flag.

**Commands:**
```bash
ls -la

### Download the files
after that I dwonload file called flag.txt by command
get flag.txt

for view what inside I just add command 
cat flag.txt
