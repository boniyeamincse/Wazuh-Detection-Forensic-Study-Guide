# 03 - Lucene Advanced Guide 🦾

While KQL is preferred for daily tasks, **Lucene** remains essential for detection engineers who need the power of **Regular Expressions (Regex)**, **Fuzzy matching**, and **Proximity searching**.

---

## 🔍 Lucene Power Syntax

### 1. Regular Expressions (Regex)
Lucene supports regex wrapped in forward slashes `/`.
- **Match all 'sh' processes:** `data.linux.process: /.*sh/`
- **Suspicious hex in command line:** `data.win.eventdata.commandLine: /.*[0-9a-fA-F]{32}.*/`

### 2. Fuzzy Searching
Useful for detecting typosquatting or variations in process names. Uses `~`.
- **Typosquatting detection:** `data.win.eventdata.image: (prowershell~ or powelshell~)`

### 3. Proximity Searching
Finds words within a specific distance of each other. Uses `~` with a number.
- **Find 'net' and 'user' within 2 words:** `data.win.eventdata.commandLine: "net user"~2`

### 4. Wildcards
- `*`: Multiple characters.
- `?`: Single character. `agent.name: "web-ser?er"`

---

## 🚀 100+ Lucene Detection Queries

### Windows Power Queries
1.  **Regex: Detecting Base64-like strings (40+ chars):** `data.win.eventdata.commandLine: /[a-zA-Z0-9+\/]{40,}/`
2.  **Fuzzy: Mimikatz evasion:** `data.win.eventdata.commandLine: (mimikatz~ OR mamikatz~)`
3.  **Regex: Internal IP in Command Line:** `data.win.eventdata.commandLine: /.*10\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}.*/`
4.  **Proximity: Net user add logic:** `data.win.eventdata.commandLine: "net user /add"~3`
5.  **Regex: Suspicious temp file execution:** `data.win.eventdata.image: /C:\\Windows\\Temp\\.*\.exe/`
6.  **Boosting: Prioritize Critical Agents:** `agent.name: "DomainController"^5 OR agent.name: "WebSrv"`
7.  **Regex: Domain admin enumeration:** `data.win.eventdata.commandLine: /net.*group.*"domain admins".*/`
8.  **Regex: Schtasks persistence:** `data.win.eventdata.commandLine: /schtasks.*\/create.*\/tn.*/`
9.  **Fuzzy: Psexec variation:** `data.win.eventdata.serviceName: psexecsvc~`
10. **Regex: Windows Defender Exclusions:** `data.win.eventdata.commandLine: /.*Add-MpPreference.*-ExclusionPath.*/`

### Linux Regex & Advanced Queries
11. **Regex: Hidden file execution in /tmp:** `data.linux.command: /\/tmp\/\..*/`
12. **Regex: Web Shell (.php in upload folder):** `data.linux.path: /\/var\/www\/html\/uploads\/.*\.php/`
13. **Proximity: Sudoers permission change:** `data.linux.command: "chmod 777 sudoers"~2`
14. **Regex: Python reverse shell pattern:** `data.linux.command: /python.*-c.*import.*socket.*/`
15. **Regex: SSH login from non-standard port in logs:** `full_log: /.*sshd.*port [0-9]{4,5}.*/`

### Web & Network Deep Dive
16. **Regex: SQLi SLEEP function:** `full_log: /.*SELECT.*SLEEP\(.*\).*/`
17. **Regex: Credit Card Number Leak (DLP):** `full_log: /[45][0-9]{3}-[0-9]{4}-[0-9]{4}-[0-9]{4}/`
18. **Regex: Path Traversal deep:** `full_log: /(\.\.\/){3,}/`
19. **Regex: Suspicious User-Agent (Nmap/Sqlmap):** `data.user_agent: /(nmap|sqlmap|nikto).*/`

---

## 🧬 Dual-Binary Proximity Hunting

Advanced attackers often use two separate binaries in quick succession (e.g., `cmd.exe` followed by `bitsadmin.exe`). Lucene's proximity operator can hunt for these patterns in the `full_log` or aggregated command fields.

- **Hunt for 'cmd' and 'bitsadmin' within 5 words:**
  `data.win.eventdata.commandLine: "cmd bitsadmin"~5`
- **Hunt for 'python' and 'socket' in reverse shell attempts:**
  `data.linux.command: "python socket"~3`
- **Hunt for 'certutil' and 'urlcache' proximity:**
  `data.win.eventdata.commandLine: "certutil urlcache"~2`

---

## ⚠️ Performance Caveats

> [!WARNING]
> **Regex queries are CPU-intensive.**
> - Never use a regex that starts with a wildcard like `/.*/` if possible.
> - Limit regex queries to a short time window.
> - Avoid using regex on `text` fields; always use `.keyword` counterparts.

---

## 🛠️ One-Page Cheat (Lucene vs KQL)

| Feature | Lucene | KQL |
| :--- | :--- | :--- |
| **Regex** | `field: /regex/` | Not Supported (use DSL) |
| **Fuzzy** | `field: value~` | Not Supported |
| **Proximity** | `"word1 word2"~5` | Not Supported |
| **Boosting** | `field: val^2` | Not Supported |
| **Auto-complete** | Minimal | High |

---

**Previous: [02 - KQL Complete Master Guide](02-kql-master.md)** | **Next: [04 - OpenSearch Query DSL](04-opensearch-dsl.md)**

[Return to Index](../../README.md)
