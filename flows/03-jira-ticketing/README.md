# Flow 3 — Jira Ticket Creation + Deduplication

> **Picks up high-priority triaged alerts from Google Sheets. Checks Jira for an existing open ticket on the same resource + finding. If one exists, updates it. If not, creates a new ticket with pre-populated remediation steps, SLA due date, and owner assignment.**

**Demo video:** [▶ Watch on YouTube](https://youtube.com) *(coming soon)*

---

## What this flow does

```
Schedule (every 10 min)
        │
        ▼
Read Sheets  ──▶  Filter: score ≥ 7, no jira_ticket_id yet
        │
        ▼
Search Jira for existing ticket (same resource_id + title)
        │
        ├── Found ──▶  Add comment + bump priority  ──▶  Update Sheets with ticket URL
        │
        └── Not found ──▶  Create new Jira ticket  ──▶  Update Sheets with ticket URL
                                    │
                                    └──▶  Slack notification with ticket link
```

---

## Deduplication logic

Before creating any ticket, the flow queries Jira using JQL:

```
project = SEC AND status != Done AND summary ~ "resource_id" AND labels = "auto-triaged"
```

If an open ticket already exists for the same resource:
- Adds a comment with the new alert details and updated risk score
- Bumps the priority if the new score is higher than the existing ticket's priority
- Updates the `last_seen` custom field

This prevents ticket sprawl — a noisy S3 bucket doesn't generate 50 Jira tickets, it generates one that accumulates evidence.

---

## Jira ticket structure

New tickets are created with the following fields:

| Field | Value |
|-------|-------|
| Project | `SEC` (configurable) |
| Issue type | `Bug` |
| Summary | `[AUTO] {source} — {title}` |
| Priority | Mapped from risk_score (see below) |
| Labels | `auto-triaged`, `{source}`, `{env_tag}` |
| Description | Full alert details + AI rationale + remediation steps |
| Due date | SLA based on severity (critical = today, high = +1 day, medium = +3 days) |
| Assignee | Looked up by `owner_tag` (requires team mapping config) |

### Priority mapping

| Risk score | Jira priority |
|-----------|--------------|
| 9–10 | Highest |
| 7–8 | High |
| 5–6 | Medium |
| 3–4 | Low |
| 1–2 | Lowest |

---

## Google Sheets — additional columns needed

Add these columns to your sheet:

```
jira_ticket_id | jira_ticket_url | jira_status | ticket_created_at
```

---

## Credentials you need

| Credential | Where to get it | n8n credential type |
|-----------|----------------|-------------------|
| Google Sheets OAuth | Same as Flow 1 | Google Sheets OAuth2 |
| Jira API | [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens) | Jira API |
| Slack Bot Token | Same as Flow 1 | Slack API |

### Jira setup

1. Create a free Jira project — suggest key `SEC`, type `Scrum`
2. Generate an API token at the link above
3. In n8n, create a Jira credential with your Atlassian email + API token + your domain (`yourname.atlassian.net`)

---

## Importing the workflow

1. Import `workflow.json` into n8n Cloud
2. Assign credentials: Google Sheets, Jira, Slack
3. Update `YOUR_GOOGLE_SHEET_ID` in the Sheets nodes
4. Update `YOUR_JIRA_PROJECT_KEY` (default: `SEC`) in the Jira nodes
5. Activate

---

## What to extend in production

- Add a Jira webhook back to n8n — when a ticket is closed in Jira, automatically mark the alert resolved in Sheets
- Add assignee lookup from a team mapping table (owner_tag → Jira user ID)
- Add Confluence page creation for P1 incidents (auto-generated incident report)
- Integrate with PagerDuty for on-call escalation on score 10 alerts

---

## Next flow

→ [Flow 4: Weekly Security Posture Digest](../04-weekly-digest/README.md)
