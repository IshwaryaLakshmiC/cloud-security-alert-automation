# Flow 4 — Weekly Security Posture Digest

> **Every Monday at 9am, queries the full alert log and generates a structured security posture report — sent as a Slack block message and HTML email. Covers MTTR, true positive rate, top recurring findings, and open vs resolved ratio.**

**Demo video:** [▶ Watch on YouTube](https://youtube.com) *(coming soon)*

---

## What this flow does

```
Schedule (Monday 09:00)
        │
        ▼
Read all alerts from Google Sheets (past 7 days)
        │
        ▼
Calculate metrics:
  • Total alerts ingested
  • True positive rate
  • Mean time to triage (MTTT)
  • Top 3 recurring finding types
  • Alerts by severity breakdown
  • Open vs resolved ratio
        │
        ├──▶  Slack — formatted block message with colour-coded summary
        │
        └──▶  Gmail — HTML digest email
```

---

## Metrics calculated

| Metric | Formula |
|--------|---------|
| Total alerts | Count of all rows with `ingested_at` in last 7 days |
| True positive rate | `true_positive` count ÷ scored alerts × 100 |
| Mean time to triage | Average of (`scored_at` − `ingested_at`) in minutes |
| Top findings | Most frequent `title` values, grouped and counted |
| Open tickets | Count where `jira_status = open` |
| Critical/high ratio | Critical + high alerts ÷ total alerts |

---

## Slack output format

The Slack message uses Block Kit for a rich, scannable layout:

```
📊 Weekly Security Posture — Week of Jun 9, 2026
─────────────────────────────────────────────────
Total alerts      47    True positive rate   68%
Open tickets      12    Mean time to triage  3.2 min
─────────────────────────────────────────────────
🔴 Critical  8    🟠 High  19    🟡 Medium  14    ⚪ Low  6

Top 3 recurring findings:
  1. S3 bucket publicly accessible (11 alerts)
  2. IAM privilege escalation detected (7 alerts)
  3. Exposed service port on EC2 instance (5 alerts)
─────────────────────────────────────────────────
View full Jira board → https://yourteam.atlassian.net/jira/SEC
```

---

## Credentials you need

| Credential | Where to get it | n8n credential type |
|-----------|----------------|-------------------|
| Google Sheets OAuth | Same as Flow 1 | Google Sheets OAuth2 |
| Slack Bot Token | Same as Flow 1 | Slack API |
| Gmail OAuth | [Google Cloud Console](https://console.cloud.google.com) | Gmail OAuth2 |

---

## Importing the workflow

1. Import `workflow.json` into n8n Cloud
2. Assign credentials: Google Sheets, Slack, Gmail
3. Update `YOUR_GOOGLE_SHEET_ID` in the Sheets node
4. Update the digest email recipient in the Gmail node
5. Update your Jira board URL in the Slack node
6. Activate — first run is next Monday 9am, or execute manually to test

---

## What to extend in production

- Add a trend line: compare this week vs last week for each metric
- Add a "most improved" metric — resources that had alerts resolved vs still open
- Generate a Confluence page with the digest for audit trail
- Add per-team breakdown (group by `owner_tag`)
- Attach a CSV export of all open tickets to the email

---

## The complete flow

This is the final piece of the four-flow system:

| Flow | Trigger | Purpose |
|------|---------|---------|
| 1 — Ingestion | Webhook | Receive + normalise alerts |
| 2 — AI Triage | Every 5 min | Score + classify unscored alerts |
| 3 — Jira Ticketing | Every 10 min | Create/update tickets for scored alerts |
| 4 — Weekly Digest | Monday 9am | Report on posture across the week |
