# 🔒 Endpoint Security Hardening & Compliance Automation

[![Benchmark](https://img.shields.io/badge/Benchmark-CIS%20Win11%20v3.0.0-red?style=flat)](https://www.cisecurity.org/)
[![SIEM](https://img.shields.io/badge/SIEM-Wazuh-005571?style=flat)](https://wazuh.com/)
[![Compliance](https://img.shields.io/badge/Final%20Score-92%25-brightgreen?style=flat)]()

Automated CIS Microsoft Windows 11 Enterprise Benchmark v3.0.0 hardening using PowerShell — from **0% to 92% compliance** measured via Wazuh SCA scans.

📝 [Full write-up with screenshots on Medium →](https://medium.com/@joshiyash1507/from-0-to-92-how-i-hardened-a-windows-11-endpoint-against-the-cis-benchmark-using-wazuh-and-32e7a0b0067b)

---

## 📑 Table of Contents

- [Lab Architecture](#lab-architecture)
- [What This Project Does](#what-this-project-does)
- [Results](#results)
- [Folder Structure](#folder-structure)
- [How to Use the Scripts](#how-to-use-the-scripts)
- [Remaining Failures Explained](#remaining-failures-explained)
- [Disclaimer](#disclaimer)

---

## 🏗️ Lab Architecture

| Machine | Role | Platform |
|---|---|---|
| Windows 11 Enterprise VM | Hardening target | VirtualBox |
| Kali Linux VM | Supporting / attack simulation | VirtualBox |
| Windows 11 (Splunk host) | Log aggregation | VirtualBox |
| Windows Server 2022 | Active Directory | VirtualBox |

> Target machine is **standalone/workgroup** — not domain-joined.

---

## 🎯 What This Project Does

1. Runs a **CIS Benchmark SCA scan** via Wazuh on a standalone Windows 11 Enterprise VM
2. Exports failing checks to CSV for structured analysis
3. Applies two rounds of **PowerShell remediation scripts** targeting:
   - Account policies (secedit)
   - LSA, NTLM, UAC, SMB hardening (registry)
   - Windows Firewall (netsh + registry)
   - Audit policies (auditpol)
   - Defender/ASR, BitLocker policy, RDS, PowerShell transcription, WinRM, SmartScreen, telemetry, event log sizes
4. Re-scans after each round to measure compliance improvement

---

## 📊 Results

| Stage | Passed | Failed | Score |
|---|---|---|---|
| Baseline | 0 | 348 | 0% |
| After Round 1 (`cis_remediate.ps1`) | 200 | 273 | 42% |
| After Round 2 (`cis_remediate_v2.ps1`) | 437 | 36 | **92%** |

---

## 📁 Folder Structure

```
02-endpoint-hardening/
├── README.md
├── scripts/
│   ├── cis_remediate.ps1          # Round 1 — account, LSA, firewall, audit policies
│   └── cis_remediate_v2.ps1       # Round 2 — registry, Defender, BitLocker, WinRM, etc.
└── reports/
    ├── checks_before_remediation.csv           # Wazuh SCA export — baseline
    └── checks_after_remediation.csv            # Wazuh SCA export — post-remediation
```

> 📸 All screenshots are in the [Medium write-up](https://medium.com/@joshiyash1507/from-0-to-92-how-i-hardened-a-windows-11-endpoint-against-the-cis-benchmark-using-wazuh-and-32e7a0b0067b)

---

## ⚙️ How to Use the Scripts

> ⚠️ Run on a **lab/test VM only** — not for production systems.

**Prerequisites:** Windows 11 Enterprise VM · PowerShell 5.1+ · Run as Administrator

```powershell
# Round 1
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
.\scripts\cis_remediate.ps1

# Reboot, then Round 2
.\scripts\cis_remediate_v2.ps1
```

Each script logs its actions to a `.log` file in the same directory.

---

## ❗ Remaining Failures Explained

The 36 remaining failures are **infrastructure-dependent**, not fixable in a standalone workgroup VM:

| Control | Reason |
|---|---|
| LAPS | Requires Active Directory domain join |
| BitLocker TPM controls | Requires physical TPM chip (unavailable in VirtualBox) |
| Password history (CIS 26000) | Domain policy dependency |

---

## ⚠️ Disclaimer

For lab and learning purposes only. Always test in an isolated environment — applying these controls to production systems without validation can break services.

---

*← [Back to Portfolio](../README.md)*
