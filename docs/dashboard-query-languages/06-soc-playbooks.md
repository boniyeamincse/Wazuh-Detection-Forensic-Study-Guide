# 06 - SOC Playbooks & Investigation Workflows 📋

This document outlines the investigation workflows for different SOC tiers and provides detailed playbooks for real-world attack scenarios using Wazuh Dashboard queries.

---

## 🕵️ Investigation Workflows

### Tier 1: Alert Triage (Initial Analysis)
**Goal:** Determine if an alert is a True Positive (TP) or False Positive (FP).
1.  **Check Rule Level & Metadata:** `rule.id` and `rule.description`.
2.  **Verify Source IP Reputation:** `data.srcip`.
3.  **Basic Context Query:** `agent.id: "XYZ" AND timestamp: [now-1h TO now]`.
4.  **Escalation:** If TP and severity > 7, escalate to Tier 2.

### Tier 2: Incident Response (Deep Dive)
**Goal:** Understand the scope of the compromise.
1.  **Process Tree Analysis:** Use `data.win.eventdata.parentImage` and `data.win.eventdata.image`.
2.  **Network Connection Audit:** `data.win.eventdata.destinationIp`.
3.  **Lateral Movement Check:** Check for 4624 (Logon) events from the compromised host to others.

### Tier 3: Threat Hunting & Forensics
**Goal:** Proactive discovery of hidden threats.
1.  **Behavioral Analysis:** Use aggregations to find rare processes.
2.  **Time-Series Correlation:** Link web logs, sysmon, and firewall logs using DSL.

---

## 📖 Scenario Walkthroughs

### 1. Ransomware Detection (LockBit/Wannacry)
- **Indicator 1: Shadow Copy Deletion**
  `data.win.eventdata.commandLine: "*vssadmin delete shadows*"`
- **Indicator 2: Mass File Renaming**
  `rule.id: 61611 AND data.win.eventdata.image: "*tasksche.exe"`
- **Indicator 3: Connection to C2**
  `rule.id: 61603 AND data.win.eventdata.destinationIp: [Malicious_IP_List]`

### 2. Domain Compromise (Golden Ticket/DCSync)
- **Detection:** Use `rule.id: 60143` (Potential Golden Ticket).
- **Investigation:** Check for unusual Kerberos ticket requests.
  `data.win.eventdata.serviceName: "krbtgt" AND NOT data.win.eventdata.ipAddress: [DC_IP_List]`

### 3. Linux Web Breach (Web Shell)
- **Detection:** `.php` file execution from an upload directory.
  `data.linux.path: /\/var\/www\/html\/uploads\/.*\.php/` (Lucene Regex)
- **Lateral Movement:** Check for netcat or python reverse shells.
  `data.linux.command: ("*nc -e*" OR "*python -c*")`

### 4. Insider Threat (Data Exfiltration)
- **Detection:** Massive file copying to USB or Cloud Storage.
  `rule.id: 61611 AND (data.win.eventdata.path: "*\\DropBox\\*" OR data.win.eventdata.path: "*\\Google Drive\\*")`

---

## 🛠️ Red Team Detection Validation
To validate your SOC's visibility, run these "canary" commands and ensure they trigger alerts:
1.  **Windows:** `whoami /priv`
2.  **Linux:** `curl http://canarytokens.com/check/xyz`
3.  **AD:** `net group "Domain Admins" /domain`

---

## 🚀 Pro-Tip: The "Pivot" Workflow
When you find a suspicious process ID (`data.win.eventdata.processId`), immediately add a filter for that ID. You will see every action that specific process took, creating a clear timeline of the attacker's actions.
