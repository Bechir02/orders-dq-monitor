# Orders Data Quality Monitor
### n8n · Google Sheets · OpenRouter (LLM) · Gmail · Slack

An automated hybrid data quality monitoring pipeline that validates 
order records daily, generates AI-powered incident summaries, and 
delivers alerts through email and Slack.

---

## What it does

Runs on a daily schedule and:

1. Reads all orders from a Google Sheet
2. Checks data freshness — flags if no new orders in 24 hours
3. Runs 5 deterministic validation rules on every row
4. Routes through 3 branches based on severity
5. Calls an LLM (via OpenRouter) to write a business-friendly summary
6. Sends email and Slack alerts only when failure rate exceeds 10%
7. Logs every run permanently to an audit sheet

---

## Workflow Architecture
