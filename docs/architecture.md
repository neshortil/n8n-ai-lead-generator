# Architecture Overview — v1.0 (Google Maps Edition)

> This document describes the architecture of **v1.0** of the n8n Google AI Lead Generator.  
> v1.0 uses **Google Maps** as the primary lead discovery source.

---

## System Design

The n8n Google AI Lead Generator v1.0 follows a **hub-and-spoke architecture** where one central AI Agent orchestrator dispatches work to specialized sub-agent workflows.

All lead discovery in v1.0 is powered by **Google Maps scraping** — businesses are found by niche and city, then enriched and contacted automatically.

---

## High-Level Flow

```
User Input (Telegram)
        |
        v
+---------------------------+
|  [IL] GOOGLE LEAD         |
|  GENERATOR  (v1.0)        |
|  (Main Orchestrator)      |
|                           |
|  - Telegram Trigger       |
|  - Voice Transcription    |
|  - AI Agent (GPT-4o-mini) |
|  - Simple Memory (10 turns)|
+---------------------------+
        |
        v (tool calls)
+------+------+------+------+------+------+
|      |      |      |      |      |      |
v      v      v      v      v      v      v
Query  Site  Scrap  Mail  Check  Send  Utils
Gen   Scrape  Info  Gen   Mail   Mail  (Sheets,
                                       Telegram,
                                       Think)
```

---

## Component Breakdown

### 1. Main Orchestrator
**Workflow:** `[IL] GOOGLE LEAD GENERATOR` *(v1.0 entry point)*

- Entry point for all user commands
- Handles both text and voice input
- Maintains conversation memory per Telegram chat ID
- AI Agent reasons about intent and selects the right tool

### 2. Query Generator
**Workflow:** `[IL] AgentLeadAddQuery`

- Calls OpenAI GPT-4-turbo directly via HTTP Request
- Generates 50 Google Maps search queries per batch
- Deduplicates against existing queries in Google Sheets
- Outputs to: Google Sheets (Queries tab)

### 3. Google Maps Scraper
**Workflow:** `[IL] AgentLeadAddSiteCompany`

- Reads queries from Google Sheets
- Scrapes **Google Maps** for matching businesses *(core of v1.0)*
- Collects: name, address, phone, website
- Outputs to: Google Sheets (SiteCompany tab)

### 4. Company Info Extractor
**Workflow:** `[IL] AgentLeadScrapInformationCompany`

- Reads website URLs from SiteCompany tab
- Scrapes each company website for contact info
- Extracts: email addresses, descriptions, contact details
- Outputs to: Google Sheets (Final/Leads tab)

### 5. Email Generator
**Workflow:** `[IL] AgentLeadMailGenerate`

- Reads unprocessed leads (`create mail? = 0`)
- Uses GPT to generate personalized HTML emails
- Based on Brevo responsive HTML template
- Outputs to: `mail` column in Leads sheet

### 6. Email Validator
**Workflow:** `[IL] AgentCheckMail`

- Reads email column from Leads sheet
- Validates format, checks for duplicates
- Filters disposable domains
- Updates `DM?` column as gate for sending

### 7. Email Sender
**Workflow:** `[IL] AgentSendMail`

- Reads validated leads with generated emails
- Sends via Gmail OAuth2
- Updates send status in Leads sheet

---

## Data Flow

```
Google Sheets Structure:

Sheet 1: Query & Company Database
+------------------+
| Queries tab      |
| - query          |
+------------------+
| SiteCompany tab  |
| - name           |
| - address        |
| - phone          |
| - website        |
+------------------+

Sheet 2: Leads Database
+------------------+
| Final tab        |
| - name           |
| - email          |
| - website        |
| - mail           |
| - create mail?   |
| - DM?            |
| - rownumber      |
+------------------+
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| Automation Engine | n8n v1.0+ |
| LLM | OpenAI GPT-4o-mini / GPT-4-turbo |
| Voice Transcription | OpenAI Whisper |
| Control Interface | Telegram Bot API |
| Lead Discovery (v1.0) | **Google Maps Scraping** |
| Lead Storage | Google Sheets API |
| Email Delivery | Gmail OAuth2 |
| Email Templates | Brevo HTML |

---

## Credential Requirements

| Service | Credential Type | Scope Needed |
|---|---|---|
| OpenAI | API Key | All models access |
| Telegram | Bot Token | Message read/write |
| Google Sheets | OAuth2 | spreadsheets |
| Gmail | OAuth2 | gmail.send |

---

## Scalability Notes

- Each sub-workflow runs independently and can be triggered manually for testing
- Google Sheets acts as a simple CRM in v1.0 — can be replaced with Airtable or a real DB for scale
- The AI Agent memory window (10 turns) is configurable in the Simple Memory node
- Gmail daily limits (500 free / 2000 Workspace) cap outreach volume per day
- Add rate limiting `Wait` nodes for large batch operations

---

## Version Notes

| Version | Lead Source | Architecture Changes |
|---|---|---|
| **v1.0 (current)** | Google Maps | Hub-and-spoke, 7 workflows, Google Sheets CRM |
| v2.0 (planned) | LinkedIn | New scraper agents, additional enrichment |
| v3.0 (planned) | Multi-source | Lead scoring, source merging, expanded CRM |
