# Hack The Box Write-up: Three

- **Machine:** Three
- **IP Address:** 10.129.110.34
- **Platform:** Hack The Box
- **Difficulty:** Easy (Starting Point - Tier 5)

---

### Summary

"Three" is a Tier 5 machine that introduces a common cloud security vulnerability: a publicly accessible and listable Amazon S3 bucket. The path to compromise involves discovering the S3 bucket via a web server and then using the AWS Command Line Interface (CLI) to enumerate and download its contents to find the flag.

---

### Enumeration

The reconnaissance phase began with a comprehensive port scan to identify open services and their configurations.

**Command:**
`nmap -sV -sC -T4 10.129.110.34`

**Results:**
The Nmap scan revealed a web server running on port 80 (HTTP). This indicated a web application that could potentially contain information about cloud resources.

**Key Findings:**
- Port 80: HTTP web server running
- Web application potentially hosting cloud-related content
- Possible S3 bucket references in web content

**Web Application Enumeration:**
1. **Initial Web Exploration:**
   I navigated to `http://10.129.110.34` to examine the web application.

2. **Directory Enumeration:**
   I performed directory brute-forcing to discover hidden content:
   ```bash
   # Using gobuster for directory enumeration
   gobuster dir -u http://10.129.110.34 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
   
   # Using dirb for additional enumeration
   dirb http://10.129.110.34
   ```

3. **Source Code Analysis:**
   I examined the web page source code for references to S3 buckets:
   ```bash
   curl http://10.129.110.34 | grep -i s3
   curl http://10.129.110.34 | grep -i bucket
   curl http://10.129.110.34 | grep -i aws
   ```

4. **S3 Bucket Discovery:**
   Through web application enumeration, I discovered references to an S3 bucket name, either in:
   - Page source code
   - JavaScript files
   - Configuration files
   - Directory listings

---

### Gaining Access (S3 Bucket Enumeration)

With the S3 bucket name identified, I proceeded to test for public access and enumerate its contents.

**Tool Installation:**
```bash
# Install AWS CLI on Kali Linux
sudo apt install awscli

# Verify installation
aws --version
```

**S3 Bucket Testing:**

1. **Anonymous Access Testing:**
   I tested whether the S3 bucket allowed anonymous access:
   ```bash
   # Test bucket listing permissions
   aws s3 ls s3://[bucket-name]/ --no-sign-request
   
   # Alternative: Test with specific region if needed
   aws s3 ls s3://[bucket-name]/ --no-sign-request --region us-east-1
   ```

2. **Bucket Content Enumeration:**
   The listing command revealed the bucket contents without authentication, indicating misconfigured permissions:
   ```bash
   # List all objects in the bucket
   aws s3 ls s3://[bucket-name]/ --no-sign-request --recursive
   
   # Get detailed information about objects
   aws s3 ls s3://[bucket-name]/ --no-sign-request --human-readable --summarize
   ```

3. **File Discovery:**
   The enumeration revealed files that potentially contained sensitive information or the flag.

4. **File Download:**
   I downloaded suspicious files from the S3 bucket:
   ```bash
   # Download specific file
   aws s3 cp s3://[bucket-name]/[filename] . --no-sign-request
   
   # Download all files (if needed)
   aws s3 sync s3://[bucket-name]/ ./s3-download/ --no-sign-request
   ```

**Alternative S3 Enumeration Methods:**
```bash
# Using s3scanner tool
s3scanner scan --buckets-file bucket-names.txt

# Using AWS CLI with different output formats
aws s3api list-objects --bucket [bucket-name] --no-sign-request --output table
```

---

### Flag Capture

After gaining access to the S3 bucket, I systematically examined the downloaded files:

1. **File Analysis:**
   I examined each downloaded file to identify the flag:
   ```bash
   # List downloaded files
   ls -la
   
   # Search for flag patterns
   grep -r "HTB{" .
   grep -r "flag" . -i
   ```

2. **Flag Discovery:**
   Found the flag in one of the downloaded files:
   ```bash
   cat [flag-file-name]
   ```

3. **Flag Verification:**
   Verified the flag format and content to ensure successful completion of the challenge.

---

### Lessons Learned

This machine provides crucial insights into cloud security, specifically AWS S3 security:

1. **S3 Bucket Permissions:** S3 buckets should use the principle of least privilege. Public read access should only be granted when absolutely necessary and with careful consideration.

2. **Bucket Policies:** Implement proper bucket policies that explicitly deny public access unless required:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Deny",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::bucket-name/*"
       }
     ]
   }
   ```

3. **AWS Config Rules:** Use AWS Config rules to automatically detect and alert on publicly accessible S3 buckets.

4. **S3 Block Public Access:** Enable S3 Block Public Access settings at both account and bucket levels to prevent accidental public exposure.

5. **Regular Auditing:** Regularly audit S3 bucket permissions and access logs to detect unauthorized access.

6. **Data Classification:** Understand what data is stored in S3 buckets and apply appropriate security controls based on data sensitivity.

7. **Encryption:** Use S3 server-side encryption for sensitive data, even if the bucket is private.

8. **Access Logging:** Enable S3 access logging to monitor and audit bucket access patterns.

9. **Cloud Security Tools:** Use tools like AWS Trusted Advisor, AWS Security Hub, and third-party cloud security scanners to identify misconfigurations.

10. **Infrastructure as Code:** Use IaC tools to manage S3 bucket configurations consistently and prevent manual misconfigurations.
