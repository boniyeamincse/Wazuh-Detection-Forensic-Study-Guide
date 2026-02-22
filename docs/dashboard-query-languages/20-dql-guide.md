# DQL & Dashboard Query Languages — Complete Guide

This page explains Dashboard Query Languages (DQL) used in Wazuh/OpenSearch Dashboards and similar platforms (KQL, Lucene, Elasticsearch Query DSL), with examples you can paste into `Discover`, saved searches, or Dashboard panels.

## 1. Introduction to Dashboard Query Language

- What is DQL? A collective term for languages used to search and filter time-series and log data in dashboards.
- What is KQL? Kibana Query Language — user-friendly, field-aware, and does not require quoting for many expressions.
- What is Elasticsearch Query DSL? JSON-based, fully expressive query language used by OpenSearch/Elasticsearch APIs.
- Why Query Languages Matter: enable precise detection, efficient dashboards, and reproducible hunts.
- Use cases: SIEM detections, SOC triage, uptime/infra monitoring, threat hunting.

## 2. Basic Query Syntax

- Field-Based Search: `agent.name: "web-01"`
- Free Text Search: `error OR failure`
- Exact Match Queries: `user: "root"` or `user.keyword: "root"`
- Keyword vs Text: use `.keyword` for exact matches/aggregations, `text` for full-text search
- Case Sensitivity: most fields are case-sensitive for exact `keyword` searches.

## 3. Logical Operators

- AND: `status:200 AND extension: ".exe"`
- OR: `rule.groups: "sshd" OR rule.groups: "auth"`
- NOT: `NOT rule.groups: "info"`
- Parentheses: `(rule.level:>8 AND agent.name: "db01") OR (data.srcip:10.0.0.5)`

## 4. Range Queries

- Numeric: `bytes:[100 TO 1000]` (Lucene style) or KQL `bytes >= 100 and bytes <= 1000`
- Date: `@timestamp:[now-24h TO now]` or KQL `@timestamp >= now-24h`
- Greater/Less: `response_time > 500`
- Time shortcuts: `now-1h`, `now-7d`

## 5. Wildcards and Pattern Matching

- Multi-character `*`: `data.process_cmdline: "*powershell*"`
- Single-character `?`: `file: "?.exe"`
- Performance: avoid leading `*` (`*abc`) on large datasets
- Escaping: use backslash to escape special chars: `message:\"error\"`

## 6. IP and Network Queries

- Exact IP: `data.srcip: "192.0.2.10"`
- CIDR (DSL example): use `ip_range` or range queries in DSL; KQL may support `network` functions depending on UI
- Private vs Public: filter by CIDR ranges (e.g., `data.srcip: 10.0.0.0/8` in systems that support it)
- Geo queries: use geo_point fields in DSL and `geo_distance`.

## 7. Exists and Missing Fields

- Field exists (KQL): `_exists_: data.user`
- Missing: `NOT _exists_: data.win.eventid`
- Null handling: fields may be absent vs null; check mappings.

## 8. Multi-Value Queries

- OR lists: `user: (alice OR bob OR carol)`
- IN-style: `user.keyword: ("alice","bob")` (Lucene/KQL vary)
- Comma-separated: depends on UI — prefer explicit OR lists for clarity.

## 9. Security Hunting Queries (examples)

- Failed logins (Windows): `data.win.eventid:4625`
- RDP brute force (aggregation): KQL `data.win.eventid:4625` then `Terms` agg on `data.srcip.keyword`
- Suspicious process (Sysmon): `data.win.eventid:1 AND data.process_name: "psexec.exe"`
- Encoded PowerShell: `data.process_cmdline: "* -EncodedCommand *"` or `process_cmdline:/EncodedCommand/`
- Privilege escalation: `rule.groups: "sudo" OR data.win.eventid:4672`
- Lateral movement: `data.process_cmdline: ("psexec" OR "wmic" OR "winrm")`

## 10. Advanced Query Techniques

- Nested fields: `user.name: "admin"` or `data.win.eventdata.TargetUserName: "admin"`
- Scripted/runtime fields: create runtime fields to normalize hostnames or parse JSON
- Aggregation filtering: use `bucket_selector` to filter aggregation results
- Sub-queries (DSL): embed `bool` queries with `must`, `should`, `filter`.

## 11. Elasticsearch Query DSL (examples)

- Match query (text):

```
{"match": {"message": "failed login"}}
```

- Term query (exact):

```
{"term": {"agent.name.keyword": "web-01"}}
```

- Bool query:

```
{"bool": {"must": [{"term": {"rule.groups": "sshd"}}, {"range": {"rule.level": {"gte": 7}}}]}}
```

- Range (JSON):

```
{"range": {"@timestamp": {"gte": "now-7d"}}}
```

- Aggregation (top source IPs):

```
{"size":0, "aggs": {"top_src": {"terms": {"field":"data.srcip.keyword","size":10}}}}
```

## 12. Dashboard Variables (Grafana-style)

- Variables let users change queries without editing panels. Common types: Query, Custom, Interval, Constant, Multi-select.
- Example: create `$host` variable with `terms` on `agent.name.keyword`, then use `agent.name:$host` in panel queries.

## 13. Loop-Like Behavior in Dashboards

- Repeat panels by variable (one panel per host or per rule group) — useful for per-host time series.

## 14. Filters and Controls

- Global filters affect all panels. Use pinned filters for persistent constraints.
- Time picker: always set reasonable defaults and allow quick ranges (15m, 1h, 24h).

## 15. Aggregations and Metrics

- Count: simple `doc_count` for alert volumes.
- Sum/Avg: numeric field metrics (bytes, duration).
- Terms: top N values for `agent.name.keyword`, `rule.id`.

## 16. Query Optimization & Best Practices

- Avoid leading wildcards and heavy regex.
- Use `.keyword` for aggregations/exact matches.
- Narrow time ranges for development; widen for low-volume tests.
- Use `filter` clauses in DSL for cacheable filters.

## 17. Troubleshooting Queries

- No results: widen time range, check index pattern, verify field mapping.
- Mapping issues: check field type in Dev Tools: `GET wazuh-alerts-*/_mapping`.
- Syntax errors: switch to DSL and validate JSON in Dev Tools.

## 18. SIEM / SOC Use Cases

- Authentication monitoring, malware detection, FIM alerts, network anomaly detection, insider threat dashboards, threat-hunting workbooks.

## 19. Comparison of Query Languages

- KQL: easy for analysts, good for interactive search.
- Lucene: powerful legacy syntax, supports regex.
- Elasticsearch DSL: most powerful and scriptable for automation.

## 20. Real-World Dashboard Examples (quick)

- SOC Monitoring Dashboard: panels — Alerts over time, Top rules, Top source IPs, Hosts with highest alerts.
- Executive: trend lines, top-3 incident types, SLA metrics.
- Infrastructure: host uptime, disk/CPU metrics, alert counts per cluster.
- Threat Intel: Alerts enriched by IOC matches, top matched IOCs.

---

If you want, I can: add full runnable examples for each section (KQL + DSL + saved panel JSON), or generate Grafana dashboard templates.

---

**Previous: [16 - Windows Auth & Lateral Movement](16-rdp-smb-windows-auth.md)** | **[Return to Index](../../README.md)**
