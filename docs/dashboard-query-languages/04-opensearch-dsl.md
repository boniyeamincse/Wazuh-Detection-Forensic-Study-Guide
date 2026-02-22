# 04 - OpenSearch Query DSL Master Guide ⚡

OpenSearch Query DSL (Domain Specific Language) is the most powerful way to query data in Wazuh. It uses a **JSON-based** structure and is the foundation for all Dashboard visualizations and API-based automation.

---

## 🏗️ Core Query DSL Components

### 1. The `bool` Query
The most common structure for combining multiple conditions.
- **`must`**: Must match (contributes to score).
- **`filter`**: Must match (does not contribute to score, **cached and faster**).
- **`should`**: If it matches, the score increases.
- **`must_not`**: Must not match.

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "rule.id": "5710" } }
      ],
      "filter": [
        { "range": { "rule.level": { "gte": 10 } } }
      ]
    }
  }
}
```

---

## 🚀 Specialized Query Types

### 1. `wildcard` Query
```json
{
  "query": {
    "wildcard": {
      "data.win.eventdata.image.keyword": "*\\powershell.exe"
    }
  }
}
```

### 2. `regexp` Query
```json
{
  "query": {
    "regexp": {
      "data.linux.process.keyword": ".*sh"
    }
  }
}
```

### 3. `exists` Query (Checking for presence of field)
```json
{
  "query": {
    "exists": {
      "field": "data.win.eventdata.hashes"
    }
  }
}
```

---

## 📊 Aggregations (Visualizing Threats)

Aggregations allow you to group data and perform calculations (e.g., "Top 10 Source IPs").

### Example: Top 5 Agents by Alert Volume
```json
{
  "size": 0,
  "aggs": {
    "top_agents": {
      "terms": {
        "field": "agent.name.keyword",
        "size": 5
      }
    }
  }
}
```

---

## 🧪 Detection Engineering Examples

### Multi-Stage Attack Detection
Detect a situation where a user logs in (4624) and then immediately creates a new user (4720) within the same context.

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "bool": {
            "should": [
              { "term": { "rule.id": "60106" } },
              { "term": { "rule.id": "60113" } }
            ],
            "minimum_should_match": 1
          }
        }
      ],
      "filter": [
        { "range": { "@timestamp": { "gte": "now-30m" } } }
      ]
    }
  }
}
```

---

## 🛠️ API & Curl Operations

To query the indexer directly for automated triage scripts:

```bash
# Query via Wazuh Indexer API
curl -X GET "https://localhost:9200/wazuh-alerts-*/_search?pretty" \
     -k -u admin:admin \
     -H 'Content-Type: application/json' \
     -d'
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "rule.id": "5710" } },
        { "range": { "timestamp": { "gte": "now-1h" } } }
      ]
    }
  }
}'
```

---

## 💡 Pro-Tip: Filter vs Must
Detection Engineers should **ALWAYS use `filter`** instead of `must` for fields like `rule.id`, `agent.id`, and `srcip`. Scores are irrelevant in SOC triage, and filters are significantly faster and lighter on the indexer's resources.
