````markdown
# Authentication Monitoring — Step-by-step Guide

This page provides stepwise examples for detecting authentication events in Wazuh: failed logins, successful logins, brute-force detection, SSH failures, and RDP activity. Includes KQL, Lucene, and OpenSearch DSL samples and instructions to build a simple dashboard panel.

## Quick notes

- Index pattern: select `wazuh-alerts-*` in `Discover` or Dashboards.
- Time range: start with `Last 7 days` when testing; shorten once you confirm queries.

---

## 1) Failed Windows logins (KQL)

Steps:
1. Open `Discover` → choose `wazuh-alerts-*`.
2. Set time picker to `Last 7 days`.
3. Paste this KQL into the search bar and press Enter:

```
data.win.eventid:4625
```

4. Confirm results include `data.win.eventdata.TargetUserName` and `data.srcip`.
5. Save the search (`Save` → name: `Failed Logins - Windows`).

DSL example (aggregate top failure sources):

```
{
  "size": 0,
  "query": { "term": { "data.win.eventid": 4625 }},
  "aggs": { "top_src": { "terms": { "field": "data.srcip.keyword", "size": 10 }}}
}
```

Use the DSL in Dev Tools or via curl to return the top 10 source IPs for failed logins.

---

## 2) Successful Windows logins (KQL)

KQL:

```
data.win.eventid:4624
```

Tip: combine with `LogonType` to find interactive/RDP logons:

```
data.win.eventid:4624 AND data.win.eventdata.LogonType:10
```

---

## 3) SSH failures (Linux) — KQL and Lucene

KQL (high-level):

```
rule.groups: "sshd" AND rule.level >= 7
```

Lucene equivalent:

```
rule.groups:"sshd" AND rule.level:[7 TO *]
```

Save this search as `SSH Failures` and build a visualization (Terms aggregation) on `data.srcip.keyword` to surface attacking IPs.

---

## 4) Brute-force detection — step-by-step with visualization

Objective: find source IPs with many failed auths in a sliding time window.

Steps to create a dashboard panel (OpenSearch Dashboards / Kibana):
1. Create a saved search for failed logins (e.g., `data.win.eventid:4625` or `rule.groups: "sshd" AND rule.level >= 7`).
2. Go to `Visualize` / `Create visualization` → choose `Lens` or `TSVB`.
3. Add metric: `Top values` (Terms) on `data.srcip.keyword`, size `10`.
4. Add a `Date Histogram` on `@timestamp` for time-series view.
5. Save visualization and add it to a dashboard.

DSL example for top attackers in last 24h:

```
{
  "size": 0,
  "query": { "bool": { "must": [ { "term": { "data.win.eventid": 4625 }}, { "range": { "@timestamp": { "gte": "now-24h" }}} ]}},
  "aggs": { "top_src": { "terms": { "field": "data.srcip.keyword", "size": 20 }}}
}
```

---

## 5) RDP specific monitoring

- Failed RDP logon (KQL): `data.win.eventid:4625 AND data.win.eventdata.LogonType:10`
- Successful RDP logon (KQL): `data.win.eventid:4624 AND data.win.eventdata.LogonType:10`
- Dashboard idea: top RDP-targeted hosts — Terms aggregation on `agent.name.keyword` filtered by the RDP KQL above.

---

## 6) Example: Encoded PowerShell (hunt)

KQL:

```
data.process_name: "powershell.exe" AND data.process_cmdline: "*EncodedCommand*"
```

DSL (match phrase):

```
{"query": {"bool": {"must": [{"match_phrase": {"data.process_cmdline": "EncodedCommand"}}]}}}
```

---

## 7) Saving searches and reusing in dashboards

1. In `Discover`, click `Save` after running a query.
2. In `Visualize` or `Dashboard`, choose `Add from saved searches` or reference saved search by its name in panel query.
3. Pin time range and filters as needed; use pinned filters to keep dashboard-context consistent.

---

## 8) Troubleshooting tips

- If no results: widen the time range, check `wazuh-alerts-*` is selected, inspect a sample document to confirm field names.
- For aggregation returning few items: use `.keyword` fields like `data.srcip.keyword` or `agent.name.keyword`.
- If fields are missing: check `wazuh-archives-*` for raw events.

---

If you'd like, I can now:
- Generate the saved visualization JSON you can import directly into OpenSearch Dashboards, or
- Expand another section (Process Monitoring, Network) into the same step-by-step format.

````
