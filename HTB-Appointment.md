# Hack The Box Write-up: Appointment

- **Machine:** Appointment
- **IP Address:** 10.129.168.197
- **Platform:** Hack The Box
- **Difficulty:** Easy (Starting Point - Tier 1)

---

### Summary

"Appointment" is a Tier 1 machine in the Starting Point path that provides a focused lesson on one of the most critical web vulnerabilities: **SQL Injection (SQLi)**. The challenge involves identifying a vulnerable login page and using a classic SQLi payload to bypass authentication and capture the flag.

---

### Enumeration

The initial reconnaissance began with a standard Nmap scan to identify open services.

**Command:**
`nmap -sV -sC -T4 10.129.168.197`

**Results:**
The Nmap scan revealed a web server running on port 80 (HTTP). This immediately became the primary attack surface.

**Web Application Discovery:**
1. Navigated to `http://10.129.168.197` in a web browser
2. Discovered an appointment system login page
3. Identified potential input fields for testing SQL Injection

---

### Gaining Access (SQL Injection)

Navigating to the web server revealed a login page for an appointment system.

1. **Initial Testing:**
   I first tested for SQL Injection by inputting a single quote (`'`) into the username and password fields. This caused the application to return a database error, which is a strong indicator of a potential SQLi vulnerability.

2. **Exploit Development:**
   To exploit this vulnerability, I used a classic boolean-based SQLi payload to bypass the login logic:
   - Username: `admin'#`
   - Password: `abc` (any value)

3. **Successful Bypass:**
   This payload successfully manipulated the SQL query on the backend. The `#` symbol comments out the password check portion of the query, causing the login check to always evaluate to 'true' and granting me access to the authenticated area of the application.

**Technical Explanation:**
The vulnerable SQL query likely looked something like:
```sql
SELECT * FROM users WHERE username='admin'#' AND password='abc'
```
The `#` symbol comments out everything after it, so the query becomes:
```sql
SELECT * FROM users WHERE username='admin'
```

---

### Flag Capture

After bypassing the login with the SQLi payload:

1. I was redirected to an internal page of the appointment system.
2. The flag was displayed directly on this authenticated page.
3. Successfully retrieved the flag to complete the challenge.

---

### Lessons Learned

This machine demonstrates fundamental web application security principles:

1. **Input Validation:** All user-supplied input must be strictly sanitized and validated before being used in database queries.

2. **Prepared Statements:** Using prepared statements or parameterized queries is essential to prevent SQL Injection vulnerabilities.

3. **SQL Injection Impact:** SQLi remains one of the most common and damaging web application vulnerabilities, often leading to data breaches and unauthorized access.

4. **Error Handling:** Applications should not expose database errors to users, as these can reveal information about the underlying database structure.

5. **Defense in Depth:** Implement multiple layers of security including WAFs, input validation, and proper database permissions.

6. **Security Testing:** Regular security testing and code reviews can help identify and remediate SQLi vulnerabilities before they're exploited.