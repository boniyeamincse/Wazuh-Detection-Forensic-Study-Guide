````markdown
# Wazuh-Specific DQL (KQL) — Full Guide (22 Topics)

This comprehensive guide explains how to use KQL and related query techniques in Wazuh/OpenSearch Dashboards. Each section includes explanations, KQL examples, optional DSL snippets, and practical tips for SOC workflows.

---

🔰 1. Introduction to Wazuh Dashboard Query Language

- What is KQL in Wazuh?
  - KQL (Kibana Query Language) is the interactive, field-aware query language used in Wazuh/OpenSearch Dashboards for searching and filtering documents.
- How Wazuh uses the Elasticsearch backend
  - Wazuh stores alerts and raw events in OpenSearch/Elasticsearch indices (e.g., `wazuh-alerts-*`, `wazuh-archives-*`). Dashboard queries are translated to the Elasticsearch Query DSL for execution.
- Where DQL/KQL is used
  - `Discover`, `Dashboards` panels, saved searches, Security Events, and Dev Tools (for DSL).
- Differences: KQL vs Lucene in Wazuh
  - KQL: simpler, safer, field-aware; Lucene: legacy syntax, regex support, sometimes required for advanced searches.
- Index patterns in Wazuh
  - `wazuh-alerts-*`: alerts; `wazuh-archives-*`: raw events; `wazuh-monitoring-*`: health & metrics.

---

📂 2. Understanding Wazuh Fields Structure

- `agent.id` — unique numeric/string ID for the agent
- `agent.name` — host hostname (use `agent.name.keyword` for exact matches)
- `agent.ip` — agent IP address
- `rule.id` — Wazuh rule identifier
- `rule.level` — severity (0–16)
- `rule.description` — human-readable summary
- `rule.groups` — tags/groups (e.g., `sshd`, `pci_dss`)
- `decoder.name` — decoder used for parsing
- `data.*` — raw parsed event fields (e.g., `data.win.eventid`, `data.process_name`)
- `manager.name` — manager hostname
- `location` — file path for FIM events
- `input.type` — log input source
- `@timestamp` — event timestamp

Tips: inspect a sample document in `Discover` to confirm exact field names and whether `.keyword` variants exist.

---

🔎 3. Basic Filtering in Wazuh (KQL examples)

- Filter by Agent Name

```
agent.name: "server01"
```

- Filter by Agent ID

```
agent.id: "001"
```

- Filter by Rule ID (single / multiple)

```
rule.id: 1002
rule.id: (1002 OR 5710 OR 18107)
```

- Filter by Rule Level

```
rule.level >= 10
```

- Filter by Event Category (groups)

```
rule.groups: "sshd"
```

- Free Text Search in Alerts

```
message: "authentication failed"
```

---

🔗 4. Logical Operators in Wazuh Queries

- AND conditions

```
rule.groups: "sshd" AND rule.level >= 7
```

- OR conditions

```
rule.groups: "sshd" OR rule.groups: "auth"
```

- NOT operator

```
NOT rule.groups: "info"
```

- Grouped conditions with parentheses

```
(rule.level >= 10 AND agent.name: "db01") OR data.srcip: "10.0.0.5"
```

- Combining rule + agent + time filters

```
rule.id: "100201" AND agent.name: "web-01" AND @timestamp >= now-24h
```

---

📊 5. Severity & Rule Level Queries

- High severity alerts

```
rule.level >= 10
```

- Critical alerts

```
rule.level >= 12
```

- Low-level noise filtering

```
NOT rule.level: [0 TO 3]
```

- Filtering by compliance group

```
rule.groups: "pci_dss"
```

---

🔐 6. Authentication Monitoring Queries

- Failed login detection (Windows)

```
data.win.eventid: 4625
```

- Successful login tracking

```
data.win.eventid: 4624
```

- Multiple failed logins (brute-force indicators)

```
data.win.eventid: 4625
```

  - In dashboards, aggregate by `data.srcip.keyword` and sort by count.
- Logon type filtering

```
data.win.eventdata.LogonType: 10  # RDP
```

- SSH login failures (Linux)

```
rule.groups: "sshd" AND rule.level >= 7
```

Tips: combine failed/successful queries with `@timestamp` windows to detect successful logins following bursts of failures.

---

🖥️ 7. Process Monitoring Queries

- Suspicious process execution

```
data.process_name: "psexec.exe" OR data.process_name: "wmic.exe"
```

- PowerShell activity

```
data.process_name: "powershell.exe"
```

- Encoded commands

```
data.process_cmdline: "*EncodedCommand*"
```

- Parent-child process relationships

```
data.process_parent_name: "winword.exe" AND data.process_name: "cmd.exe"
```

- Process created by Office apps (macro abuse)

```
data.process_parent_name: "winword.exe" AND data.process_name: "powershell.exe"
```

- Processes with unusual command-line arguments

```
data.process_cmdline: "*--encoded*" OR data.process_cmdline: "*Invoke-Expression*"
```

---

📁 8. File Integrity Monitoring (FIM) Queries

- File added

```
rule.groups: "syscheck" AND syscheck.event: "added"
```

- File modified

```
syscheck.event: "modified"
```

- File deleted

```
syscheck.event: "deleted"
```

- Monitoring sensitive directories

```
location: "/etc/*" OR location: "C:\\Windows\\System32\\*"
```

- Detecting system32 or `/etc` changes

```
syscheck.path: "C:\\Windows\\System32\\*" OR location: "/etc/*"
```

---

🌐 9. Network & Firewall Queries

- Source IP filtering

```
data.srcip: "192.168.1.10"
```

- Destination IP filtering

```
data.dstip: "10.0.0.20"
```

- Port-based filtering

```
data.dstport: 3389
```

- Internal vs External IP detection

```
NOT data.srcip: (10.* OR 192.168.* OR 172.16.*)
```

- Detecting port scanning behavior

```
rule.groups: "scan" OR data.dstport: *  # depends on available rules
```

---

🛡️ 10. Malware & Threat Detection Queries

- Antivirus alerts

```
rule.groups: "antivirus" OR rule.groups: "virus"
```

- Malware detection events

```
rule.groups: "malware"
```

- Suspicious hash detection

```
data.file.hash.keyword: "<hash>"
```

- Threat intelligence match queries

```
rule.groups: "threatintel" OR rule.description: "IOC"
```

- YARA detection filtering

```
rule.groups: "yara"
```

---

🧠 11. Threat Hunting Queries

- Lateral movement detection

```
data.process_cmdline: ("psexec" OR "wmic" OR "net use")
```

- Privilege escalation indicators

```
data.win.eventid: 4672 OR rule.groups: "sudo"
```

- Suspicious service creation

```
data.win.eventid: 7045
```

- Scheduled task abuse

```
data.win.eventid: (4698 OR 4702)
```

- Registry persistence detection

```
rule.description: "*Run*" OR data.registry.key: "*\\Run\\*"
```

- Reverse shell indicators

```
data.process_cmdline: ("nc -e" OR "bash -i >& /dev/tcp")
```

---

📡 12. IP & Geo Queries

- Exact IP match

```
data.srcip: "203.0.113.5"
```

- CIDR range filtering (DSL recommended)

```
# Use DSL with ip_range or range queries for CIDR; UI KQL may not support CIDR directly.
```

- Filtering external IP addresses

```
NOT data.srcip: (10.* OR 192.168.* OR 172.16.*)
```

- Geo-location based filtering

```
geoip.country_name: "China"
```

---

📅 13. Time-Based Queries

- Last 1 hour

```
@timestamp >= now-1h
```

- Last 24 hours

```
@timestamp >= now-24h
```

- Last 7 days

```
@timestamp >= now-7d
```

- Custom date ranges

```
@timestamp >= "2026-02-01" AND @timestamp <= "2026-02-22"
```


---

🧾 14. Compliance & Regulatory Queries

- PCI-DSS alerts

```
rule.groups: "pci_dss"
```

- GDPR-related alerts

```
rule.groups: "gdpr" OR rule.description: "personal data"
```

- HIPAA-related alerts

```
rule.groups: "hipaa"
```

- CIS benchmark checks

```
rule.groups: "cis"
```

---

🧮 15. Aggregation & Dashboard Filtering

- Top N rule IDs — Terms aggregation on `rule.id.keyword`.
- Top attacked agents — Terms agg on `agent.name.keyword`.
- Most frequent source IPs — Terms agg on `data.srcip.keyword`.
- Event count by severity — Terms agg on `rule.level`.

DSL example (top srcips):

```
{"size":0,"query":{"term":{"data.win.eventid":4625}},"aggs":{"top_src":{"terms":{"field":"data.srcip.keyword","size":10}}}}
```

---

⚙️ 16. Wazuh Index & Data Management

- Primary indices: `wazuh-alerts-*`, `wazuh-archives-*`, `wazuh-monitoring-*`.
- Searching raw logs vs alerts: use `wazuh-archives-*` for raw events that didn't trigger rules.
- Understanding indexed fields: check mappings in Dev Tools: `GET /wazuh-alerts-*/_mapping`.
- Field mapping issues: mismatched types (text vs keyword) cause aggregation problems; use runtime fields or reindex if needed.

---

🔁 17. Advanced Query Techniques

- Nested field queries: `data.win.eventdata.TargetUserName: "admin"`.
- Wildcards in `rule.description`: `rule.description: "*credential*"` (avoid leading wildcard for performance).
- Multi-value filtering: `agent.name: ("web01" OR "web02")`.
- Exists and missing field detection: `_exists_: data.srcip` and `NOT _exists_: data.srcip`.
- Filtering on deep fields: `data.win.eventdata.*` works with dotted notation.
- Escaping special characters in paths: `location: "C:\\Program Files\\*"`.

---

🧪 18. Investigation & Incident Response Queries

- Pivoting between agent and IP: start with `agent.name: "host1"` then inspect `data.srcip` values; reverse starting point with `data.srcip`.
- Correlating authentication + process events: create a dashboard with saved searches for `data.win.eventid:4625` and `data.win.eventid:1` and use the same time window.
- Tracking attacker timeline: sort results by `@timestamp` and build a timeline visualization.
- Identifying first occurrence: DSL `size:1` sort by `@timestamp` asc.
- Alert deduplication techniques: use `composite` aggregations on `rule.id` + `agent.name` + `data.srcip`.

---

🚨 19. SOC Use Case Queries

- Brute force dashboard: aggregate `data.win.eventid:4625` by `data.srcip.keyword` and `agent.name.keyword`.
- Insider threat detection: look for unusual process commands, large data transfers, and abnormal login times.
- Web attack monitoring: rule groups for web servers and suspicious URLs in `data.request` fields.
- Ransomware detection: mass FIM `modified` events in short time windows and file extension changes.
- Privileged account monitoring: `rule.level >= 10` combined with `data.win.eventdata.LogonType` and time-of-day anomalies.

---

📈 20. Query Optimization & Best Practices

- Use `.keyword` fields for aggregations and exact matches (e.g., `agent.name.keyword`).
- Avoid leading wildcards and expensive regexes.
- Reduce heavy OR conditions by using `terms` queries in DSL or aggregations in dashboards.
- Prefer `filter` clauses in DSL for cacheable filters.
- Narrow time ranges when developing queries; widen for hunting.

---

🔍 21. Troubleshooting Wazuh Queries

- No results found: verify time range, index pattern, and sample document fields.
- Incorrect field names: inspect a document in `Discover` or run mapping queries.
- Field mapping conflicts: check mappings and consider creating runtime fields to normalize.
- Timestamp problems: ensure `@timestamp` exists and mapping type is `date`.
- Index pattern problems: re-create index pattern if field list is stale.

---

🏗️ 22. Building Custom Wazuh Dashboards

Steps to build a SOC overview dashboard:
1. Create saved searches for high-value filters (Failed logins, RDP, SMB, FIM critical paths).
2. Build visualizations: Time series (Date histogram), Top N Terms (IPs, hosts, rules), Metric panels (counts).
3. Combine visualizations in a dashboard; add filter controls and time picker.
4. Save and share with RBAC-appropriate roles.
5. Export dashboard JSON for reuse across environments.

---

If you'd like, I can now:
- Expand any section into step-by-step examples and saved-panel JSON (pick 1–3 sections), or
- Run a repo-wide spellcheck and standardize doc formatting.

````
