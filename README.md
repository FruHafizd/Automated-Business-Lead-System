# Automated Business Lead System

An automated lead capture and management workflow built with **n8n** and **Google Sheets**.

This project demonstrates how incoming business leads can be collected through a webhook, validated, transformed into a standardized format, and automatically stored in Google Sheets.

## Current Version

**v0.1.0 — MVP**

## Workflow

```text
Client
  │
  │ HTTP POST
  ▼
Webhook
  │
  ▼
Code
  │
  │ Normalize lead data
  ▼
IF
  │
  │ Email exists?
  ├───────────────┐
  │ YES           │ NO
  ▼               ▼
Google Sheets    Stop
  │
  ▼
Telegram
  │
  ▼
Lead Notification
```

## Features

- Receive business leads through an HTTP `POST` webhook
- Normalize incoming lead data using JavaScript
- Validate whether an email address is provided
- Automatically assign a `NEW` lead status
- Automatically generate a creation timestamp
- Store valid leads in Google Sheets
- Send real-time Telegram notifications for new leads
- Format lead information into a readable notification
- Display budget in Indonesian Rupiah format
- Display localized creation timestamps

## Lead Data

| Field | Description |
|---|---|
| `name` | Lead's name |
| `email` | Lead's email address |
| `phone` | Lead's phone number |
| `business` | Business or company name |
| `service` | Requested service |
| `budget` | Estimated project budget |
| `status` | Current lead status |
| `created_at` | Automatically generated timestamp |

## Example Request

Send an HTTP `POST` request to the webhook:

```json
{
  "name": "Andi Pratama",
  "email": "andi@example.com",
  "phone": "08123456789",
  "business": "Toko Andi",
  "service": "Website",
  "budget": 3000000
}
```

## Example Result

After validation and processing, the lead is stored as:

```text
name       → Andi Pratama
email      → andi@example.com
phone      → 08123456789
business   → Toko Andi
service    → Website
budget     → 3000000
status     → NEW
created_at → 2026-08-25T...
```

## Google Sheets Structure

The Google Sheet should contain the following columns:

| name | email | phone | business | service | budget | status | created_at |
|---|---|---|---|---|---|---|---|

## Requirements

- [n8n](https://n8n.io/)
- Google account
- Google Sheets
- Google Sheets API
- Google Drive API

## Setup

### 1. Import the Workflow

Import:

```text
workflows/automated-business-lead-system.json
```

into your n8n instance.

### 2. Configure Google Sheets

Create a Google Sheet with these columns:

```text
name
email
phone
business
service
budget
status
created_at
```

### 3. Configure Google OAuth

Create a Google Sheets OAuth credential in n8n and connect it to your Google account.

### 4. Select Your Spreadsheet

Open the **Google Sheets** node and select your own spreadsheet and worksheet.

### 5. Configure the Webhook

The workflow expects an HTTP `POST` request containing lead information.

### 6. Test the Workflow

Example using PowerShell:

```powershell
Invoke-RestMethod `
  -Uri "YOUR_N8N_WEBHOOK_URL" `
  -Method POST `
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

### 7. Activate the Workflow

After successful testing, activate the n8n workflow and use the production webhook URL.

## Project Structure

```text
Automated-Business-Lead-System/
│
├── workflows/
│   └── automated-business-lead-system.json
│
├── .gitignore
└── README.md
```

## Roadmap

### v0.1.0 — MVP

- [x] Webhook lead intake
- [x] Lead data normalization
- [x] Email validation
- [x] Google Sheets integration
- [x] Automatic lead status
- [x] Automatic timestamp

### v0.2.0 — Notifications

- [x] Telegram lead notifications
- [x] New lead alerts
- [x] Lead summary messages

### v0.3.0 — Email Automation

- [ ] Automatic email notification
- [ ] Lead confirmation email
- [ ] Follow-up email workflow

### v0.4.0 — Lead Scoring

- [ ] Budget-based scoring
- [ ] Service-based scoring
- [ ] Lead priority classification

### v0.5.0 — Reliability

- [ ] Error handling
- [ ] Failed execution logging
- [ ] Invalid lead handling
- [ ] Retry strategy

### v0.6.0 — Security

- [ ] Environment-based configuration
- [ ] Credential security improvements
- [ ] Webhook protection

### v0.9.0 — Production Preparation

- [ ] End-to-end testing
- [ ] Documentation improvements
- [ ] Workflow optimization
- [ ] Portfolio screenshots

### v1.0.0 — Production Release

- [ ] Production-ready workflow
- [ ] Complete documentation
- [ ] Stable notification system
- [ ] Complete lead management pipeline

## Version History

### v0.1.0 — 2026-08-25

Initial MVP release.

Implemented the core lead capture pipeline:

- HTTP webhook
- JavaScript data transformation
- Email validation
- Google Sheets storage
- Automatic `NEW` status
- Automatic timestamp generation

### v0.2.0 — 2026-08-25

Added real-time Telegram notifications for new business leads.

Implemented:

- Telegram bot integration
- Automatic new lead notifications
- Formatted lead notification messages
- Indonesian Rupiah budget formatting
- Localized timestamp formatting


## Security Notice

This repository does not contain Google OAuth client secrets, access tokens, API keys, passwords, or other private credentials.

When importing the workflow, configure your own credentials and Google Sheets document.

---

## Author

**FruHafizd**

Built as a portfolio project demonstrating workflow automation with n8n.