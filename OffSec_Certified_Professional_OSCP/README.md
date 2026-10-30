# OffSec Certified Professional (OSCP / PEN-200) — Complete Mastery Curriculum

## Overview & Exam Blueprint

The **OffSec Certified Professional (OSCP)** is the premier hands-on penetration testing certification. Unlike multiple-choice exams, the OSCP requires you to compromise live target machines across a simulated enterprise network within a **23-hour and 45-minute practical exam window**, followed by **24 hours to submit a professional penetration test report**.

### Official Exam Scoring Breakdown (100 Points Total; 70 Points Required to Pass)
- **Active Directory Set (40 Points — All or Nothing / Partial Credit Model):**
  - Consists of **3 machines** inside an enterprise Active Directory domain (1 initial foothold workstation/server, 1 intermediate lateral target, and 1 Domain Controller).
  - Full 40 points are awarded upon capturing `proof.txt` on the Domain Controller via a complete attack chain.
- **Standalone Machines (60 Points — 3 Machines @ 20 Points Each):**
  - Each standalone machine awards **10 points** for initial low-privilege foothold (`local.txt`) and **10 points** for privilege escalation to `root` or `NT AUTHORITY\SYSTEM` (`proof.txt`).
- **Bonus Points (10 Points):**
  - Awarded for completing 80% of PEN-200 topic exercises and submitting 30 `proof.txt` flags in the OffSec Challenge Labs.
- **Exam Restrictions:**
  - **Strictly Prohibited:** Metasploit `autopwn`, automated commercial scanners (Nessus, Acunetix, Qualys, Burp Suite Pro active scanner), `sqlmap` (and automated SQLi exploitation tools), AI/LLM code generation during the exam, and mass spoofing tools (`Responder` poisoning in shared networks unless isolated).
  - **Metasploit Usage Restriction:** `msfconsole` (`exploit/*`, `auxiliary/*`, `post/*`, and `meterpreter` payloads) may only be used against **ONE single target machine** of your choice. However, `msfvenom` (for generating non-Meterpreter staged/stageless payloads) and `exploit/multi/handler` (for catching standard reverse shells) are **unlimited** across all targets!

---

## Curriculum Architecture

- [OSCP Master Cheat Sheet](./OSCP_Master_Cheat_Sheet.md) — Rapid-reference command bible for Enumeration, Web Exploitation, Active Directory, PrivEsc, and Tunneling.
- **Domain 1: Active Information Gathering, Service Enumeration & Web Exploitation**
  - [Study Guide](./Domain_1_Enumeration_Web_Exploitation/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_1_Enumeration_Web_Exploitation/Exercises.md)
  - [Reference White Papers](./Domain_1_Enumeration_Web_Exploitation/White_Papers.md)
  - [Quiz 1: Service Enumeration & Web Exploitation Mastery](./Domain_1_Enumeration_Web_Exploitation/Quiz_1.md)
  - [Quiz 2: LFI, RFI, File Upload & SSTI Chains](./Domain_1_Enumeration_Web_Exploitation/Quiz_2.md)
  - [Quiz 3: Manual SQL Injection & Command Injection](./Domain_1_Enumeration_Web_Exploitation/Quiz_3.md)
- **Domain 2: Linux Privilege Escalation & Post-Exploitation**
  - [Study Guide](./Domain_2_Linux_Privilege_Escalation/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_2_Linux_Privilege_Escalation/Exercises.md)
  - [Reference White Papers](./Domain_2_Linux_Privilege_Escalation/White_Papers.md)
  - [Quiz 1: SUID, Capabilities & Sudo Exploitation](./Domain_2_Linux_Privilege_Escalation/Quiz_1.md)
  - [Quiz 2: Cron Jobs, PATH Hijacking & Wildcard Injection](./Domain_2_Linux_Privilege_Escalation/Quiz_2.md)
  - [Quiz 3: Containers, NFS & Kernel Escalation](./Domain_2_Linux_Privilege_Escalation/Quiz_3.md)
- **Domain 3: Windows Privilege Escalation & Token Manipulation**
  - [Study Guide](./Domain_3_Windows_Privilege_Escalation/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_3_Windows_Privilege_Escalation/Exercises.md)
  - [Reference White Papers](./Domain_3_Windows_Privilege_Escalation/White_Papers.md)
  - [Quiz 1: Token Privileges (`SeImpersonate`, `SeBackup`)](./Domain_3_Windows_Privilege_Escalation/Quiz_1.md)
  - [Quiz 2: Service Misconfigurations, Unquoted Paths & DLL Hijacking](./Domain_3_Windows_Privilege_Escalation/Quiz_2.md)
  - [Quiz 3: Registry Abuse, Credential Harvesting & UAC Bypass](./Domain_3_Windows_Privilege_Escalation/Quiz_3.md)
- **Domain 4: Active Directory Enumeration, Lateral Movement & Domain Dominance**
  - [Study Guide](./Domain_4_Active_Directory_Exploitation/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_4_Active_Directory_Exploitation/Exercises.md)
  - [Reference White Papers](./Domain_4_Active_Directory_Exploitation/White_Papers.md)
  - [Quiz 1: BloodHound, Kerberoasting & AS-REP Roasting](./Domain_4_Active_Directory_Exploitation/Quiz_1.md)
  - [Quiz 2: Lateral Movement (Pass-the-Hash, Overpass-the-Hash, WinRM/WMI/DCOM)](./Domain_4_Active_Directory_Exploitation/Quiz_2.md)
  - [Quiz 3: ACL Abuse, Delegation, ADCS & DCSync](./Domain_4_Active_Directory_Exploitation/Quiz_3.md)
- **Domain 5: Network Pivoting, Port Forwarding, Client-Side Attacks & Exploit Modification**
  - [Study Guide](./Domain_5_Pivoting_Tunneling_Exploit_Dev/Study_Guide.md)
  - [Hands-On Lab Exercises](./Domain_5_Pivoting_Tunneling_Exploit_Dev/Exercises.md)
  - [Reference White Papers](./Domain_5_Pivoting_Tunneling_Exploit_Dev/White_Papers.md)
  - [Quiz 1: Ligolo-ng, Chisel & Dynamic SOCKS Proxies](./Domain_5_Pivoting_Tunneling_Exploit_Dev/Quiz_1.md)
  - [Quiz 2: Client-Side Attacks, File Transfers & Living Off The Land](./Domain_5_Pivoting_Tunneling_Exploit_Dev/Quiz_2.md)
  - [Quiz 3: Public Exploit Adaptation, Shellcode Fixing & Cross-Compilation](./Domain_5_Pivoting_Tunneling_Exploit_Dev/Quiz_3.md)
- **Full-Length Mock Exam Simulations**
  - [Practice Exams Overview](./Practice_Exams/README.md)
  - [Practice Exam 1: Complete 24-Hour Enterprise AD + 3 Standalone Simulation](./Practice_Exams/Practice_Exam_1.md)
  - [Practice Exam 2: Multi-Pivot Hybrid AD Forest & Hardened Standalone Simulation](./Practice_Exams/Practice_Exam_2.md)
  - [Practice Exam 3: Web-Heavy Initial Access, ADCS Escalation & Container Escape Simulation](./Practice_Exams/Practice_Exam_3.md)
