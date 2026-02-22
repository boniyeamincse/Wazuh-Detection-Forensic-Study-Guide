# 07 - Performance & Scaling in Large Environments ⚡

Scaling a Wazuh environment to handle thousands of agents and millions of events per day requires precise query tuning and index management.

---

## 🏎️ Query Optimization Strategies

### 1. The "Keyword" Principle
Always append `.keyword` to your field names when performing exact matches.
- **Bad:** `agent.name: "prod-server"` (Slow, uses analyzer)
- **Good:** `agent.name.keyword: "prod-server"` (Fast, binary match)

### 2. Avoid Leading Wildcards
`*` at the start of a query is a performance killer.
- **Bad:** `data.win.eventdata.image: "*powershell.exe"`
- **Good:** Use a specific path or the `match_phrase` query in DSL.

### 3. Filter vs Query
In Dashboards and DSL:
- **Query (`must`):** Used when you care about the relevance score.
- **Filter (`filter`):** Used when you just want to know if the document matches. **Filters are cached and significantly faster.**

---

## 📂 Index & Shard Management

### 1. Shard Sizing
- **Rule of Thumb:** Keep shard sizes between **20GB and 50GB**.
- Shards that are too small create overhead; shards that are too large make recovery and rebalancing slow.

### 2. Index Lifecycle Management (ISM)
Automate the movement of data between storage tiers:
- **Hot Tier:** SSDs for the last 7-14 days of high-speed querying.
- **Warm Tier:** HDDs for 15-90 days of forensics.
- **Cold Tier/Archive:** Long-term storage for compliance.

---

## 📊 Dashboard Load Optimization

### 1. Reduce the Number of Visualizations
Each chart on a dashboard triggers a separate query to the indexer. Limit your main "SOC Overview" dashboard to 8-10 high-value charts.

### 2. Time-Range Defaults
Set default dashboards to load only the last **15 or 30 minutes** of data. Avoid "Last 24 Hours" as a default for global views.

### 3. Use "Refresh on Demand"
Disable auto-refresh on dashboards with complex aggregations. Allow analysts to refresh manually when needed.

---

## 🔥 Large Environment Tuning Tips

| Component | Recommendation |
| :--- | :--- |
| **Circuit Breaker** | Set `indices.breaker.total.limit` to avoid OOM crashes during heavy queries. |
| **Search Concurrency** | Limit the number of concurrent searches per node to prevent CPU saturation. |
| **Field Mapping** | Disable `text` indexing for fields that only need exact match (`keyword`). |
| **Refresh Interval** | Increase index refresh interval (e.g., to 30s) to improve ingest performance. |

---

## 🛠️ Performance Audit Query
```bash
curl -X GET "https://localhost:9200/_tasks?actions=*search&detailed" -k -u admin:admin
```
This will list all running search tasks and the time they have been active.

---

**Previous: [06 - SOC Playbooks](06-soc-playbooks.md)** | **Next: [08 - Common Mistakes](08-common-mistakes.md)**

[Return to Index](../../README.md)
