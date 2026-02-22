# Wazuh Detection & Forensic Study Guide 🛡️🎓

[![Wazuh Version](https://img.shields.io/badge/Wazuh-4.7.x-blue.svg)](https://wazuh.com/)
[![Focus](https://img.shields.io/badge/Focus-Detection_&_Forensics-orange.svg)]()
[![Modules](https://img.shields.io/badge/Modules-14-purple.svg)]()
[![Author](https://img.shields.io/badge/Author-Boni_Yeamin-blue.svg)](mailto:boniyeamin.cse@gmail.com)
[![SOC Ready](https://img.shields.io/badge/SOC-Ready-red.svg)](docs/dashboard-query-languages/06-soc-playbooks.md)

This repository is a **Structured Study Guide** for mastering Wazuh Dashboard query languages, log analysis, and digital forensics. Whether you are a student, a junior SOC analyst, or a seasoned detection engineer, this curriculum is designed to elevate your technical hunting skills.

---

## 👨‍💻 Author Information

- **Author:** Boni Yeamin
- **Email:** [boniyeamin.cse@gmail.com](mailto:boniyeamin.cse@gmail.com)
- **Role:** Security Researcher / Detection Engineer

---

## 🧭 The Learning Path (Index)

The documentation is divided into four critical domains to guide your study from zero to forensic expert.

### 📍 Domain 1: The Fundamentals
Master how Wazuh processes data and the most intuitive query language.
1.  **[01. Architecture & Alert Structure](docs/dashboard-query-languages/01-overview.md)** - Indexers, Index patterns, and JSON structures.
2.  **[02. KQL Master Guide](docs/dashboard-query-languages/02-kql-master.md)** - **150+ queries** for baseline Windows/Linux detection.
3.  **[11. Log Types & Field Mapping](docs/dashboard-query-languages/11-log-types-dictionary.md)** - Understanding how your logs are normalized.

### 📍 Domain 2: Super-User Syntax
Master the technical power of Regex and JSON-based advanced querying.
4.  **[03. Lucene Advanced Guide](docs/dashboard-query-languages/03-lucene-advanced.md)** - **100+ queries** focusing on Regex and Fuzzy logic.
5.  **[04. OpenSearch Query DSL](docs/dashboard-query-languages/04-opensearch-dsl.md)** - Master JSON queries and API automation.

### 📍 Domain 3: Proactive Hunting & Forensics
Transition from alert triage to proactive threat discovery.
6.  **[05. Threat Hunting Pack](docs/dashboard-query-languages/05-threat-hunting-pack.md)** - **MITRE ATT&CK** mapped hunting logic.
7.  **[10. Deep Forensic Querying](docs/dashboard-query-languages/10-forensic-analysis.md)** - Persistence, Registry, and Network forensics.
8.  **[12. Investigative Commands Library](docs/dashboard-query-languages/12-investigative-commands.md)** - What commands to hunt for (and why).

### 📍 Domain 4: Enterprise Operations & Labs
Scale your skills for billion-event environments and hands-on practice.
9.  **[06. SOC Playbooks](docs/dashboard-query-languages/06-soc-playbooks.md)** - Scenario walkthroughs: Ransomware, Breach, Lateral Movement.
10. **[07. Performance & Scaling](docs/dashboard-query-languages/07-performance-tuning.md)** - Optimized querying for high-volume SOCs.
11. **[08. Common Mistakes](docs/dashboard-query-languages/08-common-mistakes.md)** - The "What NOT to do" guide.
12. **[09. Quick Reference](docs/dashboard-query-languages/09-quick-reference.md)** - One-page mid-incident rapid lookup.
13. **[13. Wazuh Lab Setup Guide](docs/dashboard-query-languages/13-lab-setup.md)** - Build your own Docker or OVA lab.
14. **[14. External Learning Resources](docs/dashboard-query-languages/14-learning-resources.md)** - Where to go next for Blue Team mastery.

---

## 🛡️ Why This Project?
- **Forensic Focus**: Moves beyond "was there a login" to "what did they do to the registry".
- **Study-Ready**: Categorized field mappings and investigative command libraries.
- **250+ Production Queries**: Each query is tested and ready for production deployment.
- **MITRE Mapped**: Learn to map your hunting activity to real attacker tactics.

---

## 🤝 Community & Contribution
This guide is open-source under the MIT License. If you find a useful query, please contribute via a Pull Request!

---

**Developed for the Cybersecurity Community**
*Version 1.2 - The Author & Mastery Edition*
