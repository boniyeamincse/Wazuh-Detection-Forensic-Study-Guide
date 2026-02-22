# 05 - Threat Hunting Pack (MITRE & Behavioral) 🎯

This document provides specialized queries for proactive threat hunting, mapped to the **MITRE ATT&CK** framework and focusing on behavioral anomalies.

---

## 🛡️ MITRE ATT&CK Mapping

### 1. Initial Access (TA0001)
- **T1190 - Exploit Public-Facing Application:** `rule.groups: "web" AND rule.level: > 10`
- **T1078 - Valid Accounts:** `rule.id: 60106 AND data.win.eventdata.targetUserName: ("Guest" OR "Admin" OR "Root")`
- **T1133 - External Remote Services:** `rule.id: 60106 AND NOT data.win.eventdata.ipAddress: "192.168.*"`

### 2. Persistence (TA0003)
- **T1543 - Create or Modify System Process:** `rule.id: (60114 OR 5402)`
- **T1547 - Boot or Logon Autostart Execution:** `data.win.eventdata.targetObject: "*\\CurrentVersion\\Run*"`
- **T1053 - Scheduled Task/Job:** `rule.id: 61623`

### 3. Privilege Escalation (TA0004)
- **T1548 - Abuse Elevation Control Mechanism:** `rule.id: 5402 AND data.linux.command: "*NOPASSWD*"`
- **T1068 - Exploitation for Privilege Escalation:** `rule.id: 80710` (Suid binary execution)

### 4. Defense Evasion (TA0005)
- **T1070 - Indicator Removal on Host:** `rule.id: (60132 OR 5402) AND data.linux.command: "history -c"`
- **T1218 - System Binary Proxy Execution (Lolbins):** `data.win.eventdata.image: ("*\\mshta.exe" OR "*\\regsvr32.exe" OR "*\\certutil.exe")`

### 5. Credential Access (TA0006)
- **T1003 - OS Credential Dumping:** `data.win.eventdata.targetImage: "*\\lsass.exe" AND rule.id: 61612` (Sysmon 10)
- **T1555 - Credentials from Web Browsers:** `data.win.eventdata.path: "*\\Login Data"`

---

## 📈 Behavioral & Anomaly Hunting

### 1. Rare Process Execution
Hunt for processes that are not part of the standard baseline.
- **Query:** `NOT data.win.eventdata.image: ("*\\chrome.exe" OR "*\\svchost.exe" OR "*\\explorer.exe")`

### 2. Off-Hours Login Detection
(This requires using the `timestamp` field in an aggregation or a script query in DSL).
- **Pattern:** `rule.id: 60106 AND NOT (timestamp: [09:00:00 TO 18:00:00])`

### 3. Geographic Anomaly Detection
Detect logins from countries where your company does not have offices.
- **Filter:** `rule.id: 5715 AND NOT data.geoip.country_name: "United States"`

### 4. High Volume DNS Queries (Exfiltration/C2)
- **Pattern:** `rule.id: 61619` (Aggregation of DNS queries over time)

---

## 🛠️ Automated Hunting Playbook

1.  **Select a Tactic:** (e.g., Persistence).
2.  **Apply Discovery Filter:** Use KQL to find all `rule.id: 61623` (Scheduled Task).
3.  **Analyze Command Lines:** Look for `powershell`, `cmd`, or `curl` inside the task.
4.  **Pivot to Agent:** Use `agent.name` to see all other activity on that host within +/- 30 minutes.

### 📍 Hunt 3: Impact & Resource Development (Advanced)

These tactics focus on how attackers disrupt operations or build infrastructure for future attacks.

| Technique | Intent | Wazuh Query / Hunting Logic |
| :--- | :--- | :--- |
| **T1485** | Data Destruction | `data.win.eventdata.image: ("vssadmin.exe" OR "wbadmin.exe") AND data.win.eventdata.commandLine: "*delete*"` |
| **T1490** | Inhibit Recovery | `data.win.eventdata.image: "bcdedit.exe" AND data.win.eventdata.commandLine: "*ignoreallfailures*"` |
| **T1491** | Defacement | `data.file: "*" AND (data.file: "*.html" OR data.file: "*.php") AND NOT data.user: "www-data"` |
| **T1583** | Acquire Infrastructure | `data.aws.eventName: "PurchaseReservedInstancesOffering" OR data.aws.eventName: "RequestSpotInstances"` |
| **T1588** | Obtain Capabilities | `data.win.eventdata.image: ("rclone.exe" OR "mega.exe") AND data.win.eventdata.commandLine: "*config*"` |

---

## 📈 Hunting Tip: Time-Series Anomalies
---

**Previous: [04 - OpenSearch Query DSL](04-opensearch-dsl.md)** | **Next: [06 - SOC Playbooks](06-soc-playbooks.md)**

[Return to Index](../../README.md)
