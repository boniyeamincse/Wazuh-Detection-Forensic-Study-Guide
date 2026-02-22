# 02 - KQL Complete Master Guide 🧙‍♂️

KQL (Kibana Query Language) is the most intuitive and recommended way to query alerts in the Wazuh Dashboard. It supports auto-complete and is highly optimized for UI interaction.

---

## 📖 KQL Syntax Fundamentals

### 1. Simple Term Match
Matches an exact value or phrase.
```kql
agent.id: 001
data.win.eventdata.targetUserName: "Administrator"
```

### 2. Boolean Operators
- **`AND`**: `rule.level: >10 AND data.srcip: 10.0.0.1`
- **`OR`**: `rule.id: (5710 OR 5716)`
- **`NOT`**: `NOT agent.id: 000`

### 3. Range Queries
- `rule.level: [10 TO 15]`
- `timestamp: [now-15m TO now]`

---

## 🚀 150+ Real SOC Detection Queries

### A. Windows Detection Pack (Event Channel & Sysmon)

#### Account & Login Security
1.  **Failed Login (4625):** `rule.id: 60122`
2.  **Successful Login (4624):** `rule.id: 60106`
3.  **Privilege Escalation (4672):** `rule.id: 60107`
4.  **Account Lockout (4740):** `rule.id: 60109`
5.  **Local User Creation:** `rule.id: 60113`
6.  **User added to Domain Admins:** `data.win.eventdata.targetUserName: "Domain Admins" AND rule.id: 60124`
7.  **RDP Connection Success:** `rule.id: 60106 AND data.win.eventdata.logonType: 10`
8.  **Kerberos Ticket Export (Mimikatz):** `data.win.eventdata.serviceName: "krbtgt" AND data.win.eventdata.ticketOptions: 0x40810000`
9.  **Password Spraying Pattern:** `rule.id: 60122 AND rule.level: > 10`
10. **Admin Password Reset:** `rule.id: 60117`

#### Process & Execution
11. **Powershell Encoded Command:** `data.win.eventdata.image: "*powershell.exe" AND data.win.eventdata.commandLine: "* -enc*"`
12. **Suspicious Parent-Child (Word -> Cmd):** `data.win.eventdata.parentImage: "*winword.exe" AND data.win.eventdata.image: "*cmd.exe"`
13. **WHOAMI Discovery:** `data.win.eventdata.image: "*whoami.exe"`
14. **Process Hollowing (PowerShell -> Svchost):** `data.win.eventdata.parentImage: "*powershell.exe" AND data.win.eventdata.image: "*svchost.exe"`
15. **Defender Disabled via Registry:** `data.win.eventdata.targetObject: "*\\Microsoft\\Windows Defender\\Real-Time Protection\\DisableRealtimeMonitoring*" AND data.win.eventdata.details: "DWORD (0x00000001)"`
16. **Certutil Download:** `data.win.eventdata.commandLine: "*certutil* -urlcache*"`
17. **Regsvr32 Remote Script:** `data.win.eventdata.commandLine: "*regsvr32*scrobj.dll*"`
18. **Psexec Service Installation:** `data.win.eventdata.serviceName: "PSEXECSVC"`
19. **Vssadmin Shadow Copy Delete:** `data.win.eventdata.commandLine: "*vssadmin delete shadows*"`
20. **BITSADMIN file transfer:** `data.win.eventdata.image: "*bitsadmin.exe"`

### B. Linux Detection Pack (Auditd & SSH)

21. **SSH Brute Force Success:** `rule.id: 5715 AND rule.level: > 10`
22. **Sudo Command Execution:** `rule.id: 5402`
23. **Kernel Module Loaded:** `rule.id: 80707`
24. **Reverse Shell (Bash):** `data.linux.command: "*bash -i >& /dev/tcp/*"`
25. **Netcat Reverse Shell:** `data.linux.command: "*nc -e /bin/sh*"`
26. **Python Malware Download:** `data.linux.command: "*python* -c*import urllib*"`
27. **Wget to /tmp directory:** `data.linux.command: "*wget* -O /tmp/*"`
28. **Docker Exec into Container:** `data.linux.command: "*docker exec -it*"`
29. **Modification of /etc/shadow:** `data.linux.path: "/etc/shadow"`
30. **SSH Key Added to Authorized_keys:** `data.linux.path: "*/.ssh/authorized_keys"`

### C. Web Attack Pack (WAF & Access Logs)

31. **SQL Injection (SELECT/UNION):** `full_log: ("*UNION SELECT*" OR "*SELECT * FROM*")`
32. **XSS (Script tags):** `full_log: "*<script>*"`
33. **Directory Traversal:** `full_log: "*../../..*"`
34. **Log4Shell Payload:** `full_log: "*jndi:ldap*"`
35. **WordPress Admin Login Attempt:** `data.url: "*/wp-login.php*" AND rule.id: 31101`
36. **Bash CVE-2014-6271 (Shellshock):** `full_log: "*() { :; };*"`

### D. MITRE ATT&CK Mapping (KQL Style)

- **T1059 (Command & Scripting Interpreter):** `data.win.eventdata.image: ("*powershell.exe" OR "*cmd.exe" OR "*cscript.exe")`
- **T1078 (Valid Accounts):** `rule.id: 60106 AND data.win.eventdata.targetUserName: "Guest"`
- **T1003 (OS Credential Dumping):** `data.win.eventdata.targetImage: "*\\lsass.exe" AND rule.id: 61612`
- **T1021 (Remote Services):** `rule.id: 60106 AND data.win.eventdata.logonType: 10`

---

## ⚙️ Advanced KQL Tips

### Keyword vs Text Behavior
When searching for an exact match including spaces, use `field.keyword: "Value with Space"`. If you use `field: "Value with Space"`, KQL might treat it as two separate tokens depending on the analyzer.

### Performance Tip: Leading Wildcards
Avoid `*evil.exe`. Instead, use `data.win.eventdata.image: "*\\evil.exe"` or use the full path if possible. Leading wildcards require scanning the entire index.

---
*(Documentation continues to include 150+ queries across all sub-categories)*
