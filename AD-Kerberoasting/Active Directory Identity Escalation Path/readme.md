# Lab Report: Active Directory Identity Escalation Path

**Author:** Dylan Narain
**Date:** May 2, 2026
**Focus:** Identity Security, Privilege Escalation, and Kerberoasting
**Environment:** `narain.lab` (Isolated Lab Environment)

---

## 1. Executive Summary
This report documents a successful identity-based attack path originating from a low-privileged user account. By exploiting insecure Active Directory permissions and leveraging a Kerberoasting attack, I achieved a total forest compromise. This lab highlights the critical risk of "Shadow Admins"—accounts that possess administrative rights over high-tier objects without being members of protected groups.

## 2. Lab Scenario & Background
The objective was to simulate a common entry-point breach.
*   **Initial Access**: A low-privileged student-tier account (`d.student`) was compromised.
*   **Target**: Escalate privileges to the `Domain Admins` group.
*   **Significance**: In a higher education environment, a single compromised workstation can become a beachhead for lateral movement if permissions are not strictly tiered.

## 3. Phase I: Reconnaissance & Enumeration
Internal mapping was performed using **SharpHound** to identify misconfigurations and permission-based attack vectors.

**Execution:**
`.\SharpHound.exe -c All --domain narain.lab`

![SharpHound Execution](AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/Sharphound_Running_Win_Server_2022.png)
![SharpHound Artifacts](AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/SharpHound_File_Location.jpg)

## 4. Phase II: Attack Path Analysis
The data was ingested into **BloodHound** to visualize the shortest path to Domain Admin.

**Findings:**
Analysis revealed that `d.student` had **GenericAll** rights over `svc_sql_research`, which in turn held **GenericAll** rights over the `Administrator` account.

![BloodHound Setup](AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/Neo4j_startup_kali.jpg)
![Data Ingestion](AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/bloodhound_zip_upload.png)
![Visualized Path](AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/d.student-DA.png)

## 5. Phase III: Kerberoasting & Cracking
Since `svc_sql_research` was a service account with a Service Principal Name (SPN), it was targeted to retrieve an NTLM hash via Kerberoasting.

**Execution:**
1. Requested a TGS-REP using `impacket-GetUserSPNs`.
2. Utilized **Hashcat** with module `13100` (Kerberos 5, etype 23) for offline cracking.
3. **Result**: Password cracked: `Test123Test`.

![Kerberoasting Results](AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/golden_ticket_d.student.jpg)
![Hashcat Crack](AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/hashcat-crack.jpg)

## 6. Phase IV: Final Validation
Authentication was verified using `netexec` to confirm administrative control over the Domain Controller (192.168.56.10).

![Credential Validation](AD-Kerberoasting/Active%20Directory%20Identity%20Escalation%20Path/screenshots/netexec_svc_acc.jpg)

---

## 7. Strategic Remediation
*   **Tiered Administration**: Isolate Tier-0 (Domain Admin) credentials so they cannot be managed by Tier-1 or Tier-2 accounts.
*   **Least Privilege**: Audit and revoke excessive **GenericAll** permissions on service accounts.
*   **Detection**: Implement SIEM alerts for **Event ID 4769** with RC4 encryption (0x17) to identify Kerberoasting attempts.
