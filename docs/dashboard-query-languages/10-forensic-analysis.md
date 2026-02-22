# 10 - Deep Forensic Querying & Investigation 🔍

This module focuses on using Wazuh as a forensic tool to uncover deep system artifacts, hidden persistence, and complex network behavior.

---

## 🛠️ Windows Forensics

### 1. Persistence Mechanisms (Registry)
Attackers often hide in the registry to survive reboots.
- **Run/RunOnce Keys:**
  `data.win.eventdata.targetObject: "*\\Microsoft\\Windows\\CurrentVersion\\Run*"`
- **Startup Folder Execution:**
  `data.win.eventdata.path: "*\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\*"`
- **Service Creation (Event 7045):**
  `rule.id: 60114`

### 2. WMI Event Forensics
WMI is a common way to execute code stealthily.
- **WMI Permanent Event Consumer:**
  `data.win.eventdata.image: "*\\WmiPrvSE.exe" AND NOT data.win.eventdata.parentImage: "*\\services.exe"`

### 3. Registry Forensics (Value Modification)
- **Sticky Keys Backdoor:**
  `data.win.eventdata.targetObject: "*\\Microsoft\\Windows NT\\CurrentVersion\\Image File Execution Options\\sethc.exe"`

---

## 🐧 Linux Forensics

### 1. File Integrity Monitoring (FIM)
Wazuh's FIM is powerful for forensic analysis of altered binaries.
- **Modification of /bin or /usr/bin:**
  `rule.id: 550 AND data.file: ("/bin/*" OR "/usr/bin/*")`

### 2. Suspicious Kernel Space Activity
- **Module Injection (LKM):**
  `rule.id: 80707 AND data.linux.command: "*insmod*"`

### 3. Bash Forensics
- **Command Execution via Subshell:**
  `data.linux.command: "*$(*)*" OR data.linux.command: "*`*`*"`

---

## 🌐 Network Flow Analysis

### 1. Beaconing Detection
Hunt for periodic connections to the same IP.
- **KQL Pattern:**
  `rule.id: 61603 AND data.win.eventdata.destinationIp: "External_IP" AND timestamp: [now-1h TO now]`
  *(Use aggregations on `timestamp` to see frequency and intervals).*

### 2. DNS Tunneling
- **Large DNS Answer Sizes:**
  `rule.id: 61619 AND data.win.eventdata.queryName: "*.[a-zA-Z0-9]{50,}.*"`

---

## 🧬 Process Lineage Hunting

To understand the **Root Cause**, you must follow the process parent-child relationship.

**The "Screwdriver" Query (Pivot):**
If `data.win.eventdata.processId: "1234"` is malicious, search for its parent:
`data.win.eventdata.processId: "1234" AND data.win.eventdata.parentProcessId: *`

Then pivot to that parent ID to find out who started the malicious process (e.g., was it `explorer.exe` or `powershell.exe`?).

---

## 🚀 Pro-Tip: The "Time Context" Sweep
Once you find a forensic artifact (e.g., a file arrival at 10:05 AM), run this query to see EVERYTHING that happened on that host within a 2-minute window:
`agent.id: "001" AND timestamp: ["2024-02-22T10:04:00Z" TO "2024-02-22T10:07:00Z"]`
This reveals the attacker's "footprints" before and after the specific event.

---

**Previous: [09 - Quick Reference](09-quick-reference.md)** | **Next: [11 - Log Types Dictionary](11-log-types-dictionary.md)**

[Return to Index](../../README.md)
