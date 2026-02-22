# Wazuh Administrative Commands — Cheatsheet

A compact reference of common administrative commands for Wazuh Manager, Agents, API, and OpenSearch/Indexing components. Replace placeholders (e.g., `MANAGER`, `AGENT_ID`) with your environment values.

## TL;DR

- Use `systemctl` to control services, `journalctl` and log files to troubleshoot, `manage_agents` and `agent-auth` for agent lifecycle, and the Wazuh API (port `55000`) for scripted operations.

---

## Service control

- Check service status (replace service name if different):

```
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-agent
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

- Restart / start / stop services:

```
sudo systemctl restart wazuh-manager
sudo systemctl start wazuh-agent
sudo systemctl stop wazuh-dashboard
```

- View live logs from systemd:

```
sudo journalctl -u wazuh-manager -f
sudo journalctl -u wazuh-agent -f
```

> Note: Service names may vary by installation. Run `systemctl list-units | grep -i wazuh` to confirm.

---

## Wazuh Manager files & logs

- Common paths (default package install):

- Manager and agent binary/config: `/var/ossec`
- Main log: `/var/ossec/logs/ossec.log`
- Alerts indexer outputs: check Wazuh Indexer/OpenSearch logs

- Tail manager log:

```
sudo tail -n 200 /var/ossec/logs/ossec.log
sudo tail -f /var/ossec/logs/ossec.log
```

---

## Agent management

- Interactive agent management (adds/removes agents):

```
sudo /var/ossec/bin/manage_agents
```

- Manually register an agent to manager (run on the agent):

```
sudo /var/ossec/bin/agent-auth -m MANAGER_IP
```

- List agents via the Wazuh API (replace `USERNAME:PASS` and `MANAGER`):

```
curl -u USERNAME:PASSWORD -k "https://MANAGER:55000/agents"
```

- Remove agent via API:

```
curl -u USER:PASS -k -X DELETE "https://MANAGER:55000/agents/AGENT_ID"
```

---

## Rule testing & logs

- Test raw log input against rules (logtest):

```
sudo /var/ossec/bin/ossec-logtest
```

- Reload rules without restarting manager (if supported):

```
sudo systemctl reload wazuh-manager
```

---

## Wazuh API (examples)

- Get agent status:

```
curl -u foo:bar -k "https://MANAGER:55000/agents"
```

- Get specific agent info:

```
curl -u foo:bar -k "https://MANAGER:55000/agents/AGENT_ID"
```

- Get rules or decoders via API (useful for automation):

```
curl -u foo:bar -k "https://MANAGER:55000/rules"
curl -u foo:bar -k "https://MANAGER:55000/decoders"
```

---

## OpenSearch / Indexer quick commands

- Check indices (use OpenSearch Dashboards Dev Tools or curl against OpenSearch API):

```
curl -u es_user:es_pass -k "https://OPENSEARCH:9200/_cat/indices?v"
```

- Search recent alerts (example):

```
curl -u es_user:es_pass -k -X GET "https://OPENSEARCH:9200/wazuh-alerts-*/_search" -H 'Content-Type: application/json' -d'
{"query": {"match_all": {}}, "size": 10, "sort": [{"@timestamp": {"order": "desc"}}]}
'
```

---

## Backup & Restore (high level)

- Backup Wazuh Manager config and keys:

```
sudo tar czf /root/wazuh-manager-backup-$(date +%F).tgz /var/ossec
```

- OpenSearch index snapshots: use OpenSearch snapshot API and repository (snapshot to shared FS or S3). See OpenSearch docs for snapshots.

---

## Useful troubleshooting commands

- Check recent manager errors (journal + ossec log):

```
sudo journalctl -u wazuh-manager -n 200 --no-pager
sudo tail -n 200 /var/ossec/logs/ossec.log
```

- Check disk usage (indexes can grow quickly):

```
df -h
du -sh /var/ossec /var/lib/opensearch
```

---

## Notes & best practices

- Always test commands in a staging environment before production.
- Use `.keyword` fields for aggregations in dashboards.
- Rotate and snapshot OpenSearch indices regularly to preserve storage.
- Secure the Wazuh API with strong credentials and TLS (port 55000).

---

[Return to Index](../README.md)
