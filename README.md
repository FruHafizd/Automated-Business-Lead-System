# Automated Business Lead System

An automated lead capture and management workflow built with **n8n** and **Google Sheets**.

This project demonstrates how incoming business leads can be collected through a webhook, validated, transformed into a standardized format, stored in Google Sheets, and automatically processed through Telegram and Gmail notifications.

## Current Version

**v0.4.0 — Lead Scoring**

## Workflow

```text
Client
  │
  │ HTTP POST
  ▼
Webhook
  │
  ▼
Code in JavaScript
  │
  │ Normalize lead data
  ▼
IF
  │
  │ Email exists?
  │
  ├───────────────┐
  │ YES           │ NO
  ▼               ▼
 ┌──────────────────────────────┐
 │                              │
 ▼              ▼               ▼
Google Sheets   Telegram        Gmail
 │              │               │
 ▼              ▼               ▼
Store Lead      Notification    Confirmation
```

When a valid email address is provided, the workflow sends the lead data to three independent destinations:

- **Google Sheets** — stores the lead
- **Telegram** — sends a real-time notification
- **Gmail** — sends a confirmation email to the lead

If the email field is empty, the `IF` node stops the lead from being processed.

## Features

- Receive business leads through an HTTP `POST` webhook
- Normalize incoming lead data using JavaScript
- Validate whether an email address is provided
- Automatically assign a `NEW` lead status
- Automatically generate a creation timestamp
- Store valid leads in Google Sheets
- Send real-time Telegram notifications for new leads
- Format lead information into a readable Telegram notification
- Send automated confirmation emails to leads
- Format email content using HTML
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

After processing, the lead is stored and notifications are sent automatically.

### Google Sheets

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

### Telegram

```text
🚨 NEW BUSINESS LEAD

👤 Andi Pratama
🏢 Toko Andi

📧 andi@example.com
📱 08123456789

💼 Website
💰 Rp3.000.000

📌 NEW
🕐 25 Agu 2026, 02.00
```

### Email

The lead automatically receives a confirmation email containing:

- Lead name
- Business name
- Requested service
- Estimated budget
- Confirmation that the request has been received

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
- Gmail API
- Telegram Bot

## Setup

### 1. Import the Workflow

Import the sanitized workflow template:

```text
workflows/automated-business-lead-system.template.json
```

into your n8n instance.

> **Note:** The repository contains a sanitized workflow template. You must configure your own credentials, Google Sheet, Telegram bot, and Gmail account.

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

Select your own Google Sheet in the Google Sheets node.

### 4. Configure Telegram

Create a Telegram bot using BotFather.

Configure the Telegram credential in n8n and provide your own Telegram chat ID.

The template does not contain the original chat ID.

### 5. Configure Gmail

Create a Gmail OAuth credential in n8n.

Connect your own Gmail account and authorize the required Gmail permissions.

The Gmail node sends the confirmation email to:

```text
{{ $json.email }}
```

### 6. Configure the Webhook

The workflow expects an HTTP `POST` request containing lead information.

### 7. Test the Workflow

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

### 8. Verify the Result

A successful request should:

1. Store the lead in Google Sheets
2. Send a Telegram notification
3. Send a confirmation email to the lead

### 9. Activate the Workflow

After successful testing, activate the n8n workflow and use the production webhook URL.

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

## Roadmap

### v0.1.0 — MVP

- [x] Webhook lead intake
- [x] Lead data normalization
- [x] Email validation
- [x] Google Sheets integration
- [x] Automatic lead status
- [x] Automatic timestamp

### v0.2.0 — Telegram Notifications

- [x] Telegram lead notifications
- [x] New lead alerts
- [x] Lead summary messages
- [x] Formatted Telegram messages
- [x] Indonesian Rupiah budget formatting
- [x] Localized timestamp formatting

### v0.3.0 — Email Automation

- [x] Automatic email notification
- [x] Lead confirmation email
- [x] HTML email template
- [x] Dynamic lead information
- [x] Indonesian Rupiah budget formatting
- [x] Personalized confirmation message

### v0.4.0 — Lead Scoring

- [x] Budget-based scoring
- [x] Service-based scoring
- [x] Lead priority classification
- [x] Lead score stored in Google Sheets
- [x] Lead score displayed in Telegram

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

Implemented:

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

### v0.3.0 — 2026-08-25

Added automated Gmail confirmation emails for new leads.

Implemented:

- Gmail OAuth integration
- Automatic lead confirmation emails
- HTML email template
- Dynamic lead information
- Indonesian Rupiah budget formatting
- Personalized confirmation message

### v0.4.0 — 2026-08-26

Added lead scoring and priority classification.

Implemented:

- Budget-based lead scoring
- Service-based lead scoring
- Business information scoring
- Contact information scoring
- Automatic lead score calculation
- HIGH / MEDIUM / LOW priority classification
- Lead score stored in Google Sheets
- Lead score displayed in Telegram notifications

## Security Notice

This repository does not contain:

- Google OAuth client secrets
- Access tokens
- API keys
- Passwords
- Telegram bot tokens
- Private Google Sheet IDs
- Private Telegram chat IDs

The workflow template uses placeholder values where necessary.

When importing the workflow, configure your own credentials and services.

## Author

**FruHafizd**

Built as a portfolio project demonstrating workflow automation with n8n.