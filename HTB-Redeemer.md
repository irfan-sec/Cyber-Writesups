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

The initial reconnaissance began with a standard Nmap scan to identify all open services.

**Command:**
`nmap -sV -sC -T4 10.129.75.236`

**Results:**
The scan revealed that port 6379 was open, running a Redis key-value store. This immediately became the primary target for investigation.

---

### Gaining Access (Redis Interaction)

To interact with the Redis service, the `redis-cli` tool is required.
*(Note: To install this on Kali Linux, the command is `sudo apt install redis-tools`)*

1.  I connected to the Redis server using the command-line interface:
    `redis-cli -h 10.129.75.236`

2.  Once connected, I explored the database. I used the `INFO` command to get more details about the server and the `KEYS *` command to list all keys stored in the database.

3.  The `KEYS *` command revealed a key that looked like it contained the flag. I used the `GET` command to retrieve the value associated with that key.
    `GET flag`

4.  This command returned the flag, successfully completing the challenge.

---

### Lessons Learned

This machine is a critical lesson in securing database services. Exposing a Redis instance to the network without any authentication (a password) is extremely dangerous, as it allows anyone to connect and read (or even write) sensitive data. All database services must have strong authentication controls in place.
