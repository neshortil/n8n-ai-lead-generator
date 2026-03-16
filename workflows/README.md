# Workflows

This folder contains all n8n workflow JSON files for the AI Lead Generator system.

## How to Import

1. Open your n8n instance
2. Go to **Workflows** -> **Import from File**
3. Select the `.json` file
4. Save the workflow
5. Connect your credentials
6. Activate the workflow

## Import Order

> Import sub-agent workflows FIRST, then the main orchestrator last.

| Order | Filename | Role |
|---|---|---|
| 1 | `[IL] AgentLeadAddQuery.json` | Query generator |
| 2 | `[IL] AgentLeadAddSiteCompany.json` | Google Maps scraper |
| 3 | `[IL] AgentLeadScrapInformationCompany.json` | Company info extractor |
| 4 | `[IL] AgentCheckMail.json` | Email validator |
| 5 | `[IL] AgentLeadMailGenerate.json` | HTML email generator |
| 6 | `[IL] AgentSendMail.json` | Gmail sender |
| 7 | `[IL] GOOGLE LEAD GENERATOR.json` | **Main orchestrator (import last)** |

## After Importing

- Update all **Google Sheets document IDs** in every Sheets node
- Connect **OpenAI**, **Telegram**, **Google Sheets**, and **Gmail** credentials
- Activate sub-agent workflows before activating the main workflow

## Notes

- Workflow JSON files are uploaded manually by the developer
- Do NOT rename the workflow files — the names must remain exactly as listed above
- See [docs/setup-guide.md](../docs/setup-guide.md) for full setup instructions
- See [docs/workflow-explanations.md](../docs/workflow-explanations.md) for detailed workflow documentation
