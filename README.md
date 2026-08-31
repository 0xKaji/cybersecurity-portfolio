# Cybersecurity Portfolio & Lab Write-ups

Welcome to my cybersecurity portfolio! This repository serves as a collection of technical write-ups, penetration testing reports, and lab documentation from platforms like **TryHackMe** and **HackTheBox**. 

My goal is to showcase hands-on competencies in vulnerability assessment, offensive security (Red Team), and defensive remediation (Blue Team), mapped to industry standards like **MITRE ATT&CK**.

---

## 📂 Table of Contents

| Platform / Category | Machine / Project | Key Techniques & Vulnerabilities | Report |
| :--- | :--- | :--- | :--- |
| **Windows / AD** | [Attacktive Directory](https://tryhackme.com/room/attacktivedirectory) | Kerbrute, AS-REP Roasting, SMB Enumeration, DCSync, Pass-the-Hash | [View Report](./reports/attacktivedirectory.md) |
| **Linux / Web** | [Smol](https://tryhackme.com/room/smol) | LFI, Web Shell, Password Cracking, Sudo/Su Abuse | [View Report](./reports/smol.md) |
| *Coming Soon* | *Machine Name* | *TBD* | *Coming Soon* |

---

## 🛠️ Skills & Tooling Demonstrated

*   **Reconnaissance & Network Scanning:** Nmap, Masscan, Netcat, DNS enumeration tools, and network mapping.
*   **Web Application Pentesting:** SQL injection (SQLmap), Local/Remote File Inclusion (LFI/RFI), Cross-Site Scripting (XSS), and Web Shell deployment.
*   **Active Directory & Infrastructure Attacks:** Kerberos attacks (AS-REP Roasting, Kerberoasting), SMB/RPC enumeration, DCSync, and Pass-the-Hash/Ticket techniques.
*   **Credential Auditing & Cracking:** Offline password cracking using JohnTheRipper, Hashcat, and automated brute-forcing via Hydra/Medusa.
*   **Privilege Escalation & Post-Exploitation:** Automated enumeration scripts (LinPeas, WinPEAS), SUID/Sudo misconfigurations, kernel exploits, and lateral movement.
*   **Reporting & Frameworks:** MITRE ATT&CK Matrix mapping, Executive Summaries, and actionable Remediation Planning.

---

## 📊 MITRE ATT&CK Quick Mapping Reference

Most of the reports included in this portfolio map findings to the following core MITRE ATT&CK tactics:
*   **Initial Access:** [T1190] Exploit Public-Facing Application
*   **Execution:** [T1059] Command and Scripting Interpreter, [T1505] Server Software Component
*   **Credential Access:** [T1558] Steal or Forge Kerberos Tickets, [T1003] OS Credential Dumping, [T1552] Unsecured Credentials
*   **Privilege Escalation & Lateral Movement:** [T1548] Abuse Elevation Control Mechanism, [T1550] Use Alternate Authentication Material

---

## 📬 Contact

*   **[LinkedIn](https://www.linkedin.com/in/aimar-mu%C3%B1oz-a6a932403)** 
*   **Email:** aimarmunoz@proton.me
