<div align="center">

# 🤖 Google Lead Generator - v1.0

**An AI-powered, fully automated Google Maps lead generation and outreach system built on n8n.**  
Control everything from Telegram. Speak or type — the AI does the rest.

> 🗺️ **v1.0 — Google Maps Edition** | Released: Summer 2025 | Future: v2.0 (Smart Bot + PostgreSQL), v3.0 (SaaS)

![n8n](https://img.shields.io/badge/Built%20with-n8n-orange?style=flat-square)
![OpenAI](https://img.shields.io/badge/AI-OpenAI%20GPT-blue?style=flat-square)
![Google Sheets](https://img.shields.io/badge/Database-Google%20Sheets-green?style=flat-square)
![Telegram](https://img.shields.io/badge/Interface-Telegram-2CA5E0?style=flat-square)
![Version](https://img.shields.io/badge/Version-v1.0-brightgreen?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)

</div>

---

## 🧠 What Is This?

This is **v1.0** of a production-ready, end-to-end **AI lead generation system** built entirely in [n8n](https://n8n.io/). First released in **Summer 2025**.

This version uses **Google Maps** as the primary lead source — the AI scrapes businesses by niche and city, enriches them, and sends automated outreach.

You send a command to a Telegram bot - via text or **voice message** — and the system:

1. Generates Google Maps search queries using GPT-4
2. Scrapes business listings from Google Maps
3. Collects company websites
4. Extracts detailed company information
5. Stores everything in Google Sheets
6. Generates personalized HTML outreach emails (with Brevo templates)
7. Validates email addresses
8. Sends cold outreach via Gmail

All orchestrated by a central **AI Agent** with memory, tools, and multi-step reasoning.

---

## 🗺️ Version Roadmap

| Version | Name | Lead Source | Key Upgrades | Status |
|---|---|---|---|---|
| **v1.0** | Google Maps Basic | Google Maps | n8n + GPT-4 + Google Sheets + Telegram | ✅ Released (Summer 2025) |
| v2.0 | Google Maps Pro | Google Maps | PostgreSQL + pgvector, smart AI chatbot with semantic memory & analysis | 🔜 In development |
| v3.0 | Google Maps SaaS | Google Maps | Full SaaS product, multi-tenant, dashboard, subscription billing | 🔜 Planned |

> All versions are built around **Google Maps** as the core lead discovery source.

---

## ✨ Key Features

- 🗣️ **Voice + Text Control** — Send voice memos or text commands via Telegram
- 🧠 **LLM Orchestrator** — GPT-4o-mini AI Agent decides which sub-agent to call
- 🗺️ **Google Maps Discovery** — Automated business scraping by niche + city
- 🌐 **Website Scraper** — Collects and stores company website URLs
- 📊 **Google Sheets CRM** — Two-sheet lead database (queries + enriched leads)
- ✉️ **HTML Email Generator** — GPT generates personalized emails from Brevo templates
- ✅ **Email Validation** — Checks and filters invalid or duplicate emails before sending
- 📤 **Gmail Outreach** — Sends emails directly through Gmail integration
- 💬 **Telegram Feedback** — Bot replies with status updates after every operation
- 🔁 **Loop + Dedup Logic** — Built-in deduplication and retry mechanisms

---

## 🏗️ System Architecture

```
User (Telegram: text or voice)
        ↓
[ IL ] GOOGLE LEAD GENERATOR  ← Main Orchestrator (v1.0)
        ↓
  AI Agent (GPT-4o-mini + Memory)
        ↓
┌───────────────────────────────────────────────────┐
│  Available Tools:                                 │
│                                                   │
│  AgentLeadAddQuery       → Generate queries       │
│  AgentLeadAddSiteCompany → Scrape Google Maps     │
│  AgentLeadScrapInformation → Extract company info │
│  AgentLeadMailGenerate   → Generate HTML emails   │
│  AgentCheckMail          → Validate emails        │
│  AgentSendMail           → Send via Gmail         │
│  QuerySheets             → Read query sheet       │
│  SiteCompanySheets       → Read company sheet     │
│  TelegramMessage         → Send status updates    │
│  Think                   → Internal reasoning     │
└───────────────────────────────────────────────────┘
        ↓
  Google Sheets (Lead Database)
        ↓
    Gmail (Outreach)
```

---

## 📋 Workflow Overview

| Workflow | Role | Trigger |
|---|---|---|
| `[IL] GOOGLE LEAD GENERATOR` | Main orchestrator | Telegram message/voice |
| `[IL] AgentLeadAddQuery` | AI search query generator | Called by AI Agent |
| `[IL] AgentLeadAddSiteCompany` | Google Maps scraper | Called by AI Agent |
| `[IL] AgentLeadScrapInformationCompany` | Company data extractor | Called by AI Agent |
| `[IL] AgentLeadMailGenerate` | HTML email generator | Called by AI Agent |
| `[IL] AgentCheckMail` | Email filter/validator | Called by AI Agent |
| `[IL] AgentSendMail` | Gmail email sender | Called by AI Agent |

---

## ⚡ Quick Start

### 1. Install n8n

```bash
# Docker (recommended)
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Or use [n8n Cloud](https://app.n8n.cloud/).

### 2. Import Workflows

1. Open n8n → **Workflows** → **Import from File**
2. Import all `.json` files from the `/workflows` folder
3. Import in this order:
   - Sub-agents first (AgentLeadAddQuery, AgentLeadAddSiteCompany, etc.)
   - Main workflow last (`[IL] GOOGLE LEAD GENERATOR`)

### 3. Connect Credentials

| Service | Credential Type | Where Used |
|---|---|---|
| OpenAI | API Key | All AI nodes |
| Telegram | Bot Token | Trigger + reply nodes |
| Google Sheets | OAuth2 | All Sheets nodes |
| Gmail | OAuth2 | AgentSendMail |
| Google Maps | API Key | AgentLeadAddSiteCompany |

### 4. Prepare Google Sheets

Create **two** Google Sheets:

**Sheet 1 - Query Database**
- Tab 1: `Queries` — columns: `query`
- Tab 2: `SiteCompany` — columns: `name`, `address`, `website`, `phone`

**Sheet 2 - Leads Database**
- Tab: `Final` — columns: `name`, `email`, `website`, `mail`, `create mail?`, `DM?`, `rownumber`

Update the Google Sheets document IDs in each workflow node.

### 5. Configure Telegram Bot

1. Create a bot via [@BotFather](https://t.me/BotFather)
2. Copy your bot token
3. Add the token to all Telegram credential nodes
4. Set the bot as webhook in n8n (automatic on activation)

### 6. Activate & Run

1. Activate all sub-agent workflows first
2. Activate `[IL] GOOGLE LEAD GENERATOR` last
3. Open Telegram and send your first command

---

## 🤖 Telegram Bot Commands

| Command | Description |
|--------|-------------|
| `Collect queries for [niche] in [city]` | Generate search queries for a specific business type and location |
| `Collect company websites` | Find websites based on generated queries |
| `Scrape company information` | Gather detailed info from company websites |
| `Generate emails for all leads` | Create draft emails for contacting companies |
| `Check emails before sending` | Filter duplicates and validate emails |
| `Send outreach emails` | Send emails to companies on the list |


> 💡 You can also send **voice messages** — the system transcribes them with OpenAI Whisper automatically.

---

## 📦 Requirements

- **n8n** v1.0+ (self-hosted or cloud)
- **OpenAI API Key** (GPT-4, GPT-4o-mini, Whisper)
- **Telegram Bot Token**
- **Google Account** with Sheets + Gmail OAuth2 + Google Maps Api Key
- **Brevo account** (optional — for HTML email templates)

---

## 📸 Screenshots

> Screenshots are located in `/images/`. See [docs/screenshots.md](docs/screenshots.md) for the full gallery.

| Preview | Description |
|---|---|
| `workflow-main.png` | Main orchestrator canvas |
| `workflow-query-generator.png` | Query generation sub-workflow |
| `workflow-scraper.png` | Scraper workflow |
| `workflow-email-generator.png` | Email generation workflow |
| `google-sheets-result.png` | Example output in Google Sheets |

---

## 🎥 Demo Videos

> Demo videos will be added here. See [docs/screenshots.md](docs/screenshots.md) for placeholders.

| Workflow | Demo |
|---|---|
| `[IL] GOOGLE LEAD GENERATOR` | YouTube Link: *(to be added)* |
| `[IL] AgentLeadAddQuery` | YouTube Link: *(to be added)* |
| `[IL] AgentLeadAddSiteCompany` | YouTube Link: *(to be added)* |
| `[IL] AgentLeadScrapInformationCompany` | YouTube Link: *(to be added)* |
| `[IL] AgentCheckMail` | YouTube Link: *(to be added)* |
| `[IL] AgentLeadMailGenerate` | YouTube Link: *(to be added)* |
| `[IL] AgentSendMail` | YouTube Link: *(to be added)* |

---

## 📁 Docs

- [Architecture Overview](docs/architecture.md)
- [Workflow Explanations](docs/workflow-explanations.md)
- [Setup Guide + Common Pitfalls](docs/setup-guide.md)
- [Screenshots](docs/screenshots.md)

---

## 📄 License

MIT — free to use, modify, and distribute.

---

<div align="center">

Built with ❤️ using n8n + OpenAI + Google Maps + Google Sheets

**v1.0 - Google Maps Edition - Summer 2025**

[github.com/neshortil/n8n-google-lead-generator](https://github.com/neshortil/n8n-google-lead-generator)

</div>
