# Flow 8 — Automated Remediation Suggestions (OpenRouter + Mistral)

> **For each confirmed true positive with a Jira ticket, generates: the exact AWS CLI fix command, a Terraform snippet to prevent recurrence, estimated fix time, production risk assessment, and a verification command. Posts the full package to Jira and Slack.**

**Model:** `mistralai/mistral-7b-instruct:free` via OpenRouter

## How it works

1. Reads true positives that have a `jira_ticket_id` but no `remediation_generated` timestamp
2. Calls Mistral with the full alert context + MITRE TTP data from Flow 7
3. Parses a structured remediation package (CLI + Terraform + risk + verification)
4. Writes summary fields back to Sheets
5. Adds the full remediation package as a formatted comment on the Jira ticket
6. Posts a Slack message with the quick-fix CLI command visible inline

## New Google Sheets columns needed

```
remediation_summary | immediate_fix_cli | terraform_snippet | estimated_fix_minutes | production_risk | production_risk_reason | verification_command | remediation_generated | remediation_model
```

## The full 8-flow pipeline

```
Alert arrives (webhook)
  └── Flow 1: Ingest + normalise
        └── Flow 2: AI triage (OpenAI)
              └── Flow 3: Jira ticket
                    └── Flow 5: Anomaly detection (Mistral)
                    └── Flow 6: Multi-model consensus (Llama3 + Qwen)
                    └── Flow 7: Threat intel (Qwen + MITRE + NVD)
                          └── Flow 8: Remediation (Mistral)
                                └── Flow 4: Weekly digest (every Monday)
```
