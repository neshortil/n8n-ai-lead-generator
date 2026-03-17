# Workflow Explanations - v1.0 (Google Maps Edition)

> Detailed breakdown of every workflow in the **[IL] Google Lead Generator system - v1.0**.  
> All lead discovery in this version is based on **Google Maps**.

---

## 1. `[IL] GOOGLE LEAD GENERATOR`
*(Main Orchestrator — v1.0 entry point)*

### Purpose
The **main orchestrator** of the entire v1.0 system. Receives commands from Telegram (text or voice), routes them through an AI Agent, and dispatches work to the appropriate sub-agent tools.

### How It Works
1. **Telegram Trigger** listens for incoming messages
2. **Switch Node** detects whether the message is a voice note or text
   - Voice: `Telegram (get file)` -> `OpenAI Whisper` (transcribe) -> normalized text
   - Text: `Edit Fields` -> normalized text
3. **Limit Node** caps message length before passing to the agent
4. **AI Agent** (GPT-4o-mini) receives the text, reasons about intent, and calls the appropriate tool(s)
5. **Simple Memory** (window = 10 turns) maintains conversation context per Telegram chat ID
6. **Telegram1** sends the AI Agent's final response back to the user

### Inputs
- Telegram text message
- Telegram voice message (automatically transcribed)

### Outputs
- Telegram reply with operation status
- Side effects: data written to Google Sheets via sub-agents

### Tools Available to the AI Agent

| Tool | Description |
|---|---|
| `AgentLeadAddQuery` | Generates and stores Google Maps search queries |
| `AgentLeadAddSiteCompany` | Scrapes Google Maps, collects company websites |
| `AgentLeadScrapInformationCompany` | Extracts full company info |
| `AgentLeadMailGenerate` | Generates personalized HTML emails |
| `AgentCheckMail` | Filters and validates email addresses |
| `AgentSendMail` | Sends emails via Gmail |
| `QuerySheets` | Reads the query database from Google Sheets |
| `SiteCompanySheets` | Reads the company/site database from Google Sheets |
| `TelegramMessage` | Sends progress messages during long operations |
| `Think` | Internal scratchpad for multi-step reasoning |

### Dependencies
- OpenAI API (GPT-4o-mini, Whisper)
- Telegram Bot
- All 6 sub-agent workflows must be active

### Common Mistakes
- **Sub-agents not activated**: The AI Agent will fail silently if sub-workflows are inactive
- **Voice messages not transcribed**: Ensure Telegram file download and OpenAI credentials are both set
- **Memory not scoped**: Memory key uses `chat.id` — different users get isolated sessions

### Important Notes
- The AI Agent uses `systemMessage` with explicit instructions for each tool
- GPT-4o-mini is used for cost efficiency; upgrade to GPT-4o for more complex orchestration
- The `Think` tool allows the agent to reason before acting — keep it enabled

### Demo Video
**YouTube Link:** *(to be added)*

---

## 2. `[IL] AgentLeadAddQuery`

### Purpose
Generates a batch of **Google Maps search queries** for a given niche and city using GPT-4, deduplicates them, and appends them to the Query sheet in Google Sheets.

### How It Works
1. **Execute Workflow Trigger** receives input from the parent AI Agent
2. **HTTP Request** calls OpenAI GPT-4-turbo directly with a custom prompt:
   - Generates 50 search queries per request
   - Each query is city + niche specific (e.g., `dentists Kyiv`)
   - Temperature: 0.2, Top-p: 0.9, Max tokens: 4000
3. **Code Node** parses the `choices[0].message.content` string, strips quotes, splits by comma
4. **Split Out** explodes the array into individual items
5. **Remove Duplicates** deduplicates against the `contentArray` field
6. **Loop Over Items** processes each query one at a time
7. **Google Sheets1** checks if the query already exists in the sheet
8. **If Node** skips existing queries, appends only new ones
9. **Google Sheets** appends new queries to tab `gid=0` of the Query sheet
10. **Wait nodes** (1 second intervals) prevent Google Sheets API rate limiting

### Inputs
- Natural language query string from the AI Agent (e.g., `"dentists Kyiv"`)

### Outputs
- New rows appended to Google Sheets Query tab
- Returns link to the Google Sheets URL

### Dependencies
- OpenAI API Key (GPT-4-turbo called via direct HTTP)
- Google Sheets OAuth2

### Common Mistakes
- **Hardcoded API key in HTTP Request**: Replace with n8n credential reference before sharing publicly
- **Wrong sheet ID**: Update `documentId` to your actual Google Sheets document ID
- **Comma parsing fails**: If GPT returns newline-separated queries, update the Code node split logic

### Important Notes
- Uses **direct HTTP Request** to OpenAI - gives more control over raw response parsing
- Built-in deduplication prevents the same query being generated twice
- Retry on fail enabled for Google Sheets writes (5 second wait between retries)

### Demo Video
**YouTube Link:** *(to be added)*

---

## 3. `[IL] AgentLeadAddSiteCompany`
*(Core v1.0 scraping workflow — Google Maps)*

### Purpose
Scrapes **Google Maps** to discover businesses matching queries from the Query sheet, collects their website URLs, and stores raw company data in the SiteCompany sheet.

### How It Works
1. **Execute Workflow Trigger** receives call from parent AI Agent
2. Reads pending queries from the Query sheet
3. Runs **Google Maps scraping** for each query
4. Extracts: company name, address, phone, website URL
5. Writes results to the SiteCompany tab in Google Sheets
6. Reports progress back via TelegramMessage

### Inputs
- Triggered by AI Agent
- Reads queries from Google Sheets (Query tab)

### Outputs
- New rows in SiteCompany tab: `name`, `address`, `phone`, `website`

### Dependencies
- Google Sheets OAuth2
- Google Maps scraping mechanism
- Telegram credentials (for progress messages)

### Common Mistakes
- **No queries in sheet**: Run `AgentLeadAddQuery` first before calling this workflow
- **Google Maps rate limiting**: Add wait nodes if scraping large batches
- **Missing website column**: Some businesses have no website — handle nulls in downstream workflows

### Important Notes
- This is the **primary lead source for v1.0** - all leads come from Google Maps
- This workflow feeds data that `AgentLeadScrapInformationCompany` depends on
- Described in the AI Agent as: *"Collects company websites and adds them to an Excel file"*

### Demo Video
**YouTube Link:** *(to be added)*

---

## 4. `[IL] AgentLeadScrapInformationCompany`

### Purpose
Takes company websites from the SiteCompany sheet and **scrapes full business information** (contact details, description, emails) to enrich the Leads database.

### How It Works
1. **Execute Workflow Trigger** receives call from parent AI Agent
2. Reads company records from the SiteCompany sheet
3. For each company website URL, performs a scrape:
   - Extracts email addresses
   - Extracts business description
   - Extracts contact information
4. Writes enriched data to the Leads sheet (`Final` tab)

### Inputs
- Triggered by AI Agent
- Reads from SiteCompany sheet (website URLs)

### Outputs
- Enriched lead records in the Leads Google Sheet (`Final` tab)
- Columns populated: `name`, `email`, `website`, and additional company info

### Dependencies
- Google Sheets OAuth2
- HTTP scraping capability
- SiteCompany sheet must be populated first

### Common Mistakes
- **Empty website field**: Skip rows with no website URL using an IF node check
- **Anti-scraping blocks**: Some websites block bots - implement User-Agent rotation or use a proxy
- **Email extraction fails**: Not all websites list emails publicly; expect partial fill rate

### Important Notes
- This is the most failure-prone step - build error handling and retries
- Described in AI Agent as: *"Collects information on companies and writes information into an Excel file"*

### Demo Video
**YouTube Link:** *(to be added)*

---

## 5. `[IL] AgentCheckMail`

### Purpose
**Validates and filters email addresses** in the Leads sheet before email generation or sending. Removes invalid, duplicate, or disposable email addresses.

### How It Works
1. **Execute Workflow Trigger** receives call from parent AI Agent
2. Reads email column from the Leads sheet
3. Applies validation logic:
   - Format validation (regex)
   - Duplicate detection
   - Disposable domain filtering
4. Updates rows with validation status

### Inputs
- Triggered by AI Agent
- Reads email column from the Leads Google Sheet

### Outputs
- Updated `DM?` or validation flag columns in the Leads sheet
- Clean list of sendable email addresses

### Dependencies
- Google Sheets OAuth2

### Common Mistakes
- **Running before scraping**: Ensure `AgentLeadScrapInformationCompany` has run first
- **Empty email column**: If email column is blank, the workflow processes zero rows
- **False positives**: Custom domains may be flagged — review the validation logic for your niche

### Important Notes
- Always run this **before** `AgentSendMail`
- The `DM?` column in the Leads sheet is the gate for send eligibility

### Demo Video
**YouTube Link:** *(to be added)*

---

## 6. `[IL] AgentLeadMailGenerate`

### Purpose
Generates **personalized HTML outreach emails** for each lead using GPT and a pre-designed Brevo HTML email template. Writes the generated email body back to the Leads sheet.

### How It Works
1. **Execute Workflow Trigger** receives call from parent AI Agent
2. `add email and html` node injects:
   - Default sender email (`defaultemail`)
   - A full Brevo-based responsive HTML template (`htmltemplate`)
3. **Google Sheets** filters leads where `create mail? = 0` (not yet generated)
4. **Loop Over Items** processes each lead
5. **If Node** checks if record is valid
6. **AI Agent** (GPT) generates professional HTML email:
   - Uses inline CSS, respects the Brevo template structure
   - Personalizes based on company name, website, context
   - Outputs only raw HTML, no markdown wrappers
7. **add text to mail** node writes `mail` column + sets `create mail? = 1`
8. **Error** node marks failed rows with error status
9. **Simple Memory** is scoped per `rownumber` for isolated per-lead context

### Inputs
- Triggered by AI Agent
- Reads from Leads sheet (`Final` tab), filtered by `create mail? = 0`
- Injects HTML template from `Edit Fields` node

### Outputs
- `mail` column in Leads sheet filled with generated HTML email body
- `create mail?` column updated to `1`

### Dependencies
- OpenAI API (GPT)
- Google Sheets OAuth2 (Leads sheet)
- Brevo HTML template (set in the `add email and html` node)

### Common Mistakes
- **Template not updated**: Replace the placeholder Brevo template with your own branded HTML
- **Wrong model name**: Update model to `gpt-4o-mini` or `gpt-4o`
- **`create mail?` column missing**: The filter depends on this column existing with value `0` for unprocessed rows
- **Memory conflict**: Memory is scoped by `rownumber` — ensure row numbers are unique and stable

### Important Notes
- The HTML template is a **full responsive email** with inline CSS — designed for high deliverability
- Default sender email is hardcoded in `Edit Fields` — update before production use

### Demo Video
**YouTube Link:** *(to be added)*

---

## 7. `[IL] AgentSendMail`

### Purpose
Sends the generated HTML emails to leads via **Gmail**, using the content from the `mail` column in the Leads sheet.

### How It Works
1. **Execute Workflow Trigger** receives call from parent AI Agent
2. Reads leads from the Leads sheet where email is valid and `mail` is populated
3. Sends each email via Gmail using the pre-generated HTML from the `mail` column
4. Updates send status in the Leads sheet

### Inputs
- Triggered by AI Agent
- Reads `email` and `mail` columns from the Leads Google Sheet

### Outputs
- Emails delivered to lead inboxes
- Send status updated in Leads sheet

### Dependencies
- Gmail OAuth2 credentials
- Google Sheets OAuth2
- `AgentLeadMailGenerate` must have run first (populates `mail` column)
- `AgentCheckMail` should run first (validates emails)

### Common Mistakes
- **Gmail daily send limit**: Free Gmail = 500/day, Google Workspace = 2000/day. Build in rate limiting
- **No HTML in `mail` column**: Running this before `AgentLeadMailGenerate` sends blank emails
- **OAuth token expiry**: Re-authenticate Gmail credentials if sends fail after extended period
- **Spam filters**: Sending too fast from a new account triggers spam — warm up the Gmail account first

### Important Notes
- Add a `Wait` node (1-3 second delay) between sends to avoid rate limiting
- Monitor Gmail sending quota — large lead lists can exhaust daily limits quickly

### Demo Video
**YouTube Link:** *(to be added)*

---

> 🗺️ **v1.0 — Google Maps Edition** | Released: Summer 2025 | Future: v2.0 (Smart Bot + PostgreSQL), v3.0 (SaaS)
