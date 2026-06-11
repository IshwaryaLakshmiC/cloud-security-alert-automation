# Flow 2 — AI-Powered Alert Triage + Severity Scoring

> **Polls the Google Sheets alert log every 5 minutes. For each unscored alert, calls an LLM to assess blast radius, resource exposure, and attack pattern — assigning a 1–10 risk score and a triage decision. Updates the sheet in place.**

**Demo video:** [▶ Watch on YouTube](https://youtube.com) *(coming soon)*

---

## What this flow does

```
Schedule (every 5 min)
        │
        ▼
Read Google Sheets  ──▶  Filter unscored alerts  ──▶  For each alert:
                                                            │
                                                   Build LLM prompt
                                                            │
                                                   OpenAI / Claude API
                                                            │
                                                   Parse structured response
                                                            │
                                          ┌─────────────────────────────┐
                                          │  score + decision + rationale│
                                          └─────────────────────────────┘
                                                            │
                                                   Update row in Sheets
                                                            │
                                          If score ≥ 8  ──▶  Slack urgent ping
```

---

## Triage output fields

Each alert gets three new fields written back to the sheet:

| Field | Type | Description |
|-------|------|-------------|
| `risk_score` | Integer 1–10 | Overall risk. 9–10 = immediate action, 7–8 = same day, ≤6 = scheduled |
| `triage_decision` | Enum | `true_positive` · `false_positive` · `needs_investigation` |
| `triage_rationale` | String | 2–3 sentence LLM explanation of the score |
| `scored_at` | ISO 8601 | Timestamp of when triage ran |

---

## The prompt engineering

The system prompt instructs the LLM to act as a senior cloud security engineer and evaluate each alert on four dimensions:

1. **Blast radius** — how many systems/users could be affected if this is real
2. **Resource sensitivity** — is this prod vs dev, tagged sensitive data, public-facing
3. **Attack pattern** — does this match a known TTP (MITRE ATT&CK)
4. **Exploitability** — how easy is it to exploit right now vs theoretical

The response is requested as strict JSON to enable reliable parsing:

```json
{
  "risk_score": 9,
  "triage_decision": "true_positive",
  "triage_rationale": "Public S3 bucket containing prod customer data with no access logging is an active data exposure risk. Blast radius is high given the sensitivity classification. No additional steps needed to exploit — the bucket is directly accessible."
}
```

The prompt is constructed dynamically from the normalised alert fields — source, severity, title, description, resource type, environment tag, and owner tag all feed into the context.

---

## Google Sheets — additional columns needed

Add these columns to your existing sheet from Flow 1 (append to the right):

```
risk_score | triage_decision | triage_rationale | scored_at
```

The flow reads all rows where `scored_at` is empty and writes back to those specific rows.

---

## Credentials you need

| Credential | Where to get it | n8n credential type |
|-----------|----------------|-------------------|
| Google Sheets OAuth | Same as Flow 1 | Google Sheets OAuth2 |
| OpenAI API Key | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) | OpenAI API |
| Slack Bot Token | Same as Flow 1 | Slack API |

> **Cost estimate:** At GPT-4o-mini pricing (~$0.15/1M input tokens), scoring 100 alerts costs roughly $0.02. Safe to run every 5 minutes.

---

## Importing the workflow

1. In n8n Cloud, go to **Workflows → Import from file**
2. Upload `workflow.json` from this directory
3. Assign credentials to: Google Sheets node, OpenAI node, Slack node
4. Update `YOUR_GOOGLE_SHEET_ID` in both Sheets nodes
5. Activate — it will begin polling immediately

---

## Testing

First, make sure Flow 1 has written at least one alert to your sheet.

To trigger Flow 1 quickly with a test payload:
```bash
curl -X POST YOUR_FLOW1_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d @../01-alert-ingestion/sample-payloads/wiz-alert.json
```

Then either wait up to 5 minutes for the schedule, or manually execute Flow 2 from the n8n canvas.

Expected result: the alert row in Sheets gets `risk_score`, `triage_decision`, `triage_rationale`, and `scored_at` filled in. If `risk_score` ≥ 8, a Slack message fires to `#security-alerts`.

---

## Sample triage outputs

### Wiz — public S3 bucket (critical)
```json
{
  "risk_score": 9,
  "triage_decision": "true_positive",
  "triage_rationale": "Public S3 bucket containing prod customer data with no access controls is an active data exposure. The sensitivity tag and prod environment classification make this a P1. Immediately restrict bucket ACL and enable access logging."
}
```

### CrowdStrike — Mimikatz detected (critical)
```json
{
  "risk_score": 10,
  "triage_decision": "true_positive",
  "triage_rationale": "Mimikatz execution on a prod server indicates active credential harvesting. This matches MITRE ATT&CK T1003.001 (LSASS Memory). Assume lateral movement is in progress — isolate the host immediately."
}
```

### Datadog — IAM privilege escalation (high)
```json
{
  "risk_score": 8,
  "triage_decision": "needs_investigation",
  "triage_rationale": "A service account attaching AdministratorAccess to itself is a high-confidence privilege escalation indicator. However, the source IP and user agent suggest it may be a misconfigured deployment pipeline rather than an adversary. Verify with the platform-security team before revoking."
}
```

---

## What to extend in production

- Add MITRE ATT&CK technique tagging to the triage output
- Feed historical false positive patterns back into the prompt as few-shot examples
- Add a confidence score alongside the risk score
- Route `needs_investigation` alerts to a separate Slack thread for analyst review
- Implement feedback loop — analyst marks FP/TP in Sheets, prompt improves over time

---

## Next flow

→ [Flow 3: Jira Ticket Creation + Deduplication](../03-jira-ticketing/README.md)
