# Vulnhub-Bluemoon-Lab
Lab report and walkthrough for Vulnhub Bluemoon (Week 3 Task).

# Vulnerability Analysis - Week 3: Vulnhub Bluemoon

This repository contains the documentation, walkthrough, and lab report for **Week 3 Lab: Vulnhub - Bluemoon**.

* **Machine Name:** BlueMoon-2021
* **Author:** Kirthik-[H4ck3r]
* **Difficulty:** Easy
* **Goal:** Get Root
* **Student Name:** Nurul Rabiatul Ifah Binti Mohd Azizi (L01-B01)

---

## Lab Walkthrough & Attack Chain

### Phase 1: Reconnaissance & Port Scanning
* Used `ifconfig` to verify local network interface configurations[cite: 3].
* Executed an Nmap ping scan (`nmap -sn 172.20.10.0/24`) to discover active hosts on the network and identify the target machine's IP address[cite: 4].

### Phase 2: Web Enumeration & Credential Extraction
* Performed directory enumeration on the target web server, revealing a hidden directory (`/hidden_text`)[cite: 5, 6].
* Located a link leading to a QR code image (`QR_C0d3.png`)[cite: 6, 7].
* Decoded the QR code using online decoding tools, which yielded hardcoded FTP credentials (`user: userftp`, `password: ftpp@ssword`)[cite: 8].

### Phase 3: FTP Exploitation & File Retrieval
* Authenticated to the FTP service using the extracted credentials[cite: 8, 9].
* Downloaded two key files:
  * `information.txt`: A note identifying the target user `robin` and hinting at a weak password[cite: 10].
  * `p_lists.txt`: A custom password dictionary[cite: 10].

### Phase 4: SSH Brute-Forcing & Initial Access (`robin`)
* Used **Hydra** to brute-force the SSH service for user `robin` using the downloaded wordlist (`p_lists.txt`):
  ```bash
  hydra -l robin -P p_lists.txt ssh://<target-ip>
