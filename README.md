# 🔍 Orders Data Quality Monitor
### n8n · Google Sheets · OpenRouter (LLM) · Gmail · Slack

> An automated hybrid data quality monitoring pipeline that validates order records daily, generates AI-powered incident summaries, and delivers smart alerts all for $0/month.

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![n8n](https://img.shields.io/badge/Built%20with-n8n-orange)
![LLM](https://img.shields.io/badge/LLM-OpenRouter-blue)
![Cost](https://img.shields.io/badge/Cost-Free%20Tier-success)

---

## 🧠 The Problem This Solves

Every business that receives order data faces the same silent problem:

- Someone submits an order with a blank customer ID
- A system imports a duplicate order
- A date field gets corrupted to an invalid format
- Nobody notices until it causes a real business issue

Manual checking is slow, inconsistent, and gets skipped on busy days. Expensive enterprise tools are overkill for most teams.

**This workflow automates the entire process from detects issues, explains them in plain English to alerts the right people automatically.**

---

## 📁 Repository Structure

```
orders-dq-monitor/
├── 📄 workflow.json          ← n8n workflow export
├── 📖 README.md              
├── 📊 sample-data/
│   └── orders_sample.xlsx    ← test dataset with intentional bad rows
└── 📸 screenshots/
    ├── canvas.png            ← full workflow canvas
    ├── slack-alert.png       ← sample Slack notification
    └── email-alert.png       ← sample email alert
```

---
## ✨ What it does

Runs on a daily schedule and:

1. 📥 Reads all orders from a Google Sheet
2. 🕐 Checks data freshness and flags if no new orders in 24 hours
3. ✅ Runs 5 deterministic validation rules on every row
4. 🔀 Routes through 3 branches based on severity
5. 🤖 Calls an LLM via OpenRouter to write a business-friendly summary
6. 📧 Sends email and Slack alerts only when failure rate exceeds 10%
7. 📊 Logs every run permanently to an audit sheet

---

## 🏗️ Workflow Architecture

```
⏰ Schedule Trigger
    → 📋 Read Orders (Google Sheets)
        → 🕐 Check Data Freshness
            → 🔍 Run Quality Checks (JavaScript)
                → ❓ Has Failures?
                    → ✅ FALSE: Log Run (clean)
                    → ⚠️  TRUE: Exceeds Threshold? (10%)
                        → 📝 FALSE: Log Run (below threshold)
                        → 🚨 TRUE: Shape LLM Payload
                                  → 🤖 Call LLM (OpenRouter)
                                      → 📝 Extract LLM Summary
                                          → 📊 Log Run (with failures)
                                              → 📧 Send Alert Email
                                                  → 💬 Send Slack Alert
```

---

## 🛡️ Validation Rules

| # | Rule | Description | Example Failure |
|---|---|---|---|
| 1 | 🔴 Missing fields | Flags blank order_id, customer_id, amount, order_date | order_id is empty |
| 2 | 🟠 Duplicate order_id | Detects repeated order IDs across the full dataset | ORD-1002 appears twice |
| 3 | 🟡 Invalid amount | Flags zero or negative amounts | amount = -5 |
| 4 | 🔵 Invalid date format | Expects strict YYYY-MM-DD format | order_date = not-a-date |
| 5 | 🟣 Future order date | Flags dates beyond today | order_date = 2099-01-01 |

---

## 🔀 Three-Branch Alert Logic

```
                    ┌─────────────────┐
                    │   Run Complete   │
                    └────────┬────────┘
                             │
              ┌──────────────▼──────────────┐
              │      Any failures found?     │
              └──────┬───────────────┬───────┘
                     │ NO            │ YES
                     ▼               ▼
              ✅ Log clean    ┌──────────────────┐
                 run silently │ Failure rate >10%?│
                              └──────┬───────┬───┘
                                     │ NO    │ YES
                                     ▼       ▼
                              📝 Log only  🚨 Full alert
                              no noise     LLM + Email
                                           + Slack
```

| Failure Rate | Action | Why |
|---|---|---|
| 0% | ✅ Silent log | Nothing to report |
| 1–10% | 📝 Log only, no alert | Prevents alert fatigue |
| Above 10% | 🚨 Full alert pipeline | Needs human attention |

---

## 🛠️ Tech Stack

| Component | Tool | Cost |
|---|---|---|
| 🔧 Automation engine | n8n (self-hosted) | Free |
| 📋 Data source | Google Sheets | Free |
| 💻 Validation logic | JavaScript (n8n Code node) | Free |
| 🤖 LLM summarization | OpenRouter (google/gemma-3-4b-it:free) | Free |
| 📧 Email alerts | Gmail | Free |
| 💬 Slack alerts | Slack Incoming Webhook | Free |
| 📊 Audit logging | Google Sheets (audit_log tab) | Free |

**Total monthly cost: $0**

---

## 📋 Google Sheet Structure

### Tab 1: `orders` — Source data

| Column | Type | Description | Example |
|---|---|---|---|
| order_id | String | Unique order identifier | ORD-1001 |
| customer_id | String | Customer reference | CUST-201 |
| amount | Number | Order value | 150.00 |
| order_date | Date (YYYY-MM-DD) | Order date | 2026-04-15 |

### Tab 2: `audit_log` — Run history

| Column | Description |
|---|---|
| run_timestamp | ISO timestamp of when the workflow ran |
| total_rows | Total rows checked in that run |
| passed | Rows that passed all 5 checks |
| failed | Rows with at least one failure |
| failure_summary | Technical detail of each failure |
| llm_summary | AI-generated business-friendly summary |
| run_number | Unique run identifier (Unix timestamp) |

---

## 🚀 Setup Instructions

### Prerequisites

- ✅ Self-hosted n8n instance running
- ✅ Google Cloud project with Sheets API and Gmail API enabled
- ✅ OpenRouter account (free tier)
- ✅ Slack workspace with Incoming Webhooks enabled

### Step by Step

**1. Clone this repo**

```bash
git clone https://github.com/Bechir02/orders-dq-monitor
cd orders-dq-monitor
```

**2. Import the workflow into n8n**

```
n8n → click ••• menu → Import → upload workflow.json
```

**3. Configure credentials**

| Credential | Type | Node |
|---|---|---|
| Google Sheets OAuth2 | OAuth2 | Read Orders, Log Run nodes |
| Gmail | OAuth2 or SMTP | Send Alert Email |
| OpenRouter API key | Header Auth (Authorization: Bearer YOUR_KEY) | Call LLM |
| Slack Webhook URL | Plain URL in HTTP Request body | Send Slack Alert |

**4. Set up Google Sheet**

- Create a sheet named `Orders Data`
- Add two tabs: `orders` and `audit_log`
- Use the column headers exactly as shown above
- Paste your Sheet URL into the Read Orders node

**5. Activate**

```
n8n → click Publish → workflow runs daily at 7:00 AM server time
```

---

## 📊 Sample Output

### 💬 Slack Alert
```
Data Quality Alert - 2 failures out of 40 rows. Passed: 38.
Run time: 2026-04-20T10:58:45Z.
Freshness: Data is fresh. Last order: 2026-04-19.
```

### 📧 Email Subject
```
⚠️ Data Quality Alert — 2 failure(s) detected
```

### 🤖 AI-Generated Business Summary
```
A data quality check was performed on April 20th, examining 40 records.
We identified 2 issues, including a future order date and a missing
required field. These errors require immediate attention to ensure data
accuracy and reliability for reporting and analysis. We recommend
reviewing the affected records and implementing validation at the point
of data entry.
```

### 📊 Audit Log Entry
| run_timestamp | total_rows | passed | failed | llm_summary | run_number |
|---|---|---|---|---|---|
| 2026-04-20T10:58:45Z | 40 | 38 | 2 | A data quality check... | 1776683157236 |

---

## 🕐 Data Freshness Check

Every run checks the most recent `order_date` in the dataset.

- ✅ **Fresh** — orders received within the last 24 hours
- ⚠️ **Stale** — no new orders in over 24 hours → warning included in alert

This catches silent pipeline failures where data stops flowing without anyone noticing — one of the hardest problems to detect manually.

---

## 📈 Project Roadmap

### ✅ Completed — v1.0
- [x] ⏰ Schedule trigger — daily at 7am
- [x] 📋 Google Sheets data source
- [x] 🔍 5 deterministic validation rules
- [x] 🔀 Three-path branching logic
- [x] 🕐 Data freshness check
- [x] 🤖 LLM summarization via OpenRouter
- [x] 📧 Gmail email alerts
- [x] 💬 Slack webhook notifications
- [x] 📊 Audit logging with unique run ID
- [x] ⚡ Alert threshold at 10% failure rate

### 🔄 In Progress — v1.1
- [ ] 🔧 Auto-correct fixable errors (trim whitespace, fix date formats)
- [ ] 📈 Trend analysis comparing failure rates across runs
- [ ] 🧹 Write cleaned rows to a separate orders_cleaned sheet

### 🗺️ Planned — v2.0
- [ ] 🗄️ Replace Google Sheets with Supabase (PostgreSQL)
- [ ] 🧠 RAG layer for historical pattern comparison
- [ ] 📊 Live HTML dashboard via n8n webhook trigger
- [ ] 🔔 PagerDuty or OpsGenie integration for critical alerts


---

## 👤 Author

**Becher Zribi**
Automation Engineer · Data Quality · n8n · AI Workflows

## ⭐ If this project helped you

Give it a star on GitHub — it helps others find it and motivates continued development.

---

