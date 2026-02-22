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

## 🕵️ SOC Rapid Investigation Templates

### 1. The "Attacker Pivot" (Find all from IP)
`data.srcip: "1.2.3.4" OR data.win.eventdata.ipAddress: "1.2.3.4" OR data.aws.sourceIPAddress: "1.2.3.4"`

### 2. The "Host Triage" (Critical events on 1 Host)
`agent.name: "Prod-DB-01" AND rule.level: >= 12`

### 3. The "Account Sweep" (Where did this user login?)
`data.win.eventdata.targetUserName: "jdoe" OR data.linux.user: "jdoe" OR data.aws.userIdentity.userName: "jdoe"`

### 4. The "LOLBIN Hunt" (Suspicious binaries)
`data.win.eventdata.image: ("*\\powershell.exe" OR "*\\cmd.exe" OR "*\\certutil.exe" OR "*\\mshta.exe" OR "*\\regvsvr32.exe")`

---

## 📊 Common Performance Rule Mapping

| Field | Type | Search Syntax |
| :--- | :--- | :--- |
| `rule.id` | Keyword | `rule.id: 5710` |
| `agent.name`| Keyword | `agent.name.keyword: "web-01"` |
| `rule.description` | Text | `rule.description: "failed password"` |
| `full_log` | Text | `full_log: "*malicious*"` |

---

## 🚀 Pro-Tip: The "Golden Interval"
For high-speed triage, always use the filter:
`@timestamp: [now-15m TO now]`
This keeps the search space small and the dashboard snappy.

---

**Previous: [08 - Common Mistakes](08-common-mistakes.md)** | **Next: [10 - Forensic Analysis](10-forensic-analysis.md)**

[Return to Index](../../README.md)
