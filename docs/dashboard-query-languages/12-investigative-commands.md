# 12 - Investigative OS Commands Library 🛠️

This library provides a curated list of commands that are commonly used by attackers. Searching for these in your Wazuh logs is a primary method for proactive threat hunting.

---

## 🔍 Discovery Actions (TA0007)
Attackers use these commands to understand their environment after initial access.

### Windows
| Command | Hunt Query (KQL) | Intent |
| :--- | :--- | :--- |
| `whoami /priv` | `data.win.eventdata.commandLine: "*whoami /priv*"` | Check user privileges |
| `net view /domain` | `data.win.eventdata.commandLine: "*net view /domain*"` | Enumerate domain resources |
| `nltest /domain_trusts` | `data.win.eventdata.commandLine: "*nltest /domain_trusts*"` | Find trusted domains |
| `systeminfo` | `data.win.eventdata.commandLine: "*systeminfo*"` | Get OS/Patch details |
| `quser` | `data.win.eventdata.commandLine: "*quser*"` | See logged-in users |

### Linux
| Command | Hunt Query (KQL) | Intent |
| :--- | :--- | :--- |
| `uname -a` | `data.linux.command: "*uname -a*"` | Get kernel version (exploit prep) |
| `id` | `data.linux.command: "id"` | Check current UID/GID |
| `cat /etc/passwd` | `data.linux.command: "*cat /etc/passwd*"` | Enumerate system users |
| `find / -perm -4000` | `data.linux.command: "*find * -perm -4000*"` | Search for SUID binaries |

---

## 🏃 Execution & C2 (TA0002 / TA0011)
Commands used to run malicious payloads or download tools.

### Malware Download
- **Windows (Certutil):** `data.win.eventdata.commandLine: "*certutil -urlcache -f*"`
- **Linux (Curl/Wget):** `data.linux.command: ("*curl * -o *" OR "*wget * -O *")`

### Reverse Shells
- **Python:** `data.linux.command: "*python* -c *import socket*"`
- **Netcat:** `data.linux.command: "*nc* -e */bin/sh*"`
- **PowerShell:** `data.win.eventdata.commandLine: "*New-Object System.Net.Sockets.TCPClient*"`

---

## 🛡️ Defense Evasion (TA0005)
Commands used to hide activity or disable security tools.

### Log & History Cleaning
- **Clear Event Logs:** `rule.id: 60132` (Windows)
- **Clear Bash History:** `data.linux.command: "history -c"` or `rm .bash_history`

### Disabling Security
- **Disable Win Defender:** `data.win.eventdata.commandLine: "*Set-MpPreference -DisableRealtimeMonitoring $true*"`
- **Disable SELinux:** `data.linux.command: "setenforce 0"`
- **Disable Firewall:** `data.linux.command: "ufw disable"` or `iptables -F`

---

## 🔑 Credential Access (TA0006)
Hunting for password theft and credential dumping.

- **Mimikatz Pattern:** `data.win.eventdata.commandLine: ("*sekurlsa*" OR "*logonpasswords*")`
- **Searching for passwords in files:** `data.linux.command: "*grep -r password*"`
- **LSASS Access:** `data.win.eventdata.targetImage: "*\\lsass.exe"` (Sysmon Type 10)

---

## 🚀 Pro-Tip: The "Command Chain" Hunt
Instead of searching for a single command, search for a **sequence**. An attacker usually runs `whoami`, then `systeminfo`, then `net user` within 1-2 minutes.
**DSL Correlation:**
```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "data.win.eventdata.commandLine": "whoami" } },
        { "match": { "agent.id": "001" } }
      ]
    }
  }
}
```
*(Search for one, then pivot to the time window to find the others).*

---

## 📤 Exfiltration & Data Transfer

Hunting for the "Final Stage" of an attack.

| Tool / Intent | Wazuh Query / Hunting Logic |
| :--- | :--- |
| **Rclone (S3/Cloud)** | `data.win.eventdata.image: "*rclone.exe" AND data.win.eventdata.commandLine: "*copy*"` |
| **Mega.nz Upload** | `data.win.eventdata.image: "*MEGAcmd.exe" OR data.win.eventdata.image: "*MEGAsync.exe"` |
| **Certutil Download** | `data.win.eventdata.image: "certutil.exe" AND data.win.eventdata.commandLine: "*-urlcache*"` |
| **Ingress Tool** | `data.win.eventdata.image: "bitsadmin.exe" AND data.win.eventdata.commandLine: "*/transfer*"` |

## 🛠️ Advanced Recon & Discovery

| Command | Intent | Wazuh Query |
| :--- | :--- | :--- |
| **Adfind.exe** | AD Recon | `data.win.eventdata.image: "*adfind.exe"` |
| **BloodHound** | AD Pathfinding | `data.win.eventdata.image: "*SharpHound.exe"` |
| **Whoami /all** | Token Discovery | `data.win.eventdata.commandLine: "whoami /all"` |
| **Net View** | Network Share Recon | `data.win.eventdata.commandLine: "net view *"` |

---

**Previous: [11 - Log Types Dictionary](11-log-types-dictionary.md)** | **Next: [13 - Lab Setup Guide](13-lab-setup.md)**

[Return to Index](../../README.md)
