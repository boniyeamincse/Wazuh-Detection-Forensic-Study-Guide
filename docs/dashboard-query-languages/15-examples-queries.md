# The Daily SOC Hunter's Checklist ⚡

A lean, high-speed reference for common "Triage" and "Hunting" activities. Copy and paste these directly into the Wazuh Dashboard search bar.

## 🚨 Tier 1: Critical Triage (First 15 Mins)

| Objective | KQL Query |
| :--- | :--- |
| **All Level 12+ Alerts** | `rule.level: >= 12` |
| **Successful Logins (Admin)** | `rule.id: 60106 AND data.win.eventdata.targetUserName: "Administrator"` |
| **Failed Logins (Spike)** | `rule.id: 60122 AND rule.level: > 10` |
| **New User Created** | `rule.id: (60113 OR 5710)` |
| **Defender Disabled** | `data.win.eventdata.targetObject: "*DisableRealtimeMonitoring*" AND data.win.eventdata.details: "0x00000001"` |

## 🔍 Tier 2: Behavioral Hunting (Deep Dive)

| Objective | Lucene / Regex Query |
| :--- | :--- |
| **Suspicious Powershell** | `data.win.eventdata.image: "*powershell.exe" AND data.win.eventdata.commandLine: /.*-e(nc)? .*/` |
| **LOLBIN Execution** | `data.win.eventdata.image: ("*certutil.exe" OR "*mshta.exe" OR "*regsvr32.exe")` |
| **Network Discovery** | `data.win.eventdata.commandLine: ("*net view*" OR "*nltest /domain_trusts*")` |
| **Data Exfiltration** | `data.win.eventdata.image: ("*rclone.exe" OR "*mega.exe")` |

## ☸️ Tier 3: Cloud & Container Ops

| Objective | KQL Query |
| :--- | :--- |
| **K8s Secret Access** | `data.k8s.objectRef.resource: "secrets" AND data.k8s.verb: "get"` |
| **Azure Login Fail** | `data.azure.operationName: "Sign-in activity" AND data.azure.properties.status.errorCode: 50126` |
| **AWS IAM Changes** | `data.aws.eventName: ("CreateUser" OR "PutUserPolicy")` |

---

## 🚀 Speed Tip
Save your most frequent queries using the **"Save"** button at the top of the Discover tab. Pin them to your "Morning Triage" dashboard for 1-click visibility.

---

**Previous: [14 - Learning Resources](14-learning-resources.md)** | **Next: [16 - Windows Auth & Lateral Movement](16-rdp-smb-windows-auth.md)**

[Return to Index](../../README.md)
