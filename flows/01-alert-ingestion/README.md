# Flow 1 — Alert Ingestion + Normalisation

> **Receives raw webhook alerts from simulated Wiz, CrowdStrike, and Datadog sources. Normalises them into a standard internal schema. Logs every alert to Google Sheets.**

**Demo video:** [▶ Watch on YouTube](https://youtube.com) *(coming soon)*

---

## What this flow does

```
Webhook POST  ──▶  Detect source  ──▶  Normalise schema  ──▶  Validate  ──▶  Google Sheets log
                        │
                        └──▶  Slack notification (critical/high only)
```

1. **Webhook trigger** — listens for incoming alert payloads on a public n8n URL
2. **Source detection** — identifies whether the alert came from Wiz, CrowdStrike, Datadog, or a generic format by inspecting the payload structure
3. **Schema normalisation** — maps each source's field names to the standard internal schema (see root `README.md`)
4. **Severity validation** — rejects malformed payloads with a 400 response; passes valid ones forward
5. **Google Sheets log** — appends a new row to the alert log with all normalised fields + ingestion timestamp
6. **Slack notification** — sends a formatted message to `#security-alerts` for `critical` and `high` severity only

---

## Credentials you need

| Credential | Where to get it | n8n credential type |
|-----------|----------------|-------------------|
| Google Sheets OAuth | [Google Cloud Console](https://console.cloud.google.com) | Google Sheets OAuth2 |
| Slack Bot Token | [Slack API Apps](https://api.slack.com/apps) — Bot Token Scopes: `chat:write` | Slack API |

---

## Google Sheets setup

Create a new Google Sheet with the following headers in row 1 (exact column order matters):

```
alert_id | source | severity | title | resource_id | resource_type | cloud_account | region | owner_tag | env_tag | timestamp | ingested_at | raw_payload
```

Copy the Sheet ID from the URL:
`https://docs.google.com/spreadsheets/d/SHEET_ID_HERE/edit`

Paste it into the Google Sheets node inside the workflow.

---

## Importing the workflow

1. In n8n Cloud, go to **Workflows → Import from file**
2. Upload `workflow.json` from this directory
3. Open the workflow, click each node with a credential warning, and assign your credentials
4. Activate the workflow (toggle top-right)
5. Copy the webhook URL from the Webhook trigger node

---

## Testing with sample payloads

Three sample payloads are included, one per alert source.

### Wiz alert
```bash
curl -X POST YOUR_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d @sample-payloads/wiz-alert.json
```

### CrowdStrike alert
```bash
curl -X POST YOUR_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d @sample-payloads/crowdstrike-alert.json
```

### Datadog alert
```bash
curl -X POST YOUR_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d @sample-payloads/datadog-alert.json
```

Expected result: a new row in your Google Sheet within 2–3 seconds, and a Slack message if severity is `critical` or `high`.

---

## Normalisation logic

The code node maps source-specific fields to the internal schema:

| Internal field | Wiz source field | CrowdStrike source field | Datadog source field |
|---------------|-----------------|------------------------|-------------------|
| `alert_id` | `issueId` | `id` | `id` |
| `severity` | `severity` | `severity` (mapped) | `priority` (mapped) |
| `title` | `control.name` | `name` | `name` |
| `resource_id` | `entitySnapshot.id` | `device_id` | `host` |
| `resource_type` | `entitySnapshot.type` | `platform` | `source_type_name` |
| `cloud_account` | `entitySnapshot.subscriptionId` | `account_id` | `org_name` |
| `region` | `entitySnapshot.region` | `region` | `tags.region` |

Severity mapping (CrowdStrike uses numeric 1–100, Datadog uses P1–P5):

```
CrowdStrike: 75–100 → critical, 50–74 → high, 25–49 → medium, 0–24 → low
Datadog:     P1 → critical, P2 → high, P3 → medium, P4/P5 → low
```

---

## What to extend in production

- Add PagerDuty notification for `critical` alerts
- Add a deduplication check before writing to Sheets (prevent duplicate alert_ids)
- Replace Google Sheets with a proper database (Supabase, PostgreSQL) for scale
- Add IP geolocation enrichment for network-based alerts
- Store the raw payload in S3 for audit trail

---

## Next flow

→ [Flow 2: AI-Powered Triage + Scoring](../02-ai-triage/README.md)
