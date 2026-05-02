# Lab Report: Active Directory Identity Escalation Path

**Author:** Dylan Matthew Narain
**Date:** May 2, 2026
**Environment:** `narain.lab` (Isolated Research Environment)

---

## 1. Executive Summary
This report documents a successful identity-based attack path originating from a low-privileged user account. By exploiting insecure Active Directory permissions and leveraging a Kerberoasting attack, I achieved a total forest compromise. This lab highlights the critical risk of "Shadow Admins"—accounts that possess administrative rights over high-tier objects without being members of protected groups.

## 2. Lab Scenario & Background
The objective of this lab was to simulate a "Common Entry" breach. 
*   **Initial Access**: A low-privileged account (`d.student`) was compromised via simulated phishing.
*   **Environment**: A Windows Server 2022 Domain Controller managing a research-focused forest.
*   **Goal**: Escalate privileges to the `Domain Admins` group.

## 3. Phase I: Reconnaissance & Enumeration
The engagement began with mapping the internal Active Directory environment to identify misconfigurations and trust relationships.

### Steps:
1.  Transferred the **SharpHound** collector to the target workstation.
2.  Executed the collector to gather all domain data.
    ```powershell
    .\SharpHound.exe -c All --domain narain.lab
    ```

![SharpHound Execution](Narain-Cybersecurity-Portfolio/AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/Sharphound_Running_Win_Server_2022.png)
![SharpHound Artifacts](Narain-Cybersecurity-Portfolio/AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/SharpHound_File_Location.jpg)

## 4. Phase II: Attack Path Analysis
The collected data was ingested into **BloodHound** to visualize the attack surface.

### Steps:
1.  Started the **Neo4j** database and BloodHound interface.
2.  Uploaded the SharpHound zip file.
3.  Mapped the path from `d.student` to the `Domain Admins` group.

**Findings:**
The analysis revealed that `d.student` had **GenericAll** rights over `svc_sql_research`, which in turn had **GenericAll** rights over the `Administrator` account.

![BloodHound Setup](Narain-Cybersecurity-Portfolio/AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/Neo4j_startup_kali.jpg)
![Data Upload](Narain-Cybersecurity-Portfolio/AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/bloodhound_zip_upload.png)
![Visualized Path](Narain-Cybersecurity-Portfolio/AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/d.student-DA.png)

## 5. Phase III: Kerberoasting (Credential Harvesting)
Since `svc_sql_research` was a service account with an SPN, it was targeted to retrieve a crackable hash.

### Steps:
1.  Used `impacket-GetUserSPNs` to identify and request a TGS-REP for the service account.
2.  The resulting hash was saved for offline cracking.

![Kerberoasting](Narain-Cybersecurity-Portfolio/AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/golden_ticket_d.student.jpg)

## 6. Phase IV: Offline Password Cracking & Validation
The final stage involved recovering the plaintext password to execute the privilege escalation.

### Steps:
1.  Utilized **Hashcat** with the Kerberos 5 etype 23 module.
    ```bash
    hashcat -m 13100 svc_hash.txt password_list.txt
    ```
2.  The password was cracked: **`Test123Test`**.
3.  Validated the credentials using `netexec` to confirm administrative access.

![Hashcat Crack](Narain-Cybersecurity-Portfolio/AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/hashcat-crack.jpg)
![Credential Validation](Narain-Cybersecurity-Portfolio/AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/netexec_svc_acc.jpg)

---

## 7. Remediation Recommendations
*   **Implement Tiered Administration**: Ensure service accounts cannot manage Tier-0 objects (Domain Admins).
*   **Enforce Managed Service Accounts (gMSA)**: These accounts use complex, automatically rotated passwords that are resistant to Kerberoasting.
*   **Audit Active Directory Permissions**: Regularly run BloodHound to identify and revoke unintended **GenericAll** or **WriteDacl** permissions.
