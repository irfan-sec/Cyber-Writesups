# Hack The Box Write-up: Redeemer

- **Machine:** Redeemer
- **IP Address:** 10.129.75.236
- **Platform:** Hack The Box
- **Difficulty:** Very Easy (Tier 3)

---

### Summary

"Redeemer" is the Tier 3 Starting Point machine on Hack The Box. This challenge introduces interaction with a NoSQL database, **Redis**. The objective is to enumerate the Redis service, connect to it, and query the database to find the hidden flag.

---

### Enumeration

The reconnaissance phase began with a comprehensive port scan to identify open services and their configurations.

**Command:**
`nmap -sV -sC -T4 10.129.75.236`

**Results:**
The scan revealed that port 6379 was open, running a Redis key-value store. Redis is an in-memory data structure store commonly used as a database, cache, and message broker.

**Key Findings:**
- Port 6379: Redis service running (default port)
- No authentication required (insecure configuration)
- In-memory database accessible over network

**Additional Redis Enumeration:**
```bash
# Check Redis version and configuration
nmap --script redis-info -p 6379 10.129.75.236

# Test for authentication requirements
redis-cli -h 10.129.75.236 ping
```

---

### Gaining Access (Redis Interaction)

With Redis identified as the target service, I proceeded to connect and enumerate the database contents.

**Tool Installation:**
To interact with Redis, the `redis-cli` tool is required:
```bash
# On Kali Linux
sudo apt install redis-tools

# On Ubuntu/Debian
sudo apt-get install redis-tools
```

**Connection and Enumeration:**

1. **Initial Connection:**
   I connected to the Redis server using the command-line interface:
   `redis-cli -h 10.129.75.236`
   
   The connection was successful without authentication, indicating a misconfigured Redis instance.

2. **Server Information Gathering:**
   I gathered information about the Redis server:
   ```bash
   INFO                    # Get comprehensive server information
   INFO server             # Get server-specific information
   CONFIG GET "*"          # List all configuration parameters
   ```

3. **Database Exploration:**
   I explored the database structure and contents:
   ```bash
   DBSIZE                  # Get number of keys in current database
   SELECT 0                # Switch to database 0 (default)
   KEYS *                  # List all keys in current database
   ```

4. **Key Enumeration:**
   The `KEYS *` command revealed several keys stored in the database. I examined each key to understand the data structure:
   ```bash
   TYPE flag               # Check the data type of 'flag' key
   GET flag                # Retrieve the value of 'flag' key
   ```

5. **Flag Discovery:**
   One of the keys contained the flag. I retrieved its value:
   `GET flag`
   
   This command returned the flag, successfully completing the challenge.

**Alternative Redis Commands:**
Other useful Redis commands for enumeration:
```bash
SCAN 0                      # Iterate through keys (better for large datasets)
RANDOMKEY                   # Get a random key
FLUSHALL                    # Clear all databases (destructive - use with caution)
```

---

### Lessons Learned

This machine provides valuable insights into NoSQL database security, specifically Redis:

1. **Authentication Security:** Redis instances should always require authentication. Use the `AUTH` command with strong passwords to secure access.

2. **Network Exposure:** Redis should not be exposed to public networks. Use firewalls and network segmentation to restrict access.

3. **Configuration Hardening:** Default Redis configurations are often insecure and should be hardened:
   - Enable authentication (`requirepass`)
   - Bind to specific interfaces only
   - Disable dangerous commands (`FLUSHALL`, `CONFIG`, etc.)

4. **Data Sensitivity:** Even cache and temporary data stores can contain sensitive information that should be protected.

5. **Access Controls:** Implement proper access controls and use Redis ACLs (Access Control Lists) for fine-grained permissions.

6. **Monitoring:** Implement logging and monitoring for Redis instances to detect unauthorized access.

7. **Regular Updates:** Keep Redis updated to the latest version to protect against known vulnerabilities.

8. **Encryption:** Use TLS/SSL encryption for Redis connections, especially over networks.
