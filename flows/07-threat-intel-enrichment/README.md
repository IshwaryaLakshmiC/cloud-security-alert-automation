# Flow 7 — Threat Intelligence Enrichment (OpenRouter + MITRE + NVD)

> **For each confirmed true positive, hits the MITRE ATT&CK STIX API and NIST NVD CVE API, then uses Qwen to synthesise matched TTPs, related CVEs, threat actor groups, and detection gaps.**

**Model:** `qwen/qwen2-7b-instruct:free` via OpenRouter  
**External APIs:** MITRE ATT&CK (free, no key), NVD CVE API (free, no key)

## How it works

1. Reads true positives with `risk_score >= 6` and no `threat_intel_enriched` timestamp
2. Calls MITRE ATT&CK search API with alert title + resource type
3. Calls NVD CVE API with alert title as keyword
4. Merges both API responses and feeds them into Qwen with the alert context
5. Parses: TTP IDs, tactic, related CVEs, threat actors, detection gap
6. Writes enrichment back to Sheets + posts to Slack

## New Google Sheets columns needed

```
mitre_ttp_ids | mitre_tactic | related_cves | threat_actors | detection_gap | intel_summary | threat_intel_enriched | intel_model
```

## APIs used (both free, no auth required)

- MITRE ATT&CK: `https://attack.mitre.org/api/`
- NVD CVE: `https://services.nvd.nist.gov/rest/json/cves/2.0`
