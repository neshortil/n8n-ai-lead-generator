# Setup Guide & Common Pitfalls - v1.0 (Google Maps Edition)

> This guide covers the setup of **v1.0** of the n8n Google AI Lead Generator.  
> Lead source for this version: **Google Maps**.

---

## Quick Start (Step by Step)

### Step 1 - Install n8n

**Option A: Docker (recommended for self-hosted)**
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

**Option B: npm**
```bash
npm install n8n -g
n8n start
```

**Option C: n8n Cloud**
Sign up at [app.n8n.cloud](https://app.n8n.cloud) — no installation needed.

---

### Step 2 - Import Workflows

Import in this exact order to avoid broken workflow references:

1. `[IL] AgentLeadAddQuery.json`
2. `[IL] AgentLeadAddSiteCompany.json`
3. `[IL] AgentLeadScrapInformationCompany.json`
4. `[IL] AgentCheckMail.json`
5. `[IL] AgentLeadMailGenerate.json`
6. `[IL] AgentSendMail.json`
7. `[IL] GOOGLE LEAD GENERATOR.json` ← **last**

> **Why this order?** The main workflow references sub-workflows by ID. Importing sub-workflows first ensures the Tool nodes in the main workflow can resolve them correctly.

How to import:
1. Open n8n -> **Workflows** -> **Import from File** (or drag & drop)
2. Select the `.json` file from the `/workflows` folder
3. Save and repeat for each file

---

### Step 3 - Connect Credentials

Go to **Settings -> Credentials** in n8n and create:

| Credential | Type | Notes |
|---|---|---|
| OpenAI | `openAiApi` | Used in all AI nodes |
| Telegram | `telegramApi` | Bot token from @BotFather |
| Google Sheets | `googleSheetsOAuth2Api` | Enable Google Sheets API in GCP |
| Gmail | `gmailOAuth2` | Enable Gmail API in GCP |
| Google Maps | `googleMapsApi` | Enable Google Maps API in GCP |

**Google Cloud Setup:**
1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project or use existing
3. Enable **Google Sheets API** and **Gmail API**, **Google Maps API**
4. Create OAuth 2.0 credentials
5. Add your n8n instance URL as authorized redirect URI

---

### Step 4 - Prepare Google Sheets

**Sheet 1 - Query & Company Database**

Create a Google Sheet with two tabs:

| Tab Name | Columns |
|---|---|
| `Queries` (gid=0) | `query` |
| `SiteCompany` | `name`, `address`, `phone`, `website` |

**Sheet 2 - Leads Database**

Create a second Google Sheet with one tab:

| Tab Name | Columns |
|---|---|
| `Final` | `name`, `email`, `website`, `mail`, `create mail?`, `DM?` |

After creating both sheets:
- Copy their document IDs from the URL (`/spreadsheets/d/YOUR_ID_HERE/`)
- Update the `documentId` field in every Google Sheets node across all workflows

> **Tip:** Add a `rownumber` column to the Final sheet. It helps the email generator scope memory per lead.

---

### Step 5 - Configure Telegram Bot

1. Message [@BotFather](https://t.me/BotFather) -> `/newbot`
2. Name your bot and copy the token
3. In n8n: update Telegram credential with your token
4. Activate the main workflow — n8n registers the webhook automatically
5. Start chatting with your bot

**Optional - Restrict bot to your Telegram ID only:**
Add an IF node after the Telegram Trigger that checks:
```
{{ $json.message.from.id }} == YOUR_TELEGRAM_USER_ID
```
This prevents others from using your bot.

---

### Step 6 - Update Email Template

In `[IL] AgentLeadMailGenerate`:
1. Open the `add email and html` node (Edit Fields)
2. Replace `defaultemail` value with your real sender email
3. Replace `htmltemplate` value with your own Brevo HTML template (or keep the provided one)

To get a Brevo template:
1. Sign up at [brevo.com](https://www.brevo.com)
2. Go to Email Templates -> Create Template
3. Design your template, export raw HTML
4. Paste into the `htmltemplate` field

---

### Step 7 - Activate & Run

1. Activate all 6 sub-agent workflows (toggle ON)
2. Activate `[IL] GOOGLE LEAD GENERATOR` last
3. Send a Telegram message to your bot: `collect dentists Kyiv`

---

## Common Pitfalls

### Telegram Bot Configuration

| Issue | Cause | Fix |
|---|---|---|
| Bot doesn't respond | Webhook not registered | Deactivate and reactivate the main workflow |
| Voice messages not transcribed | Telegram `get file` node credential mismatch | Ensure same Bot Token in all Telegram nodes |
| Bot replies to all users | No user filter | Add IF node checking `message.from.id` against allowed IDs |
| Webhook conflict error | Another process registered same webhook | Delete webhook via `https://api.telegram.org/botTOKEN/deleteWebhook` |

---

### OpenAI Credentials

| Issue | Cause | Fix |
|---|---|---|
| HTTP 401 in AgentLeadAddQuery | API key hardcoded and expired | Replace HTTP Request auth with n8n OpenAI credential |
| Model not found | Placeholder model name | Change to `gpt-4o-mini` or `gpt-4o` in AgentLeadMailGenerate |
| High token costs | GPT-4-turbo used in query generator | Downgrade to `gpt-4o-mini` if query quality is acceptable |
| Rate limit errors | Too many requests | Add Wait nodes between API calls |

---

### Google Sheets Permissions

| Issue | Cause | Fix |
|---|---|---|
| 403 Forbidden | OAuth scope missing | Re-authorize with `spreadsheets` scope enabled |
| Wrong sheet written | Document ID not updated | Update all `documentId` fields after importing |
| `create mail?` filter returns 0 rows | Column missing or wrong value | Add column manually and set value `0` for unprocessed rows |
| Append creates duplicates | Dedup not running | Ensure `Remove Duplicates` node is active in AgentLeadAddQuery |
| Quota exceeded | Too many reads/writes | Add Wait nodes to slow down batch operations |

---

### Gmail Sending Limits

| Issue | Cause | Fix |
|---|---|---|
| Emails stop sending mid-batch | Daily quota exceeded (500/day free) | Add counter + stop logic, or use Google Workspace (2000/day) |
| Emails go to spam | New account, high volume | Warm up Gmail: start with 20/day, increase gradually |
| OAuth token expired | Gmail token expires if unused | Re-authenticate or use a service account |
| `mail` column is blank | Mail generator not run first | Always run `AgentLeadMailGenerate` before `AgentSendMail` |

---

### Google Maps Scraping (v1.0 specific)

| Issue | Cause | Fix |
|---|---|---|
| Google Maps returns no results | Wrong query format | Check query format: `niche + city` (e.g., `dentists Kyiv`) |
| Scraping returns empty | Rate limiting or CAPTCHA | Add Wait nodes; reduce batch size |
| Missing website column | Business has no website listed | Handle nulls with IF nodes in downstream workflows |
| Duplicate companies | Same business from different queries | Enable dedup logic before writing to SiteCompany tab |

---

### n8n Memory Limits

| Issue | Cause | Fix |
|---|---|---|
| Agent forgets previous commands | Memory window too small | Increase `contextWindowLength` from 10 to 20 in Simple Memory node |
| Memory bleeds between users | Wrong session key | Session key must be `message.chat.id` |
| Workflow runs out of memory | Large payloads in loops | Use pagination and limit batch sizes |

---

### Workflow Execution Errors

| Issue | Cause | Fix |
|---|---|---|
| Sub-agent not found by AI Agent | Workflow inactive or wrong ID | Activate sub-workflow and check Tool node references |
| Loop runs forever | Missing exit condition | Ensure IF nodes have both true/false branches connected |
| Scraping returns empty | Website blocked or no email on page | Add error handling, log failed domains |

---

## Recommended Workflow Testing Order

```
1. Test AgentLeadAddQuery manually (Execute Workflow)
   -> Check: queries appear in Google Sheets

2. Test AgentLeadAddSiteCompany manually
   -> Check: companies appear in SiteCompany tab (from Google Maps)

3. Test AgentLeadScrapInformationCompany manually
   -> Check: emails and info appear in Final tab

4. Test AgentCheckMail manually
   -> Check: DM? column updated for valid emails

5. Test AgentLeadMailGenerate manually (1-2 rows)
   -> Check: mail column populated with HTML

6. Test AgentSendMail manually (1 row)
   -> Check: email received in inbox

7. Test full flow via Telegram
   -> Send: "collect dentists Kyiv"
   -> Monitor: workflow executions in n8n
```

---

🗺️ v1.0 — Google Maps Edition | Released: Summer 2025 | Future: v2.0 (Smart Bot + PostgreSQL), v3.0 (SaaS)
