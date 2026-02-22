# 09 - One-Page Quick Reference ⚡

A high-density reference guide for SOC analysts who need rapid lookup of query syntax and detection templates.

---

## 🏎️ Syntax Quick Lookup

| Task | KQL | Lucene | Query DSL (JSON) |
| :--- | :--- | :--- | :--- |
| **Exact Match** | `field: "val"` | `field:"val"` | `"term": {"field": "val"}` |
| **Wildcard** | `field: val*` | `field:val*` | `"wildcard": {"field": "val*"}`|
| **Range** | `f: [1 TO 5]` | `f:[1 TO 5]` | `"range": {"f": {"gte": 1}}` |
| **Exists** | `field: *` | `_exists_:field` | `"exists": {"field": "field"}` |
| **Regex** | (Use DSL) | `field: /regex/`| `"regexp": {"field": ".*"}` |
| **Not** | `NOT field: x` | `-field:x` | `"must_not": [{...}]` |

---

## 🛠️ Boolean Logic Reference

| Operator | Logic | Example (KQL) |
| :--- | :--- | :--- |
| **AND** | Both true | `rule.id: 5710 AND data.srcip: 1.1.1.1` |
| **OR** | Either true | `rule.id: (5710 OR 5716)` |
| **NOT** | Inverse true| `rule.groups: "sshd" AND NOT data.user: "root"` |
| **Group** | Priority | `(rule.id: 123 OR 456) AND agent.id: 001` |

---

---

## 🕵️ SOC Investigation & Checklists

For a lean, high-speed list of copy-paste triage and hunting activities, refer to the specialized modules:
- **[Module 15 - Daily SOC Hunter's Checklist](15-examples-queries.md)**: Tiered triage and hunting.
- **[Module 16 - Windows Auth & Lateral Movement](16-rdp-smb-windows-auth.md)**: Targeted AD/Logon hunts.

---

---

## 📊 Common Performance Rule Mapping

| Field | Type | Search Syntax |
| :--- | :--- | :--- |
| `rule.id` | Keyword | `rule.id: 5710` |
| `agent.name`| Keyword | `agent.name.keyword: "web-01"` |
| `rule.description` | Text | `rule.description: "failed password"` |
| `full_log` | Text | `full_log: "*malicious*"` |

---

## 🛠️ Wazuh API Quick Shortcuts

The Wazuh API is essential for advanced management. Perform these actions from any terminal with `curl`.

| Action | API Endpoint | Purpose |
| :--- | :--- | :--- |
| **List Agents** | `GET /agents` | Check status of all protected hosts |
| **Agent Config** | `GET /agents/{id}/config/viz/ossec` | View remote `ossec.conf` |
| **Last Scan** | `GET /syscollector/{id}/packages` | List all software on an agent |
| **Trigger Scan** | `PUT /syscollector/{id}/scan` | Force a vulnerability scan |

## 🧬 SOC-Standard Regex Library

Use these in Module 03 (Lucene) or the Search Bar for high-fidelity hunting.

1.  **Sussuspicious Base64 (long):** `/[A-Za-z0-9+\/]{100,}/`
2.  **Hex-encoded PowerShell:** `/powershell.*-e(nc(odedcommand)?)? [A-Za-z0-9+/=]+/`
3.  **Internal IPv4 Leak:** `/10\.\d{1,3}\.\d{1,3}\.\d{1,3}/`
4.  **Suspicious PHP upload:** `/move_uploaded_file\(.*\)/`

---

## 🚀 Pro-Tip: The "Golden Interval"
For high-speed triage, always use the filter:
`@timestamp: [now-15m TO now]`
This keeps the search space small and the dashboard snappy.

---

**Previous: [08 - Common Mistakes](08-common-mistakes.md)** | **Next: [10 - Forensic Analysis](10-forensic-analysis.md)**

[Return to Index](../../README.md)
