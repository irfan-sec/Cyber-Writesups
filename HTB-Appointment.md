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

---

### Gaining Access (SQL Injection)

Navigating to the web server revealed a login page for an appointment system.

1.  I first tested for SQL Injection by inputting a single quote (`'`) into the username and password fields. This caused the application to return a database error, which is a strong indicator of a potential SQLi vulnerability.

2.  To exploit this, I used a classic boolean-based SQLi payload in one of the fields to bypass the login logic:
    username = admin'#
    password = abc

3.  This payload successfully manipulated the SQL query on the backend, causing the login check to always evaluate to 'true' and granting me access to the authenticated area of the application.

---

### Flag Capture

After bypassing the login with the SQLi payload, I was redirected to an internal page. The flag was displayed directly on this page, confirming a successful compromise.

---

### Lessons Learned

This machine is a fundamental example of why all user-supplied input must be strictly sanitized and validated before being used in database queries. Using prepared statements or parameterized queries is essential to prevent SQL Injection, which remains one of the most common and damaging web application vulnerabilities.

Here are all steps:

What does the acronym SQL stand for?

Structured Query Language

What is one of the most common type of SQL vulnerabilities?

sql injection

What is the 2021 OWASP Top 10 classification for this vulnerability?

A03:2021-Injection

What does Nmap report as the service and version that are running on port 80 of the target?

Apache httpd 2.4.38 ((Debian))

What is the standard port used for the HTTPS protocol?

443

What is a folder called in web-application terminology?

directory

What is the HTTP response code is given for ‘Not Found’ errors?

404

Gobuster is one tool used to brute force directories on a webserver. What switch do we use with Gobuster to specify we’re looking to discover directories, and not subdomains?

dir

What single character can be used to comment out the rest of a line in MySQL?

#

If user input is not handled carefully, it could be interpreted as a comment. Use a comment to login as admin without knowing the password. What is the first word on the webpage returned?

Congratulations

Submit root flag

login: admin'#

password: abc
