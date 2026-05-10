# Bug-Hunting-
# Jenkins Arbitrary File Read Analysis (CVE-2024-23897)

## 📌 Overview
[cite_start]This report demonstrates the identification and exploitation of an **Arbitrary File Read** vulnerability in Jenkins[cite: 3]. [cite_start]The vulnerability stems from insecure CLI argument parsing, allowing unauthenticated attackers to retrieve sensitive files from the server using specially crafted input[cite: 4].

## 🎯 Target Information
* [cite_start]**Target URL:** `http://123.60.57.169:8080/` [cite: 12]
* [cite_start]**Service:** Jenkins Web Interface [cite: 13]
* [cite_start]**Vulnerability Version:** Potentially $\le 2.441$ [cite: 14]
* [cite_start]**CVE ID:** CVE-2024-23897 [cite: 14]
* [cite_start]**Severity:** **High (CVSS 7.5)** [cite: 15]

---

## 🔍 Attack Lifecycle

### 1. Reconnaissance
[cite_start]Target enumeration was conducted using **Shodan** to find internet-facing Jenkins services[cite: 20]. [cite_start]Publicly exposed instances were fingerprinted using specific HTTP headers[cite: 25]:
* [cite_start]`X-Jenkins` [cite: 23]
* [cite_start]`X-Hudson` [cite: 24]

### 2. Scanning & Enumeration
[cite_start]The **Nuclei** vulnerability scanner was utilized to validate the flaw[cite: 32]. 
* [cite_start]**Command:** `nuclei -u http://123.60.57.169:8080 -tags cve` [cite: 34]
* [cite_start]**Observation:** The scan confirmed that the application improperly processes `@file-path` inputs[cite: 39]. [cite_start]Two matches were found in approximately 10 seconds[cite: 57].

### 3. Vulnerability Analysis
[cite_start]The root cause lies in the Jenkins CLI argument parsing logic[cite: 61]:
* [cite_start]The `@` symbol is incorrectly interpreted as a file reference[cite: 62].
* [cite_start]Jenkins reads the file contents directly into the command arguments without sanitization[cite: 63].
* [cite_start]This allows **unauthenticated** direct access to the server filesystem[cite: 66, 67].

---

## 🚀 Exploitation & Evidence
[cite_start]The exploitation phase utilized a Python script to inject payloads into CLI arguments[cite: 111, 113].

### File Retrieval: `/etc/passwd`
[cite_start]**Command:** `python3 exploit.py http://123.60.57.169:8080 @/etc/passwd` [cite: 111]
Extracted system users include:
* [cite_start]`root:x:0:0:root:/root:/bin/bash` [cite: 135]
* [cite_start]`dev:x:1001:1001::/home/dev:/bin/bash` [cite: 128]
* [cite_start]`zabbix:x:997:995:Zabbix Monitoring System:/var/lib/zabbix` [cite: 127]

### File Retrieval: `/etc/shadow`
[cite_start]**Command:** `python3 51993.py - http://123.60.57.169:8080/p/etc/shadow` [cite: 137]
Extracted hashes for sensitive accounts:
* [cite_start]**Root Hash:** `$6$U6rhtRAk$viy/AAyNt9fclXg1...` [cite: 146]
* [cite_start]**Dev Hash:** `$6$CGS8Hf2T$g.C6hkY1LVQcLUJU...` [cite: 144]

---

## ⚠️ Impact & Risk
* [cite_start]**Confidential Data Disclosure:** Access to internal system files[cite: 164].
* [cite_start]**Credential Leakage:** Potential extraction of SSH keys, API tokens, and Jenkins credentials[cite: 158, 159, 160].
* [cite_start]**Privilege Escalation:** Exposed hashes allow for offline cracking and further compromise[cite: 166].

## 🛡️ Mitigation Recommendations
1.  [cite_start]**Upgrade Jenkins:** Update to version **2.442** or later, or LTS **2.426.3** or later[cite: 92, 169].
2.  [cite_start]**Disable CLI Remoting:** Disable this feature if it is not business-critical[cite: 170].
3.  [cite_start]**Network Access Control:** Restrict Jenkins access via firewall or VPN[cite: 171].
4.  [cite_start]**Security Monitoring:** Implement authentication and monitor for suspicious CLI activity[cite: 172].

---
> [cite_start]**Disclaimer:** This report and its associated PoC are for educational and authorized security testing purposes only[cite: 175].
