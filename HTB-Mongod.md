# Hack The Box Write-up: Mongod

- **Machine:** Mongod
- **IP Address:** 10.129.228.30
- **Platform:** Hack The Box
- **Difficulty:** Very Easy (Tier 5)

---

### Summary

"Mongod" is the Tier 5 Starting Point machine on Hack The Box. This challenge focuses on MongoDB enumeration and exploitation. The machine demonstrates the security risks associated with exposing MongoDB databases without proper authentication, allowing unauthorized access to sensitive data.

---

### Enumeration

The reconnaissance phase started with a thorough Nmap scan to identify open services and their configurations.

**Command:**
`nmap -sV -sC -T4 10.129.228.30`

**Results:**
The scan revealed that port 27017 was open, running MongoDB. This is the default port for MongoDB databases, making it the obvious target for investigation.

---

### Gaining Access (MongoDB Enumeration)

With MongoDB exposed, I proceeded to connect and enumerate the database contents.

1. First, I connected to the MongoDB instance using the `mongo` client:
   `mongo 10.129.228.30:27017`

2. Once connected, I explored the available databases using MongoDB commands:
   `show dbs`

3. This revealed several databases. I focused on examining each database to find interesting data:
   `use [database_name]`

4. For each database, I listed the available collections:
   `show collections`

5. I examined the contents of various collections using the `find()` command:
   `db.[collection_name].find()`

6. Eventually, I discovered a collection containing sensitive information, including what appeared to be the flag.

---

### Capturing the Flag

Through systematic enumeration of the MongoDB databases and collections:

1. I identified a database containing user data or configuration information.

2. Within one of the collections, I found a document containing the flag.

3. Used the `find()` command to retrieve the flag:
   `db.[collection_name].find().pretty()`

The `.pretty()` method formatted the output for easier reading, revealing the flag needed to complete the challenge.

---

### Lessons Learned

This machine provides important insights into NoSQL database security:

1. **Database Authentication:** MongoDB instances should never be exposed to the network without authentication enabled. Default installations often have authentication disabled, creating immediate security vulnerabilities.

2. **Network Exposure:** Database services should not be directly accessible from external networks. They should be properly firewalled and accessible only from authorized application servers.

3. **Data Sensitivity:** Even development or test databases can contain sensitive information that could be valuable to attackers. All databases should be treated as potentially sensitive.

4. **Default Configurations:** Many NoSQL databases ship with security features disabled by default for ease of development. These settings must be hardened before production deployment.

5. **Regular Auditing:** Database contents should be regularly audited to ensure no sensitive information is inadvertently stored in easily accessible locations.