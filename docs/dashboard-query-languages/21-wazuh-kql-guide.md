````markdown
# Wazuh-Specific KQL Guide

This guide maps Wazuh field names and common detection/use-case queries to KQL and short DSL snippets. Use the `wazuh-alerts-*` index pattern in `Discover` or Dashboard panels.

## 1. Introduction to Wazuh Dashboard Query Language

- What is KQL in Wazuh? KQL (Kibana Query Language) is the user-friendly query language exposed in OpenSearch/Wazuh Dashboards for field-aware searches.
- How Wazuh uses Elasticsearch backend: Wazuh stores alerts and events in OpenSearch/Elasticsearch indices (`wazuh-alerts-*`, `wazuh-archives-*`), and queries are translated to DSL for execution.
- Where DQL/KQL is used: `Discover`, saved searches, Dashboard panels, Security Events views, and Dev Tools for JSON DSL.
- KQL vs Lucene in Wazuh: KQL is safer and easier (field-aware); Lucene supports regex and some legacy syntax.
- Index patterns: `wazuh-alerts-*`, `wazuh-archives-*`, `wazuh-monitoring-*`.

## 2. Understanding Wazuh Fields Structure

- Common fields and examples:

- `agent.id` — agent identifier
- `agent.name` — hostname (use `agent.name.keyword` for exact match)
- `agent.ip` — agent IP
- `rule.id` — rule identifier
- `rule.level` — alert severity (0–16)
- `rule.description` — human-readable description
- `rule.groups` — groups/tags for the rule
- `decoder.name` — decoder that parsed the log
- `data.*` — raw event payload (Sysmon, Windows event fields, etc.)
- `manager.name` — Wazuh manager host
- `location` — file path (for FIM events)
- `input.type` — log input (syslog, filebeat, etc.)
- `@timestamp` — event time

## 3. Basic Filtering in Wazuh (KQL)

- Filter by Agent Name: `agent.name: "linux-prod-01"`
- Filter by Agent ID: `agent.id: "005"`
- Filter by Rule ID: `rule.id: "100201"`
- Filter by Rule Level: `rule.level >= 10`
- Filter by Event Category (example): `rule.groups: "sshd"`
- Free Text Search in Alerts: `message: "authentication failed"`

## 4. Logical Operators in Wazuh Queries

- AND: `rule.groups: "sshd" AND rule.level >= 7`
- OR: `rule.groups: "sshd" OR rule.groups: "auth"`
- NOT: `NOT rule.groups: "info"`
- Grouping: `(rule.level >= 10 AND agent.name: "db01") OR data.srcip: "10.0.0.5"`
- Combine rule + agent + time: `rule.id: "100201" AND agent.name: "web-01" AND @timestamp >= now-24h`

## 5. Severity & Rule Level Queries

- High severity: `rule.level >= 10`
- Critical: `rule.level >= 12`
- Low-noise filter: `NOT rule.level: [0 TO 3]`
- Compliance group: `rule.groups: "pci_dss"`

## 6. Authentication Monitoring Queries

- Failed login (Windows): `data.win.eventid: 4625`
- Successful login: `data.win.eventid: 4624`
- Multiple failed logins (start with raw filter, aggregate in dashboard): `data.win.eventid:4625`
- Logon type filtering (RDP): `data.win.eventdata.LogonType: 10`
- SSH failures (Linux): `rule.groups: "sshd" AND rule.level >= 7`

## 7. Process Monitoring Queries

- Suspicious process execution (Sysmon): `data.win.eventid:1 AND data.process_name: "psexec.exe"`
- PowerShell activity: `data.process_name: "powershell.exe"`
- Encoded commands: `data.process_cmdline: "*EncodedCommand*"`
- Parent-child relationships: use `data.process_parent_name` and `data.process_name` fields (example): `data.process_parent_name: "explorer.exe" AND data.process_name: "cmd.exe"`

## 8. File Integrity Monitoring (FIM) Queries

- File added: `rule.groups: "syscheck" AND data.change: "added"`
- File modified: `rule.groups: "syscheck" AND data.change: "modified"`
- File deleted: `rule.groups: "syscheck" AND data.change: "deleted"`
- Sensitive dirs: `location: "/etc/*" OR location: "C:\\Windows\\System32\\*"`

## 9. Network & Firewall Queries

- Source IP filter: `data.srcip: "198.51.100.10"`
- Destination IP: `data.dstip: "10.0.0.20"`
- Port filter: `data.dstport: 3389`
- Internal vs External: combine CIDR lists or use enrichment tags (example): `NOT data.srcip: 10.0.0.0/8`
- Port scanning indicator: `data.dstport: * AND rule.groups: "scan"` (depends on rule names)

## 10. Malware & Threat Detection Queries

- Antivirus alerts: `rule.groups: "antivirus"`
- Suspicious hash: `data.file.hash: "*"` (match exact hash with `.keyword` when available)
- Threat intel IOC match: `rule.groups: "threatintel" OR rule.description: "IOC"`

## 11. Threat Hunting Queries

- Lateral movement: `data.process_cmdline: ("psexec" OR "wmic" OR "net use")`
- Privilege escalation: `data.win.eventid: 4672 OR rule.groups: "sudo"`
- Suspicious service creation: `data.win.eventid: 7045 OR rule.groups: "service"`
- Scheduled task abuse: `data.win.eventid: 4698 OR 4702`

## 12. IP & Geo Queries

- Exact IP: `data.srcip: "203.0.113.5"`
- CIDR range: depends on UI support; use DSL for advanced ranges and `ip_range` queries
- Filter external IP addresses: `NOT data.srcip: 10.0.0.0/8 AND NOT data.srcip: 192.168.0.0/16`
- Country detection: use geoip enrichment fields if present (e.g., `geoip.country_name: "Russia"`)

## 13. Time-Based Queries

- Last 1 hour: `@timestamp >= now-1h`
- Last 24 hours: `@timestamp >= now-24h`
- Last 7 days: `@timestamp >= now-7d`
- Custom ranges: `@timestamp >= "2025-01-01" AND @timestamp < "2025-01-02"`

## 14. Compliance & Regulatory Queries

- PCI-DSS: `rule.groups: "pci_dss"`
- GDPR-related: `rule.groups: "gdpr" OR rule.description: "personal data"`
- HIPAA: `rule.groups: "hipaa"`
- CIS checks: `rule.groups: "cis"`

## 15. Aggregation & Dashboard Filtering

- Top N rule IDs (use Terms agg on `rule.id.keyword`)
- Top attacked agents: Terms agg on `agent.name.keyword`
- Most frequent src IPs: Terms agg on `data.srcip.keyword`
- Event count by severity: Terms agg on `rule.level`

## 16. Wazuh Index & Data Management

- Primary indices: `wazuh-alerts-*` (alerts), `wazuh-archives-*` (raw events), `wazuh-monitoring-*` (health).
- Search raw logs vs alerts: use `wazuh-archives-*` to find non-alert events.
- Field mapping: check `GET /wazuh-alerts-*/_mapping` in Dev Tools for types and `.keyword` availability.

## 17. Advanced Query Techniques

- Nested fields: query directly using dotted notation (e.g., `data.win.eventdata.TargetUserName: "admin"`).
- Wildcards in `rule.description`: `rule.description: "*credential*"`
- Multi-value filtering: `agent.name: ("web01" OR "web02")`
- Exists/missing: `_exists_: data.win.eventid` or `NOT _exists_: data.win.eventid`

## 18. Investigation & Incident Response Queries

- Pivot agent ↔ IP: `agent.name: "host1"` then check `data.srcip` values; or start with `data.srcip: "x.x.x.x"` and pivot to `agent.name`
- Correlate auth + process: `data.win.eventid: (4624 OR 4625) OR data.win.eventid:1` and then narrow by `@timestamp` window
- First occurrence: use sorting by `@timestamp` asc and `size:1` in DSL

## 19. SOC Use Case Queries

- Brute force dashboard: failed logins (`data.win.eventid:4625`), aggs on `data.srcip.keyword`
- Insider threat: unusual data exports, `data.process_cmdline` with cloud upload tools
- Ransomware: mass file modifications in FIM (`rule.groups: "syscheck" AND data.change: "modified"`)

## 20. Query Optimization & Best Practices

- Use `.keyword` for exact matches and aggregations, avoid leading wildcards, narrow time ranges, prefer `filter` clauses in DSL for cacheable operations.

## 21. Troubleshooting Wazuh Queries

- No results: check time range, index pattern, and mapping. Use Dev Tools to inspect mappings and a sample document.
- Incorrect field names: confirm sample document keys in `Discover` table.
- Timestamp issues: ensure `@timestamp` exists and mapping uses `date` type.

## 22. Building Custom Wazuh Dashboards

- Create saved searches for frequent filters, add visualization panels (Terms, Date histogram), compose a SOC overview dashboard, and secure sharing via role-based access.

---

If you want, I can expand any numbered section with KQL + Lucene + DSL runnable examples and saved dashboard JSON for direct import.

````
