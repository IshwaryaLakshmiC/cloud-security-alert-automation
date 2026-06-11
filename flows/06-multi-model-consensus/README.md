# Flow 6 — Multi-Model Consensus Triage (OpenRouter)

> **Sends each high-priority alert to Llama 3 and Qwen simultaneously. If they agree — consensus is recorded. If they disagree by 2+ points or give different decisions — alert is flagged as "contested" and routed for human review.**

**Models:** `meta-llama/llama-3-8b-instruct:free` + `qwen/qwen2-7b-instruct:free` via OpenRouter

## How it works

1. Reads scored alerts with `risk_score >= 7` and no `consensus_verdict` yet
2. Builds a shared triage prompt from alert fields
3. Sends to both models in parallel via n8n's fan-out
4. Parses both responses, computes score divergence and decision match
5. Sets `consensus_status`: `agreed` | `partial` | `disagreed`
6. Writes all agent outputs + consensus verdict to Sheets
7. Slacks contested alerts with both models' reasoning side-by-side

## New Google Sheets columns needed

```
agent1_score | agent1_decision | agent1_model | agent2_score | agent2_decision | agent2_model | consensus_verdict | consensus_status | consensus_avg_score | score_divergence | consensus_checked_at
```

## Why this matters in an SE interview

Single-model triage has a confidence problem — you don't know when the model is uncertain. Multi-model consensus surfaces that uncertainty explicitly and routes it to a human. This demonstrates production-grade thinking about AI reliability.
