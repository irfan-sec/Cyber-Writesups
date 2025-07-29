# Hack The Box Write-up: Sequel

- **Machine:** Sequel
- **IP Address:** 10.129.95.232
- **Platform:** Hack The Box
- **Difficulty:** Easy (Starting Point - Tier 1)

---

### Summary

"Sequel" is a Tier 1 machine that builds upon the SQL theme from the previous challenge. The primary vulnerability is a publicly exposed MySQL database service, which is misconfigured with default credentials (a blank root password). The path to compromise involves connecting directly to the database and querying it to find the flag.

---

### Enumeration

The initial reconnaissance started with a standard Nmap scan to identify open ports and services.

**Command:**
`nmap -sV -sC -T4 10.129.95.232`

**Results:**
The Nmap scan revealed that the MySQL database port (3306) was open to the public. This immediately became the primary focus of the investigation.

---

### Gaining Access (MySQL Database)

With the MySQL port exposed, I attempted to connect directly to the database from my Kali terminal using the `mysql` command-line client.

1.  I used the following command to attempt a login as the common default user 'root':
    `mysql -h 10.129.95.232 -u root -p`

2.  When prompted for a password, I simply pressed Enter (for a blank password).

3.  The login was successful, granting me full administrative access to the MySQL server.

---

### Flag Capture

Once inside the database, I used standard SQL queries to explore its contents and find the flag.

1.  List all databases: `SHOW DATABASES;`
2.  Switch to the relevant database (e.g., `htb`): `USE htb;`
3.  List all tables in the database: `SHOW TABLES;`
4.  View the contents of the relevant table: `SELECT * FROM users;` (or a similar table name)

The flag was stored in plain text within one of the tables. I retrieved it by running a final `SELECT` query.

---

### Lessons Learned

This machine is a powerful lesson in network security basics: **never expose a database service directly to the internet.** Databases should always be firewalled and only accessible from trusted application servers. Furthermore, production systems should never use default credentials, especially a blank password for the `root` user.
