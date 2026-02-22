# RDP, SMB & Windows Authentication Detection Queries

Practical KQL, Lucene, and OpenSearch DSL queries for common Windows authentication and lateral movement indicators. Use the `wazuh-alerts-*` index pattern.

## TL;DR

- Quick queries to find RDP failed/successful logons, SMB share access, and aggregate source IPs for brute-force detection.

## RDP — Failed logon (KQL)

```
data.win.eventid:4625 and data.win.eventdata.LogonType:10
```

Notes: Event ID 4625 = failed logon. LogonType 10 typically indicates RemoteInteractive (RDP). Adjust `LogonType` and time range as needed.

## RDP — Successful logon (KQL)

```
data.win.eventid:4624 and data.win.eventdata.LogonType:10
```

## SMB — File/share access (KQL)

```
data.win.eventid:5140 OR data.win.eventid:5145
```

Notes: 5140/5145 relate to network share access and file creation over SMB — useful to spot lateral file access.

## Brute-force / credential stuffing (aggregation example)

KQL to find many failed logons grouped by source IP (then use Dashboard visualization):

```
data.win.eventid:4625
```

Then create a `Terms` aggregation on `data.srcip.keyword` and sort by count desc, or use this OpenSearch DSL to get top source IPs:

```
{
  "size": 0,
  "query": { "term": { "data.win.eventid": 4625 }},
  "aggs": {
    "top_src": { "terms": { "field": "data.srcip.keyword", "size": 10 }}
  }
}
```

## Lucene equivalents

- RDP failed (Lucene):

```
data.win.eventid:4625 AND data.win.eventdata.LogonType:10
```

- SMB access (Lucene):

```
data.win.eventid:5140 OR data.win.eventid:5145
```

## Quick detection tips

- Correlate `4625` failed logon spikes with `4624` successes from the same `data.srcip`.
- Watch for `LogonType` values: 10 (RemoteInteractive/RDP), 3 (Network), 2 (Interactive/local).
- Combine with `agent.name` or `agent.ip` to find which host is being targeted.
- Use wider time ranges when testing to capture low-volume environments.

- Create saved dashboard panels: RDP brute-force heatmap, SMB access top talkers, and recent successful RDP sessions.

---

**Previous: [15 - Examples & Queries](15-examples-queries.md)** | **Next: [20 - DQL Complete Guide](20-dql-guide.md)**

[Return to Index](../../README.md)
