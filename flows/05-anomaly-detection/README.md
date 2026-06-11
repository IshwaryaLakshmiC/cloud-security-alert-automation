# Flow 5 — Anomaly Detection (OpenRouter + Mistral)

> **Builds a 30-day rolling baseline per resource type + source. Every 15 minutes, scores new alerts against that baseline — flagging frequency spikes, severity jumps, and never-seen-before patterns. Confirmed anomalies go to Slack.**

**Model:** `mistralai/mistral-7b-instruct:free` via OpenRouter

## How it works

1. Reads full alert log from Google Sheets
2. Builds baseline stats (avg severity, hourly rate) per `resource_type + source` group over 30 days
3. Scores new alerts (last 15 min) on: severity spike, frequency spike on same resource, unknown baseline
4. Sends anomaly candidates to Mistral for behavioural confirmation
5. Writes `anomaly_score`, `llm_anomaly_verdict`, `recommended_action` back to Sheets
6. Slacks confirmed anomalies with full context

## New Google Sheets columns needed

```
anomaly_score | anomaly_reasons | is_anomaly | llm_anomaly_verdict | llm_anomaly_reasoning | recommended_action | anomaly_checked | anomaly_model
```

## n8n credential needed

- **OpenRouter API** — HTTP Header Auth credential, header name `Authorization`, value `Bearer YOUR_KEY`
