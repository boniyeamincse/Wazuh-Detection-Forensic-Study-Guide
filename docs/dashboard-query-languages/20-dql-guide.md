# DQL Rosetta Stone: Comparative Query Guide 🧭

This module provides a side-by-side comparison of the three major query languages used in the Wazuh Dashboard. It is designed to help analysts quickly translate a "hunt" from one language to another.

## 🔀 The "Rosetta Stone" Table

| Use Case | KQL (Intuitive) | Lucene (Legacy/Regex) | OpenSearch DSL (Power User) |
| :--- | :--- | :--- | :--- |
| **Simple Filter** | `agent.name: "web-01"` | `agent.name:"web-01"` | `{"term": {"agent.name.keyword": "web-01"}}` |
| **Logic (AND)** | `rule.level: >10 AND rule.groups: "sshd"` | `rule.level:[10 TO *] AND rule.groups:"sshd"` | `{"bool": {"must": [{"range": {"rule.level": {"gt": 10}}}, {"match": {"rule.group": "sshd"}}]}}` |
| **Logic (OR)** | `rule.id: (5710 OR 5716)` | `rule.id:(5710 OR 5716)` | `{"bool": {"should": [{"term": {"rule.id": 5710}}, {"term": {"rule.id": 5716}}]}}` |
| **Wildcard** | `data.process_cmdline: *mimikatz*` | `data.process_cmdline:*mimikatz*` | `{"wildcard": {"data.process_cmdline.keyword": "*mimikatz*"}}` |
| **Regex Search** | (Use DSL or Lucene) | `data.file: /.*\.tmp/` | `{"regexp": {"data.file.keyword": ".*\\.tmp"}}` |
| **Field Exists** | `data.srcip: *` | `_exists_:data.srcip` | `{"exists": {"field": "data.srcip"}}` |
| **IP Range** | (Depend on UI) | `data.srcip: 10.0.0.0/24` | `{"range": {"data.srcip": {"gte": "10.0.0.0", "lte": "10.0.0.255"}}}` |

---

## 💎 When to Use Which?

### 1. KQL (Kibana Query Language)
- **Best for:** Everyday triage, rapid filtering, and auto-complete support.
- **Limitation:** Does not support deep regex or complex scripted logic.

### 2. Lucene
- **Best for:** Complex wildcard patterns and regex directly in the search bar.
- **Limitation:** Strict syntax; prone to errors with special characters.

### 3. OpenSearch Query DSL (JSON)
- **Best for:** Python/Curl automation, building complex Dashboard visualizations, and pipeline aggregations.
- **Limitation:** High complexity; requires valid JSON structure.

---

## ⚡ Quick Conversion Cheat-Sheet

- **KQL `NOT`** → **Lucene `-`** → **DSL `must_not`**
- **KQL `>`** → **Lucene `[X TO *]`** → **DSL `range: { "gt": X }`**
- **KQL `*`** → **Lucene `*`** → **DSL `wildcard`**

---

**Previous: [16 - Windows Auth & Lateral Movement](16-rdp-smb-windows-auth.md)** | **[Return to Index](../../README.md)**
