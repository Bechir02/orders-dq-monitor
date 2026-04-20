# Orders Data Quality Monitor
### n8n · Google Sheets · OpenRouter (LLM) · Gmail · Slack

An automated hybrid data quality monitoring pipeline that validates order records daily, generates AI-powered incident summaries, and delivers alerts through email and Slack.

---

## What it does

Runs on a daily schedule and:

1. Reads all orders from a Google Sheet
2. Checks data freshness — flags if no new orders in 24 hours
3. Runs 5 deterministic validation rules on every row
4. Routes through 3 branches based on severity
5. Calls an LLM via OpenRouter to write a business-friendly summary
6. Sends email and Slack alerts only when failure rate exceeds 10%
7. Logs every run permanently to an audit sheet

---

## Workflow Architecture

```
Schedule Trigger
    → Read Orders (Google Sheets)
        → Check Data Freshness
            → Run Quality Checks (JavaScript)
                → Has Failures?
                    → FALSE: Log Run (clean)
                    → TRUE: Exceeds Threshold? (10%)
                        → FALSE: Log Run (below threshold)
                        → TRUE: Shape LLM Payload
                                → Call LLM (OpenRouter)
                                    → Extract LLM Summary
                                        → Log Run (with failures)
                                            → Send Alert Email (Gmail)
                                                → Send Slack Alert
```

---

## Validation Rules

| Rule | Description |
|---|---|
| Missing fields | Flags blank order_id, customer_id, amount, order_date |
| Duplicate order_id | Detects repeated order IDs across the dataset |
| Invalid amount | Flags zero or negative amounts |
| Invalid date format | Expects YYYY-MM-DD format |
| Future order date | Flags dates beyond today |

---

## Alert Threshold Logic

| Failure Rate | Action |
|---|---|
| 0% | Silent log — clean run recorded |
| 1–10% | Silent log — failures recorded, no alert sent |
| Above 10% | Full alert — LLM summary + email + Slack |

---

## Tech Stack

| Component | Tool |
|---|---|
| Automation engine | n8n (self-hosted) |
| Data source | Google Sheets |
| Validation logic | JavaScript (n8n Code node) |
| LLM summarization | OpenRouter API (google/gemma-3-4b-it:free) |
| Email alerts | Gmail |
| Slack alerts | Slack Incoming Webhook |
| Audit logging | Google Sheets (audit_log tab) |

---

## Google Sheet Structure

### Tab 1: orders
| Column | Type | Description |
|---|---|---|
| order_id | String | Unique order identifier |
| customer_id | String | Customer reference |
| amount | Number | Order value |
| order_date | Date YYYY-MM-DD | Order date |

### Tab 2: audit_log
| Column | Description |
|---|---|
| run_timestamp | When the workflow ran |
| total_rows | Total rows checked |
| passed | Rows that passed all checks |
| failed | Rows with at least one failure |
| failure_summary | Technical detail of each failure |
| llm_summary | AI-generated business summary |
| run_number | Unique run identifier |

---

## Setup Instructions

### Prerequisites
- Self-hosted n8n instance
- Google Cloud project with Sheets API and Gmail API enabled
- OpenRouter account (free tier sufficient)
- Slack workspace with Incoming Webhooks enabled

### Steps

**1. Clone this repo**

```bash
git clone https://github.com/YOUR_USERNAME/orders-dq-monitor
```

**2. Import the workflow into n8n**
- Open your n8n instance
- Click the menu icon
- Select Import
- Upload workflow.json

**3. Configure credentials**

| Credential | Type | Where used |
|---|---|---|
| Google Sheets OAuth2 | OAuth2 | Read Orders, Log Run nodes |
| Gmail | OAuth2 or SMTP | Send Alert Email node |
| OpenRouter API key | Header Auth (Authorization: Bearer) | Call LLM node |
| Slack Webhook URL | Plain URL | Send Slack Alert node |

**4. Set up your Google Sheet**
- Create a new Google Sheet named Orders Data
- Add two tabs: orders and audit_log
- Use the column structure shown above
- Copy the Sheet URL into the Read Orders node Document field

**5. Activate the workflow**
- Click Publish in n8n
- The workflow will run automatically every day at 7:00 AM server time
- Use Execute Workflow to test manually at any time

---

## How the three branches work

### Branch 1 — Clean run
No failures detected. A clean entry is written to the audit log. No alerts sent. Inbox stays quiet.

### Branch 2 — Below threshold
Failures exist but the failure rate is under 10%. The run is logged with failure details but no email or Slack message is sent. Prevents alert fatigue from minor isolated issues.

### Branch 3 — Full alert
Failure rate exceeds 10%. The LLM generates a plain English incident summary. An email is sent to the configured recipient. A Slack message is posted to the configured channel. The full run details are logged to the audit sheet.

---

## Sample Output

### Slack Message
```
Data Quality Alert - 2 failures out of 40 rows. Passed: 38.
Run time: 2026-04-20T10:58:45Z.
Freshness: Data is fresh. Last order: 2026-04-19.
```

### Email Subject
```
Data Quality Alert — 2 failure(s) detected
```

### AI-Generated Business Summary
```
A data quality check was performed on April 20th, examining 40
records. We identified 2 issues, including a future order date
and a missing required field. These errors require immediate
attention to ensure data accuracy and reliability for reporting
and analysis. We recommend reviewing the affected records and
implementing validation at the point of data entry.
```

### Audit Log Entry
| run_timestamp | total_rows | passed | failed | llm_summary | run_number |
|---|---|---|---|---|---|
| 2026-04-20T10:58:45Z | 40 | 38 | 2 | A data quality check... | 1776683157236 |

---

## Data Freshness Check

Every run checks the most recent order_date in the dataset. If no orders have been added in the last 24 hours, a freshness warning is included in the alert. This catches silent pipeline failures where data stops flowing without anyone noticing.

---

## Project Status

- [x] Schedule trigger — daily at 7am
- [x] Google Sheets data source
- [x] 5 deterministic validation rules
- [x] Three-path branching logic
- [x] Data freshness check
- [x] LLM summarization via OpenRouter
- [x] Gmail email alerts
- [x] Slack webhook notifications
- [x] Audit logging with run ID
- [x] Alert threshold at 10% failure rate
- [ ] Auto-correct fixable errors — Level 2
- [ ] Trend analysis comparing runs over time — Level 2
- [ ] Supabase database as data source — Level 3
- [ ] RAG layer for historical pattern comparison — Level 3
- [ ] Live HTML dashboard via webhook — Level 3

---

## Why this project matters

Most data quality tools are either fully manual or require expensive enterprise software. This workflow is a lightweight alternative that any team can run for free on self-hosted n8n.

The hybrid approach — deterministic rules for reliable detection, LLM layer for human-readable reporting — is the same pattern used by modern data observability platforms like Monte Carlo and Great Expectations, built here from scratch for under $0/month.

---

## Real world applications

- E-commerce: validate orders before they reach fulfillment
- Finance: check transaction records before end of day reporting
- HR: validate employee data imports before payroll runs
- Logistics: catch bad shipment dates before dispatch
- Any team receiving data from external sources or partners

---

## LinkedIn Bullet Points

- Built a self-hosted data quality monitoring pipeline using n8n, Google Sheets, and OpenRouter — automatically detects bad records, generates AI-written incident summaries, and delivers alerts via email and Slack with zero manual intervention

- Designed a hybrid deterministic plus LLM workflow that separates rule-based validation from natural language reporting — keeping costs at zero while making quality alerts readable by non-technical stakeholders

- Implemented threshold-based alert routing with three execution branches — clean runs log silently, minor issues are recorded without noise, critical failures trigger full AI-powered incident reports

---

## Author

Built by Becher Zribi
