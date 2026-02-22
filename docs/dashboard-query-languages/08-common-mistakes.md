# 08 - Common Mistakes & Troubleshooting ⚠️

Even experienced analysts make mistakes that lead to empty results or performance degradation. This document lists the most frequent pitfalls and how to fix them.

---

## ❌ 1. Case Sensitivity Confusion

**The Mistake:** Searching for `image: "powershell.exe"` when the log contains `PowerShell.exe`.
- **Fact:** `keyword` fields are **case-sensitive**. `text` fields are usually case-insensitive.
- **Fix:** Check the mapping. If it's a keyword, use the exact case or use a wildcard `*owershell.exe`.

---

## ❌ 2. Space in Unquoted Values

**The Mistake:** `agent.name: My Prod Server`
- **Result:** KQL will search for `agent.name: "My"` AND the word `"Prod"` AND the word `"Server"` anywhere in the document.
- **Fix:** Always wrap values with spaces in double quotes: `agent.name: "My Prod Server"`.

---

## ❌ 3. Misunderstanding the `NOT` Operator

**The Mistake:** `NOT rule.level: 3`
- **Fact:** On nested or multi-value fields, `NOT` might behave unexpectedly.
- **The Better Way:** Use `!(rule.level: 3)` for clarity or ensure you are targeting a single-value field.

---

## ❌ 4. Overusing Leading Wildcards

**The Mistake:** Searching `*mimikatz*` across all fields.
- **Impact:** This forces the indexer to scan every single term in the index. It can lag the dashboard for all users.
- **Fix:** Target a specific field: `data.win.eventdata.commandLine: *mimikatz*`.

---

## ❌ 5. Quoting Numerical Ranges

**The Mistake:** `rule.level: ["10" TO "15"]`
- **Result:** This treats the levels as strings, which might fail or produce incorrect results in a numeric sort.
- **Fix:** Do not quote numbers in ranges: `rule.level: [10 TO 15]`.

### ❌ 6. Case Sensitivity in Keywords
KQL is case-insensitive for some operations, but Lucene and DSL are **strictly case-sensitive** for `.keyword` fields.
- **Wrong:** `agent.name.keyword: "Windows-Server"` (if the actual name is `windows-server`)
- **Fix:** Always check the exact case in the Discover tab.

---

## 🛠️ Deep Troubleshooting: Mapper Parsing Exception

This is the most common error when a new log source is added. It happens when two different logs send different data types for the same field.

**Case:** `data.id` is expected to be a `long` (number), but a log sends "N/A" (string).
- **Result:** The Indexer rejects the log entirely.
- **Fix:** 
    1. Check `GET /wazuh-alerts-*/_mapping` to find the conflicting field.
    2. Create a **Custom Mapping** or use a **Wazuh Decoder** to force the field into a specific type.
    3. Re-index the data or wait for the next index creation (midnight).

---

## ❌ 7. The "Analyzed" Field Trap

**The Mistake:** Trying to aggregate on a `text` field (like `full_log`).
- **Error:** "Fielddata is disabled on text fields by default."
- **Fix:** You cannot aggregate on text fields. You must create a custom sub-field of type `keyword` or use a field already mapped as a keyword.

---

## 🛠️ Troubleshooting Checklist (Empty Results?)

If your query returns no results but you KNOW the data is there:
1.  **Check the Time Range:** Is it set to "Last 15 minutes" while the event happened 20 minutes ago?
2.  **Check the Index Pattern:** Are you looking in `wazuh-alerts-*` when the data is in `wazuh-archives-*`?
3.  **Check for Hidden Characters:** Copy-pasting from some editors can introduce non-printing characters.
4.  **Verify Field Mapping:** Use the "Dev Tools" in Dashboards to check the mapping: `GET /wazuh-alerts-*/_mapping`.

---

**Previous: [07 - Performance & Scaling](07-performance-tuning.md)** | **Next: [09 - Quick Reference](09-quick-reference.md)**

[Return to Index](../../README.md)
