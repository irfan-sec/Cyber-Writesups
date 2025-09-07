# Hack The Box Write-up: Ignition

- **Machine:** Ignition
- **IP Address:** 10.129.1.27
- **Platform:** Hack The Box
- **Difficulty:** Very Easy (Tier 6)

---

### Summary

"Ignition" is the Tier 6 Starting Point machine on Hack The Box. This challenge introduces web application enumeration and exploitation through default credentials. The machine hosts a Magento e-commerce platform with default administrative credentials, demonstrating the critical importance of changing default passwords in web applications.

---

### Enumeration

The initial reconnaissance began with port scanning to identify running services.

**Command:**
`nmap -sV -sC -T4 10.129.1.27`

**Results:**
The scan revealed that port 80 was open, running an Apache HTTP server. This indicated a web application was hosted on the target machine.

**Web Enumeration:**
I proceeded to examine the web application:

1. Navigated to `http://10.129.1.27` in a web browser
2. The site appeared to be a Magento e-commerce platform
3. I identified the admin login page at: `http://10.129.1.27/admin`

---

### Gaining Access (Web Application Exploitation)

With the admin login page discovered, I attempted to gain access using common default credentials.

1. **Initial Login Attempts:**
   I tried various common default credential combinations:
   - admin:admin
   - admin:password
   - administrator:password

2. **Successful Authentication:**
   The credentials `admin:qwerty123` successfully granted access to the Magento administrative panel.

3. **Admin Panel Access:**
   Once authenticated, I had full administrative access to the Magento e-commerce platform, allowing me to view all system configurations, user data, and administrative functions.

---

### Capturing the Flag

After gaining administrative access to the Magento panel:

1. I explored the administrative interface to locate the flag.

2. The flag was found within the administrative dashboard, either displayed directly or accessible through one of the administrative functions.

3. Successfully retrieved the flag to complete the challenge.

---

### Lessons Learned

This machine highlights several critical web application security principles:

1. **Default Credentials:** Never leave default usernames and passwords in production systems. The combination of `admin:qwerty123` represents a common weak password that should be immediately changed after installation.

2. **Password Policies:** Implement strong password requirements for all administrative accounts. Passwords should be complex, unique, and regularly updated.

3. **Admin Interface Security:** Administrative interfaces should be properly secured with:
   - Strong authentication mechanisms
   - Multi-factor authentication where possible
   - IP-based access restrictions
   - Regular security updates

4. **Web Application Hardening:** E-commerce platforms like Magento require regular updates and proper security configuration to prevent unauthorized access.

5. **Security Testing:** Organizations should regularly test their web applications for default credentials and other common security misconfigurations.

6. **Access Logging:** All administrative access should be logged and monitored for suspicious activity.