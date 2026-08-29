# Automated Business Lead System

An automated business lead capture, validation, scoring, notification, reliability, security, and lifecycle-management workflow built with **n8n**, **Google Sheets**, **Telegram**, and **Gmail**.

This project demonstrates how incoming business leads can be automatically collected through a webhook, normalized, validated, scored, classified by priority, deduplicated, stored in Google Sheets, distributed through Telegram and Gmail, and followed up automatically until closed.

The workflow also includes invalid lead handling, duplicate detection, error logging, independent retry strategies for external services, and a daily follow-up reminder system.

## Current Version

**v0.7.1 — Lead Management (Retry Counter Bugfix)**

---

## Table of Contents
- [Features](#features)
- [Requirements](#requirements)
- [Architecture & Workflow](#architecture--workflow)
- [Setup & Installation](#setup--installation)
- [Usage Examples](#usage-examples)
- [Core Concepts](#core-concepts)
  - [Lead Scoring](#lead-scoring)
  - [Invalid Lead Handling](#invalid-lead-handling)
  - [Duplicate Detection](#duplicate-detection)
  - [Lead Lifecycle & Follow-up Reminders](#lead-lifecycle--follow-up-reminders)
  - [Retry Strategy](#retry-strategy)
  - [Error Logging](#error-logging)
- [Data Structure](#data-structure)
- [Project Structure](#project-structure)
- [Roadmap](#roadmap)
- [Version History](#version-history)
- [Security Notice](#security-notice)
- [Author](#author)

---

## Features

* Receive business leads through an HTTP `POST` webhook
* **Webhook authentication using Header Auth** (Bearer token)
* Normalize incoming lead data using JavaScript
* Validate required lead information (email format, phone number, project budget, etc.)
* Automatically assign lead status and creation timestamp
* Calculate lead score and classify leads into `HIGH`, `MEDIUM`, or `LOW` priority
* **Detect duplicate leads** by email (case-insensitive) or phone number (digits-only) before storing
* **Generate a unique, collision-safe lead ID** (`LD-YYYYMMDD-XXXX`) for every valid, non-duplicate lead
* Store valid leads in Google Sheets
* Send real-time Telegram notifications and automated Gmail confirmation emails
* Send a lightweight Telegram alert (no new row created) when a duplicate lead is detected
* Handle invalid leads separately and log validation errors
* **Daily automated follow-up reminders** for leads in `CONTACTED`, `QUALIFIED`, or `PROPOSAL` status whose `next_follow_up_at` has passed, with same-day duplicate-reminder protection
* Independent retry mechanism for Google Sheets, Telegram, and Gmail operations (max 3 attempts with wait time)
* Log permanent service failures
* Format budget values as Indonesian Rupiah and display localized timestamps
* **Secure credential management** — secrets stored in n8n credential system, not in workflow JSON

---

## Requirements

* [n8n](https://n8n.io/)
* Google account (Google Sheets, Google Sheets API, Google Drive API, Gmail API)
* Telegram Bot

---

## Architecture & Workflow

### Main Workflow (Lead Intake)

```text
Client
  │
  │ HTTP POST + Authorization: Bearer TOKEN
  ▼
Webhook (Header Auth)
  │
  │ Authentication Check
  │
  ├── INVALID TOKEN → 401 Unauthorized (workflow NOT executed)
  │
  └── VALID TOKEN
       │
       ▼
  Code in JavaScript
       │
       │ Normalize lead data
       ▼
  Validate Lead
       │
       ▼
  Lead Scoring
       │
       ▼
  IF — Valid Lead?
       │
       ├── TRUE
       │    │
       │    ▼
       │  Get All Leads (Duplicate Check)
       │    │
       │    ▼
       │  Detect Duplicate
       │    │
       │    ▼
       │  IF — Is Duplicate?
       │    │
       │    ├── TRUE
       │    │    │
       │    │    ▼
       │    │  Handle Duplicate Lead
       │    │    │
       │    │    ▼
       │    │  Notify Duplicate Lead (Telegram, best-effort, no row created)
       │    │
       │    └── FALSE
       │         │
       │         ▼
       │       Generate Lead ID
       │         │
       │         ├── Google Sheets (Error → Retry up to 3x → Failure Log)
       │         ├── Telegram (Error → Retry up to 3x → Failure Log)
       │         └── Gmail (Error → Retry up to 3x → Failure Log)
       │
       └── FALSE
            │
            ▼
       Handle Invalid Lead
            │
            ▼
       Log Invalid Lead
```

### Follow-up Reminder Workflow (Scheduled)

```text
Follow-up Schedule Trigger (daily cron, 09:00 instance time)
  │
  ▼
Get Leads For Follow-up (reads all rows from Sheet1)
  │
  ▼
Filter Due Leads
  │   status in [CONTACTED, QUALIFIED, PROPOSAL]
  │   AND next_follow_up_at is set and has passed
  │   AND not already reminded today
  ▼
Send Follow-up Reminder (Telegram)
  │
  ▼
Mark Reminder Sent (updates last_reminded_at in Sheet1)
```

### Valid Lead Flow

When a lead passes validation and is not a duplicate, the workflow:
1. Calculates a lead score
2. Assigns a priority
3. Checks for duplicates against existing leads (by email or phone)
4. Generates a unique `lead_id`
5. Stores the lead in Google Sheets
6. Sends a Telegram notification
7. Sends a confirmation email
8. Retries failed external services up to 3 times

### Duplicate Lead Flow

If a lead matches an existing lead's email or phone number, no new row is created. A best-effort Telegram alert is sent referencing the matched `lead_id` or email, and the inquiry is otherwise ignored.

### Invalid Lead Flow

If a lead fails validation, it is not processed as a normal lead. Instead, validation errors are stored for later review in the Invalid Leads Sheet.

---

## Setup & Installation

### 1. Import the Workflow

Import the sanitized workflow template into your n8n instance:
```text
workflows/automated-business-lead-system.template.json
```
> **Note:** The repository contains a sanitized workflow template. You must configure your own credentials, Google Sheets, Telegram bot, Gmail account, and other environment-specific settings.

### 2. Configure Google Sheets

Create a Google Sheet (`Sheet1`) with the required columns for valid leads:
`name`, `email`, `phone`, `business`, `service`, `budget`, `status`, `score`, `priority`, `scoring_reasons`, `created_at`, `lead_id`, `last_contacted_at`, `next_follow_up_at`, `last_reminded_at`

Create a separate sheet for invalid leads (`validation_errors` added) and retry failure logs (`timestamp`, `service`, `name`, `email`, `retry_count`, `error`).

> **Note:** `last_reminded_at` is required for the Follow-up Reminder workflow — without this column header present, `Mark Reminder Sent` cannot match and update the correct row.

### 3. Configure Google OAuth

Create a Google Sheets OAuth credential in n8n. Connect it to your Google account and select your own Google Sheet.

### 4. Configure Telegram

Create a Telegram bot using BotFather. Configure the Telegram credential in n8n and provide your own Telegram chat ID.

### 5. Configure Gmail

Create a Gmail OAuth credential in n8n. Connect your own Gmail account and authorize the required permissions. The confirmation email is sent to `{{ $json.email }}`.

### 6. Configure Webhook Authentication

The webhook is protected using n8n Header Auth.

Create a Header Auth credential in n8n:

- **Name:** `Authorization`
- **Value:** `Bearer YOUR_SECRET_TOKEN`
- **Allowed HTTP Request Domains:** Leave empty unless required.

Attach the credential to the Webhook node:

`Webhook → Authentication → Header Auth`

Use your own secret token. Never commit the token to GitHub or include it in the workflow template.

The production webhook requires the following HTTP header:

Authorization: Bearer YOUR_SECRET_TOKEN

### 7. Configure the Follow-up Schedule Trigger

The `Follow-up Schedule Trigger` node runs on cron expression `0 9 * * *`, evaluated in the **n8n instance's configured timezone** (often UTC by default, not the local timezone of the operator). Adjust either the cron expression or the node's timezone option so the daily check fires at the intended local time.

### 8. Activate the Workflow

After configuring the Webhook authentication and successfully testing the workflow, activate the n8n workflow and use the production webhook URL. Activation also enables the Follow-up Schedule Trigger to run automatically going forward — no manual execution is required in production.

---

## Usage Examples

### Example Request

Send an HTTP `POST` request to the webhook using PowerShell:

```powershell
Invoke-RestMethod `
  -Uri "YOUR_N8N_WEBHOOK_URL" `
  -Method POST `
  -Headers @{
      Authorization = "Bearer YOUR_SECRET"
  } `
  -ContentType "application/json" `
  -Body '{
    "name": "Andi Pratama",
    "email": "andi@example.com",
    "phone": "08123456789",
    "business": "Toko Andi",
    "service": "Website",
    "budget": 3000000
  }'
```

### Example Results

**Google Sheets:**
```text
name                → Andi Pratama
email               → andi@example.com
phone               → 08123456789
business            → Toko Andi
service             → Website
budget              → 3000000
status              → NEW
score               → 70
priority            → HIGH
lead_id             → LD-20260828-K3F9
last_contacted_at   →
next_follow_up_at   →
last_reminded_at    →
created_at          → 2026-08-28T...
```

**Telegram (new lead):**
```text
🚨 NEW BUSINESS LEAD

🆔 LD-20260828-K3F9
👤 Andi Pratama
🏢 Toko Andi

📧 andi@example.com
📱 08123456789

💼 Website
💰 Rp3.000.000

📊 Score: 70/100
🔥 Priority: HIGH

📌 NEW
🕐 28 Agu 2026, 02.00
```

**Telegram (duplicate lead):**
```text
ℹ️ DUPLICATE LEAD DETECTED

👤 Andi Pratama
🏢 Toko Andi
📧 andi@example.com
📱 08123456789

🔗 Matches existing lead: LD-20260828-K3F9

⚠️ No new record was created. Ignoring unless this is a genuinely new inquiry.
```

**Telegram (follow-up reminder):**
```text
⚠️ FOLLOW-UP REMINDER

🆔 LD-20260828-K3F9
👤 Andi Pratama
🏢 Toko Andi
📧 andi@example.com

📊 Score: 70/100
🔥 Priority: HIGH
📌 Status: CONTACTED

⏰ This lead is due for follow-up.
```

**Email:**
The lead automatically receives a confirmation email containing the Lead name, Business name, Requested service, Estimated budget, and Confirmation that the request has been received.

---

## Core Concepts

### Lead Scoring

The workflow automatically calculates a score based on several factors.

**Budget Score:**
| Budget | Score |
| --- | --- |
| `>= Rp10,000,000` | +40 |
| `>= Rp5,000,000` | +30 |
| `>= Rp2,000,000` | +20 |
| `>= Rp1,000,000` | +10 |

**Service Score:** High-value services (Website, Web Development, Web App, Application, Mobile App, Ecommerce) receive additional scoring.
**Business & Contact Info:** A valid business name, email address, and phone number contribute to the score.

**Priority Classification:**
| Score | Priority |
| --- | --- |
| `70+` | HIGH |
| `40–69` | MEDIUM |
| `<40` | LOW |

### Invalid Lead Handling

The validation system checks:
* Required name, email, business name, service
* Email format, phone number length, budget validity

Example validation errors: `name is required`, `invalid email format`, `invalid phone number`, `invalid budget`.

### Duplicate Detection

Before a new lead is stored, the workflow reads all existing rows from `Sheet1` and compares the incoming lead against them:

* **Primary match:** case-insensitive email comparison
* **Secondary match:** digits-only phone number comparison

If either matches an existing row, the lead is flagged as a duplicate: no new row is appended to `Sheet1`, and a best-effort Telegram alert references the matched `lead_id` (or email, if `lead_id` is unavailable). An empty sheet never produces a false-positive match, so the very first lead is never blocked.

### Lead Lifecycle & Follow-up Reminders

Every valid, non-duplicate lead receives a unique `lead_id` in the format `LD-YYYYMMDD-XXXX` (date plus a 4-character random base36 suffix). The suffix is randomly generated rather than sequentially counted, avoiding race conditions from concurrent webhook calls reading "last row + 1" from Google Sheets.

Three additional fields track the lead through its lifecycle:

| Field | Purpose |
| --- | --- |
| `last_contacted_at` | Set manually (or by a future automation) when the lead is first contacted |
| `next_follow_up_at` | Set manually to schedule the next follow-up check |
| `last_reminded_at` | Automatically stamped by the workflow when a reminder is sent |

A daily scheduled trigger (`Follow-up Schedule Trigger`, cron `0 9 * * *`) checks all leads and sends a Telegram reminder for any lead that is:
* In `CONTACTED`, `QUALIFIED`, or `PROPOSAL` status (leads in `NEW`, `WON`, or `LOST` are excluded by definition), **and**
* Has a `next_follow_up_at` timestamp that has already passed, **and**
* Has not already been reminded earlier the same day (tracked via `last_reminded_at`)

After a reminder is sent, `last_reminded_at` is updated on the matching row (matched by `lead_id`) to prevent duplicate reminders within the same day.

### Retry Strategy

External services (Google Sheets, Telegram, Gmail) are protected by independent retry mechanisms (up to **3 times**, with a wait step between attempts). If the service continues to fail, the workflow stops retrying and records the failure in the error log.

> **v0.7.1 bugfix:** Earlier retry counters read `retry_count` from the failing node's own error output, which does not reliably carry the count forward across the retry loop — causing the counter to reset to `1` on every pass and the 3-attempt cap to never engage. Each retry counter node now self-references its own previous execution (e.g. `$('Retry Counter - Telegram')`) so the count survives even when the failing node's error output loses context.

### Error Logging

Permanent service failures are recorded with:
| Field | Description |
| --- | --- |
| `timestamp` | Time of failure |
| `service` | Failed service |
| `name` | Lead name |
| `email` | Lead email |
| `retry_count` | Number of retry attempts |
| `error` | Error message |

---

## Security Testing

The v0.6.0 security layer was tested against the following scenarios:

| Test | Scenario | Expected Result |
| --- | --- | --- |
| TEST 1 | Valid authentication | 200 OK — lead processed |
| TEST 2 | Missing authentication | 401 Unauthorized |
| TEST 3 | Wrong token | 401 Unauthorized |
| TEST 4 | Malformed Authorization header | 401 Unauthorized |
| TEST 5 | Valid token + invalid lead | Validation and invalid lead handling |
| TEST 6 | Valid token + service failure | Retry mechanism remains functional |

Security testing confirmed that webhook authentication does not interfere with the existing v0.5.0 reliability features, and that the v0.7.x duplicate detection and follow-up reminder logic operate correctly on top of the existing security layer.

## Data Structure

### Lead Data

| Field | Description |
| --- | --- |
| `name` | Lead's name |
| `email` | Lead's email address |
| `phone` | Lead's phone number |
| `business` | Business or company name |
| `service` | Requested service |
| `budget` | Estimated project budget |
| `status` | Current lead status (`NEW`, `CONTACTED`, `QUALIFIED`, `PROPOSAL`, `WON`, `LOST`, `DUPLICATE`, `INVALID`) |
| `score` | Automatically calculated lead score |
| `priority` | `HIGH`, `MEDIUM`, or `LOW` |
| `scoring_reasons` | Reasons contributing to the score |
| `lead_id` | Unique identifier, format `LD-YYYYMMDD-XXXX` |
| `last_contacted_at` | Timestamp of last contact with the lead |
| `next_follow_up_at` | Timestamp when the next follow-up reminder is due |
| `last_reminded_at` | Timestamp of the most recent follow-up reminder sent |
| `created_at` | Automatically generated timestamp |

---

## Project Structure

```text
Automated-Business-Lead-System/
│
├── workflows/
│   └── automated-business-lead-system.template.json
│
├── .gitignore
└── README.md
```

---

## Roadmap

### v0.1.0 — MVP (Completed)
* Webhook lead intake, Lead data normalization, Email validation, Google Sheets integration, Automatic lead status and timestamp.

### v0.2.0 — Telegram Notifications (Completed)
* Telegram lead notifications, New lead alerts, Lead summary messages, Formatted Telegram messages, Indonesian Rupiah budget formatting, Localized timestamp formatting.

### v0.3.0 — Email Automation (Completed)
* Automatic email notification, HTML email template, Dynamic lead info, Personalized confirmation.

### v0.4.0 — Lead Scoring (Completed)
* Budget/Service/Business/Contact scoring, Priority classification, Score stored in Sheets & Telegram.

### v0.5.0 — Reliability (Completed)
* Error handling, Invalid lead handling, Max 3 retry attempts (Google Sheets, Telegram, Gmail), Independent retry handling, Retry failure logging.

### v0.6.0 — Security (Completed)
* Webhook authentication using Header Auth
* Bearer token protection
* Unauthorized requests rejected with HTTP 401
* Valid authentication tested successfully
* Invalid authentication scenarios tested
* Credential secrets stored in n8n credential system
* Sanitized workflow template for repository distribution
* Security testing for valid, missing, wrong, and malformed authentication
* Compatibility testing with v0.5.0 reliability features

### v0.7.0 — Lead Management (Completed)
* Duplicate lead detection (email + phone matching against existing Sheet1 rows)
* Best-effort Telegram alert on duplicate detection (no duplicate row created)
* Unique, collision-safe `lead_id` generation (`LD-YYYYMMDD-XXXX`)
* Lead lifecycle fields: `last_contacted_at`, `next_follow_up_at`, `last_reminded_at`
* Daily scheduled follow-up reminder workflow (Telegram)
* Same-day duplicate-reminder protection via `last_reminded_at`

### v0.7.1 — Retry Counter Bugfix (Completed)
* Fixed retry counters (Google Sheets, Telegram, Gmail) resetting to 1 on every retry pass instead of incrementing, which caused the 3-attempt cap to never engage
* Retry counter nodes now self-reference their own previous execution instead of relying on the failing node's error output

### Upcoming Versions
* **v0.8.0 — Analytics:** Conversion metrics, Priority/Budget distribution.
* **v0.9.0 — Production Preparation:** End-to-end testing, Documentation, Portfolio screenshots.
* **v1.0.0 — Production Release:** Production-ready workflow and security configuration.

---

## Version History
* **v0.7.1 — 2026-08-30:** Fixed retry counter logic (Google Sheets, Telegram, Gmail) so the 3-attempt cap engages correctly across retry passes.
* **v0.7.0 — 2026-08-29:** Added duplicate lead detection, unique lead ID generation, lead lifecycle fields, and a daily automated follow-up reminder workflow.
* **v0.6.0 — 2026-08-29:** Added webhook authentication, Bearer token protection, credential security, and security testing.
* **v0.5.0 — 2026-08-28:** Added reliability and error recovery mechanisms (retries, invalid lead handling, failure logs).
* **v0.4.0 — 2026-08-26:** Added lead scoring and priority classification.
* **v0.3.0 — 2026-08-25:** Added automated Gmail confirmation emails.
* **v0.2.0 — 2026-08-25:** Added real-time Telegram notifications.
* **v0.1.0 — 2026-08-25:** Initial MVP release.

---

## Security Notice

This repository does not contain:

- Google OAuth client secrets
- Access tokens
- API keys
- Passwords
- Telegram bot tokens
- Private Google Sheet IDs
- Private Telegram chat IDs
- Webhook authentication secrets

The workflow template uses placeholder values where necessary.

All credentials and secrets must be configured directly in n8n's credential system and must never be committed to the repository.

If a secret is accidentally exposed, revoke or rotate it immediately.

---

## Author

**FruHafizd**

Built as a portfolio project demonstrating business workflow automation with n8n.