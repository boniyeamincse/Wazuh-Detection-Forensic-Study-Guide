# Examples & Queries for Wazuh Dashboard

This page collects practical topics, short examples, and ready-to-run queries you can paste into the Wazuh Dashboard `Discover` or `Search` panels (use the `wazuh-alerts-*` index pattern).

## TL;DR

- Quick, copy-paste KQL, Lucene, and OpenSearch DSL examples for common detections and dashboard visualizations.

## Topics Covered

- Failed authentication (SSH/Winlogon)
- Suspicious process creation (Sysmon)
- PowerShell / Commandline detections
- Lateral movement indicators (SMB, RDP)
- Aggregations for dashboards (top IPs, rule levels)

## How to use

- Open the Wazuh Dashboard > Discover. Select the `wazuh-alerts-*` index pattern. Paste a KQL query into the search bar and run it. For Lucene or DSL, switch query language accordingly.

---

## KQL Examples

- Failed SSH authentication (high-level):

```
rule.groups: "sshd" and rule.level >= 7
```

- Failed SSH authentication (specific user):

```
rule.groups: "sshd" and data.user: "root" and rule.level >= 7
```

- Suspicious process creation (Windows Sysmon):

```
data.win.eventid: 1 and data.process_name: "powershell.exe" and data.process_cmdline: "*-EncodedCommand*"
```

- Command-line containing suspicious tools (Linux):

```
data.process_name: "bash" and data.process_cmdline: "*nc *" or data.process_cmdline: "*curl *--output*"
```

---

## Lucene Examples

- Failed SSH (Lucene):

```
rule.groups:"sshd" AND rule.level:[7 TO *]
```

- Suspicious process creation (Lucene):

```
data.win.eventid:1 AND data.process_name:"powershell.exe" AND data.process_cmdline:/EncodedCommand/
```

---

## OpenSearch Query DSL (JSON) — Failed SSH example

```
{
  "query": {
    "bool": {
      "must": [
        { "match": { "rule.groups": "sshd" }},
        { "range": { "rule.level": { "gte": 7 }}}
      ]
    }
  }
}
```

Paste this JSON into the Dashboard dev tools or use it with the OpenSearch API to reproduce the same filter programmatically.

---

## Aggregations & Dashboard widgets (ideas)

- Top 10 source IPs triggering alerts: use a `Terms` aggregation on `data.srcip.keyword`.
- Alert volume over time: `Date Histogram` on `@timestamp` or `timestamp`.
- Rule level distribution: `Terms` on `rule.level` (or `Histogram` for numeric buckets).
- Hosts with most alerts: `Terms` on `agent.name.keyword`.

## Quick Tips

- Always use `.keyword` fields for aggregation and exact matches (e.g., `agent.name.keyword`).
- When testing queries, widen time range to "Last 90 days" to get more results in low-volume environments.
- Save useful searches and pin them to dashboards as panels.

- Run a spellcheck and lint across docs to standardize style.

---

**Previous: [14 - Learning Resources](14-learning-resources.md)** | **Next: [16 - Windows Auth & Lateral Movement](16-rdp-smb-windows-auth.md)**

[Return to Index](../../README.md)
