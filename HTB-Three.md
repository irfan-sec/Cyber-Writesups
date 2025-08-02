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

Initial reconnaissance started with a standard Nmap scan to identify open services.

**Command:**
`nmap -sV -sC -T4 10.129.110.34`

**Results:**
The Nmap scan revealed a web server running on port 80 (HTTP). Browse to the website showed a simple page. Further enumeration of the website revealed a directory or link that exposed the name of an S3 bucket.

---

### Gaining Access (S3 Bucket Enumeration)

The key vulnerability was a misconfigured S3 bucket that allowed anonymous listing and reading of its contents. To interact with the bucket, I used the **AWS Command Line Interface (CLI)**.

*(Note: To install the AWS CLI on Kali, use: `sudo apt install awscli`)*

1.  First, I listed the contents of the S3 bucket anonymously using the `--no-sign-request` flag.
    `aws s3 ls s3://<bucket-name>/ --no-sign-request` 
    *(Replace <bucket-name> with the name you discovered)*

2.  This command showed a file that likely contained the flag. I then copied this file from the S3 bucket to my local machine.
    `aws s3 cp s3://<bucket-name>/<flag-file-name> . --no-sign-request`

---

### Flag Capture

After downloading the file from the S3 bucket, I read its contents to retrieve the flag, successfully completing the challenge.

---

### Lessons Learned

This machine is a critical lesson in cloud security fundamentals. S3 buckets and other cloud storage must have strict, private-by-default permissions. Allowing public, anonymous `List` and `Read` access is a major security flaw that can lead to significant data exposure for an organization.
