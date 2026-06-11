# Cloud Security Alert Intelligence — n8n Automation Portfolio

> **4 production-grade n8n workflows** that turn raw cloud security alerts into prioritised, actionable tickets — reducing mean time to triage from hours to minutes.

Built by [Ishwarya Lakshmi](https://github.com/IshwaryaLakshmiC) as a public portfolio demonstrating security automation engineering for Solutions Engineer roles in cloud security.

---

## The problem this solves

Security teams at mid-market companies receive 500+ cloud alerts per day. Most are noise. The ones that aren't get buried. The result: critical misconfigurations sit unresolved for days, analysts burn out on manual triage, and MTTR (mean time to remediate) stays stubbornly high.

This project automates the full alert lifecycle — ingestion → AI triage → ticketing → reporting — using n8n as the orchestration layer.

---

## Flows

| # | Flow | What it does | Demo |
|---|------|-------------|------|
| 1 | [Alert Ingestion + Normalisation](./flows/01-alert-ingestion/) | Receives webhooks from simulated Wiz/CrowdStrike/Datadog alerts, normalises schema, logs to Google Sheets | [▶ Watch](https://youtube.com) |
| 2 | [AI-Powered Triage + Scoring](./flows/02-ai-triage/) | Scores each alert 1–10 using LLM reasoning on blast radius, resource exposure, and historical pattern | [▶ Watch](https://youtube.com) |
| 3 | [Jira Ticket Creation + Deduplication](./flows/03-jira-ticketing/) | Creates or updates Jira tickets for high-priority alerts; deduplicates by resource + finding type | [▶ Watch](https://youtube.com) |
| 4 | [Weekly Security Posture Digest](./flows/04-weekly-digest/) | Monday 9am Slack + email summary: MTTR, true positive rate, top recurring findings | [▶ Watch](https://youtube.com) |

---

## Architecture

```
Webhook (simulated alert source)
        │
        ▼
┌─────────────────────────┐
│  Flow 1: Ingestion      │  ── normalise ──▶  Google Sheets (alert log)
└─────────────────────────┘
        │
        ▼ (schedule, every 5 min)
┌─────────────────────────┐
│  Flow 2: AI Triage      │  ── score + classify ──▶  update Google Sheets
└─────────────────────────┘
        │
        ▼ (high priority only)
┌─────────────────────────┐
│  Flow 3: Jira Ticketing │  ── create / update ──▶  Jira + Slack alert
└─────────────────────────┘
        │
        ▼ (weekly schedule)
┌─────────────────────────┐
│  Flow 4: Weekly Digest  │  ── summarise ──▶  Slack block + Gmail
└─────────────────────────┘
```

---

## Alert schema

All flows use a normalised internal alert schema. Source alerts from Wiz, CrowdStrike, and Datadog are mapped to this format in Flow 1.

```json
{
  "alert_id": "string",
  "source": "wiz | crowdstrike | datadog | generic",
  "severity": "critical | high | medium | low | info",
  "title": "string",
  "description": "string",
  "resource_id": "string",
  "resource_type": "ec2 | s3 | iam | eks | rds | lambda | other",
  "cloud_account": "string",
  "region": "string",
  "tags": { "owner": "string", "env": "prod | staging | dev" },
  "timestamp": "ISO 8601",
  "raw_payload": {}
}
```

---

## Tools used

| Tool | Purpose | Free tier |
|------|---------|-----------|
| n8n Cloud | Workflow orchestration | Yes (limited executions) |
| Google Sheets | Alert log / state store | Yes |
| OpenAI API | Alert triage + scoring | Pay-per-use (pennies/run) |
| Jira | Ticket creation + deduplication | Yes (up to 10 users) |
| Slack | Notifications + digest | Yes |
| Gmail | Weekly digest email | Yes |

---

## Running locally

Each flow directory contains:
- `workflow.json` — importable n8n workflow export
- `README.md` — setup guide, credential requirements, test payloads
- `sample-payload.json` — test webhook data you can send directly

### Import a flow into n8n Cloud

1. Open your n8n Cloud instance
2. Go to **Workflows → Import from file**
3. Upload the `workflow.json` from the flow directory
4. Follow the flow's `README.md` to configure credentials

---

## YouTube demo series

Each flow has a companion video covering:
- What problem it solves
- Live walkthrough of the n8n canvas
- Test run with a real payload
- What you'd extend in production

→ [Playlist link — coming soon]

---

## About

Built to demonstrate automation engineering, security domain knowledge, and technical communication — the core skills of a Solutions Engineer in cloud security.

**Ishwarya Lakshmi C** — Senior DevOps / Platform Engineer transitioning to Solutions Engineering.
[GitHub](https://github.com/IshwaryaLakshmiC) · [LinkedIn](https://linkedin.com/in/) · [Website — coming soon]
