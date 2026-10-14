# 🛡️ CyberDefenders Write-Ups

<p align="center">
  <img src="https://img.shields.io/badge/Platform-CyberDefenders-0078D4?style=for-the-badge&logo=shield" alt="CyberDefenders" />
  <img src="https://img.shields.io/badge/Focus-DFIR%20%7C%20SOC%20%7C%20Blue%20Team-blueviolet?style=for-the-badge" alt="Focus" />
  <img src="https://img.shields.io/badge/Author-thanhhao06%20(Azaki)-orange?style=for-the-badge&logo=github" alt="Author" />
</p>

---

## 📌 Overview

This repository serves as a centralized collection of comprehensive **Write-Ups and Digital Forensics & Incident Response (DFIR)** investigation reports for blue team challenges on the **[CyberDefenders](https://cyberdefenders.org/)** platform.

The primary goals of this repository are to:
- Document methodical investigation workflows, investigative mindsets, and digital evidence triage.
- Detail artifact extraction techniques across system logs, memory dumps, network captures, disk images, and malware samples.
- Provide a clean, structured reference with step-by-step walkthroughs and visual evidence for cybersecurity research and continuous learning.

---

## 📂 Repository Structure

Each lab is structured as an independent module, making it easy to navigate and scale as new write-ups are added over time:

```text
Cyberdefenders-Write-Up/
├── <Lab Name> Lab/
│   ├── <Lab Name> Lab.md       # Detailed report, investigative questions & analysis
│   └── images/                 # Relevant screenshots, forensic evidence & artifacts
└── README.md                   # Repository overview and documentation
```

---

## 🔍 Core Domains Covered

The investigations in this repository span a wide spectrum of defensive security disciplines:

### 1. Endpoint Forensics
- **Windows Artifact Analysis**: NTFS artifacts ($MFT, $LogFile, $UsnJrnl), Registry hives, Prefetch, ShimCache, Amcache, ShellBags, Jump Lists, and SRUM.
- **Log Analysis**: Windows Security Event Logs, PowerShell audit logs (ScriptBlock & transcription), and Sysmon telemetry.
- **Incident Investigation**: Tracing privilege escalation, persistence mechanisms, abnormal process execution, and lateral movement.

### 2. Network Forensics
- **Traffic Analysis**: Deep packet inspection (PCAP/PCAPNG) using Wireshark, Tshark, NetworkMiner, and Zeek.
- **Threat Detection**: Identification of malicious command-and-control (C2) beaconing, suspicious TLS/SSL handshakes, protocol abuse, brute-force attempts, and DNS data exfiltration.

### 3. Threat Intelligence & OSINT
- **Open-Source Intelligence**: Pivot-based investigation using public threat feeds, passive DNS, WHOIS records, and search dorks.
- **Adversary Tracking**: Extracting indicators of compromise (IOCs), identifying threat actor infrastructure, and mapping adversary behavior to the MITRE ATT&CK framework.

### 4. Malware Analysis & Triage
- **Initial Vector Triage**: Static and dynamic analysis of weaponized Office documents, malicious XLM macros, LNK shortcuts, and malicious PDFs.
- **Payload Extraction**: Deobfuscating loaders, unpacking payloads, and extracting C2 server configurations from infostealers and Remote Access Trojans (RATs).

### 5. Memory Forensics
- **Volatile Memory Triage**: Analyzing Windows memory dumps using Volatility 2 and Volatility 3 frameworks.
- **In-Memory Threat Detection**: Spotting process hollowing, DLL injection, API hooking, orphaned processes, and dumping injected binaries and credentials.

---

## 🛠️ Tooling & Arsenal

A variety of industry-standard DFIR tools and utilities are utilized across these write-ups:

| Category | Tools & Technologies |
| :--- | :--- |
| **Endpoint & Triage** | Eric Zimmerman's Tools (EZ Tools: MFTECmd, Registry Explorer, Timeline Explorer, JLECmd), KAPE, Hayabusa, Chainsaw, FTK Imager, Autopsy |
| **Network Analysis** | Wireshark, Tshark, NetworkMiner, Zeek, Brim / Zui, Snort, Suricata |
| **Memory Analysis** | Volatility 2, Volatility 3, MemProcFS |
| **Malware & Deobfuscation** | CyberChef, pestudio, FLOSS, Ghidra, x64dbg, YARA |
| **Threat Intelligence** | VirusTotal, AlienVault OTX, Any.Run, Hybrid Analysis, URLhaus, AbuseIPDB |

---

## ⚠️ Disclaimer

- All write-ups and analyses provided in this repository are strictly for **educational, academic research, and non-commercial security training purposes**.
- Some challenges on the CyberDefenders platform remain active. Practitioners and learners are strongly encouraged to attempt the labs independently before consulting these write-ups.

---

<p align="center">
  <i>Maintained with ❤️ by <a href="https://github.com/thanhhao06">thanhhao06 (Azaki)</a></i>
</p>
