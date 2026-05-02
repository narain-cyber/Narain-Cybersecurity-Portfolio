# Lab Report: Active Directory Identity Escalation Path

**Author:** Dylan Matthew Narain
**Date:** May 2, 2026

---

## 1. Executive Summary
This report documents a successful identity-based attack path originating from a low-privileged user account. [Description of findings...]

## 2. Phase I: Reconnaissance & Enumeration
* **Tooling**: Utilized **SharpHound.exe** for data collection.
* **Execution**: `.\SharpHound.exe -c All --domain narain.lab`

![SharpHound Execution](Sharphound_Running_Win_Server_2022.png)
![SharpHound Output](SharpHound_File_Location.jpg)

## 3. Phase II: Attack Path Analysis (BloodHound)
Data was analyzed using **BloodHound Community Edition**.

![Attack Graph](d.student-DA.png)

## 4. Phase III: Kerberoasting
* **The Vulnerability**: The account possessed an SPN.
* **Execution**: Using `impacket-GetUserSPNs`.

![Kerberoasting Results](golden_ticket_d.student.jpg)

## 5. Phase IV: Offline Password Cracking
* **Tooling**: **Hashcat** mode `13100`.
* **Result**: Password cracked: `Test123Test`.

![Hashcat Success](hashcat-crack.jpg)
