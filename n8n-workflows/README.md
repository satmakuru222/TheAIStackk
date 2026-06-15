# n8n Workflow Collection — The AI Stack

15 production-ready n8n workflow JSON files. Import any workflow via **n8n → Workflows → Import from File**.

---

## Workflow Index

| # | File | Trigger | Services | Use Case |
|---|------|---------|----------|----------|
| 01 | `01-new-lead-crm-email-slack.json` | Webhook (form) | HubSpot · Gmail · Slack | Lead routing |
| 02 | `02-daily-sales-report-email.json` | Schedule 8AM | Google Sheets · Gmail | Sales reporting |
| 03 | `03-ai-support-ticket-triage.json` | Webhook | OpenAI · Slack Switch | AI ticket routing |
| 04 | `04-invoice-email-sheets-accounting.json` | Gmail trigger | Google Sheets · Slack | Invoice capture |
| 05 | `05-github-issue-notion-slack.json` | GitHub webhook | Notion · Slack | Dev issue sync |
| 06 | `06-rss-social-media-auto-post.json` | Schedule 2×/day | RSS · OpenAI · Twitter · LinkedIn | Content automation |
| 07 | `07-shopify-order-fulfillment-email.json` | Shopify order | Shopify · Gmail | Order processing |
| 08 | `08-youtube-upload-notify-platforms.json` | YouTube webhook | YouTube API · Twitter · Instagram · Slack | Video distribution |
| 09 | `09-employee-offboarding-checklist.json` | Webhook (HR form) | Gmail · GSuite Admin · Slack | HR offboarding |
| 10 | `10-weekly-ai-newsletter-autodraft.json` | Schedule Monday | RSS · OpenAI · Beehiiv · Slack | Newsletter automation |
| 11 | `11-form-pdf-generator-email.json` | Webhook | PDFMonkey · Gmail | Document generation |
| 12 | `12-airtable-gsheets-bidirectional-sync.json` | Schedule 30min | Airtable · Google Sheets | Database sync |
| 13 | `13-broken-link-monitor-alert.json` | Schedule hourly | HTTP · Google Sheets · Slack | Site monitoring |
| 14 | `14-ai-meeting-notes-action-items-notion.json` | Webhook | Whisper · GPT-4o · Notion | Meeting intelligence |
| 15 | `15-multichannel-customer-support-router.json` | Webhook | Zendesk · HubSpot · Slack · Gmail | Support routing |

---

## Architecture Diagrams

### 01 — New Lead → CRM + Email + Slack
```
Form POST → [Validate] → ┬→ HubSpot (Create Contact)
                         ├→ Gmail (Welcome Email)
                         └→ Slack (Rich Block Alert)
                              ↓
                         [Respond 200 OK]
```

### 02 — Daily Sales Report
```
CRON 8AM Mon-Fri → [Google Sheets Read] → [Code: Calc Totals + Build HTML]
                                                    ↓
                                          Gmail → Leadership Distribution
```

### 03 — AI Support Ticket Triage
```
Webhook → [Verify HMAC Sig] → [OpenAI GPT-4o Classify] → [Parse Result]
                                                                ↓
                                                    Switch (department)
                                                   ↙  ↓  ↓  ↓  ↘
                                               Billing Tech Account Product General
                                               (Slack channels per department)
```

### 04 — Invoice Email → Accounting
```
Gmail Monitor (unread invoices) → [Code: Regex Extract Amount]
                                          ↓
                               Google Sheets Append Row
                                          ↓
                                  IF amount ≥ $1000?
                                 YES ↓         NO → (done)
                              Slack Finance Alert
```

### 05 — GitHub Issue → Notion + Slack
```
GitHub Webhook → IF action=opened? → [Code: Parse Labels/Priority]
                                              ↓              ↓
                                    Notion Create Page   Slack Dev Alert
                                    (with full body)     (with links to both)
```

### 06 — RSS → Social Media (AI Agent)
```
CRON 9AM/2PM → [RSS Feed] → [Filter Recent <6hrs]
                                      ↓
                              [GPT-4o Write Posts]
                                      ↓
                              [Parse & Format]
                             ↙              ↘
                    Twitter/X Post      LinkedIn Post
```

### 07 — Shopify Order → Fulfillment + Email
```
Shopify Order Created → [Parse Order]
                               ↓
                    IF paid AND needs shipping?
                   YES ↓               ↓ YES (both branches)
              Shopify Fulfillment   Gmail Thank You Email
                   NO  ↓               ↓ (email only)
                   (skip)          Gmail Thank You Email
```

### 08 — YouTube Upload → All Platforms
```
YouTube PubSubHubbub → [Parse XML] → [YouTube API: Get Details]
                                              ↓
                                    [Format Platform Messages]
                                   ↙        ↓          ↘
                          Twitter Post   Slack Notify  Instagram (Create→Publish)
```

### 09 — Employee Offboarding
```
HR Form POST → [Parse] → ┬→ Gmail (Employee Checklist)
                          ├→ Gmail (Manager Action Items)
                          ├→ GSuite Admin (Flag Account)
                          └→ Slack HR Ops (Summary)
```

### 10 — Weekly AI Newsletter
```
CRON Monday 7AM → ┬→ RSS TechCrunch
                   ├→ RSS AI News       → [Merge & Dedupe]
                   └→ RSS VentureBeat         ↓
                                      [GPT-4o Draft Newsletter]
                                              ↓
                                  ┬→ Beehiiv Create Draft
                                  └→ Slack Notify Team
```

### 11 — Form → PDF → Email
```
Webhook → [Code: Build Invoice HTML] → [PDFMonkey Create]
                                              ↓
                                       [Wait 5s]
                                              ↓
                                    [PDFMonkey Get Status]
                                              ↓
                                    [Download PDF Binary]
                                              ↓
                                    Gmail (PDF Attachment)
```

### 12 — Airtable ↔ Google Sheets Sync
```
CRON 30min → ┬→ Airtable Read All   → [Code: Diff by Record ID]
              └→ Google Sheets Read         ↓
                                   IF any changes?
                              YES ↙              ↘ NO
                   GSheets Upsert    Airtable Upsert    → Log Run
```

### 13 — Broken Link Monitor
```
CRON Hourly → [URL List] → [HTTP GET each URL] → [Evaluate Status]
                                                         ↓
                                              IF status ≥ 400 OR timeout?
                                             YES ↙            ↘ NO
                                      Slack Alert         (log only)
                                             ↓                 ↓
                                      Google Sheets Log   Google Sheets Log
```

### 14 — AI Meeting Notes → Notion
```
Webhook (audio/text) → [Parse Input Type]
                               ↓
                    IF audio binary input?
                  YES ↓             ↓ NO (text transcript)
             Whisper API         [Merge Transcript]
                  ↓                     ↑
              [Merge Transcript] ←──────┘
                      ↓
              [GPT-4o: Extract Action Items/Decisions]
                      ↓
              [Parse & Format for Notion]
                      ↓
              Notion: Create Structured Meeting Page
```

### 15 — Multi-channel Support Router
```
Unified Webhook ← (Intercom / Zendesk / Slack / Web Form)
        ↓
[Normalize Payload: detect source, extract email/message/tier]
        ↓
Switch: Paid vs Free tier
   PAID ↙                    ↘ FREE
Zendesk Priority        Zendesk Standard
HubSpot Update              ↓
Slack VIP Alert       Gmail Confirmation
       ↓
Gmail Confirmation
       ↓
Respond 200 with ticket ID
```

---

## Global Prerequisites

### n8n Setup
- n8n version 1.0+ (self-hosted or n8n Cloud)
- Webhook URL format: `https://your-n8n.domain.com/webhook/PATH`

### Credentials to Configure (per workflow)
Each workflow lists its specific prerequisites in the `meta.prerequisites` field inside the JSON.

**Common credentials needed across multiple workflows:**
- **Gmail OAuth2** — 01, 02, 07, 09, 11, 15
- **Slack Bot Token** — 01, 03, 04, 05, 06, 08, 09, 10, 12, 13, 15
- **OpenAI API Key** — 03, 06, 10, 14
- **Google Sheets OAuth2** — 02, 04, 12, 13
- **Notion API** — 05, 14
- **HubSpot API** — 01, 15
- **Zendesk API** — 03, 15

### Environment Variables (set in n8n Settings → Variables)
```
WEBHOOK_SECRET=your-hmac-secret          # 03
OPENAI_API_KEY=sk-...                    # 14 (HTTP node)
LINKEDIN_PERSON_ID=xxxxxxx               # 06
INSTAGRAM_BUSINESS_ID=xxxxxxx            # 08
INSTAGRAM_ACCESS_TOKEN=EAAxx...          # 08
YOUTUBE_CHANNEL_ID=UCxxxxxxx             # 08
BEEHIIV_API_KEY=key_xxxx                 # 10
BEEHIIV_PUBLICATION_ID=pub_xxxx          # 10
PDFMONKEY_API_KEY=xxxx                   # 11
ENFORCE_SIGNATURE=true                   # 03
```

---

## How to Import

1. Open n8n dashboard
2. Click **Workflows** in the left sidebar
3. Click **+ New** → **Import from File**
4. Select the `.json` file
5. Configure credentials (n8n will highlight missing ones in red)
6. Update any IDs (Spreadsheet IDs, Database IDs, Channel IDs)
7. **Test** using the workflow's test payload from `meta.testingScenario`
8. Toggle **Active** to enable

---

## Testing Each Workflow

Each workflow JSON contains a `meta.testingScenario` object with:
- **happy_path** — the normal success flow to verify
- **test_payload** — exact JSON to POST to the webhook
- **edge_cases** — scenarios to test after happy path works

For scheduled workflows (02, 06, 10, 12, 13), use the **Execute Workflow** button to test manually without waiting for the schedule.

---

## Security Notes

- All webhooks should be protected with HMAC signatures where possible (see workflow 03 pattern)
- Store all API keys in n8n credentials, never hardcode in Code nodes
- Set `ENFORCE_SIGNATURE=true` in production environments
- Webhooks are unauthenticated by default in n8n — add IP allowlisting in n8n proxy/reverse proxy
- For public-facing webhooks, add rate limiting at the reverse proxy level (nginx/Cloudflare)
