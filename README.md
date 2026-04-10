# 4-LLM Autonomous Business Pipeline

An n8n workflow that autonomously generates, critiques, and validates business concepts using a 4-LLM pipeline — GPT-4o Mini, Gemini 1.5 Flash, Claude, and Perplexity Sonar.

---

## What It Does

Send a business prompt to a webhook. The pipeline runs autonomously:

1. **GPT-4o Mini** generates a structured business idea
2. **Gemini 1.5 Flash** formats and scores the idea, then logs it to Google Sheets
3. **Claude** critiques the concept in a loop (up to 3 cycles), making the final approval decision
4. **Perplexity Sonar** researches and validates the approved concept with live web data
5. Google Sheets is updated with all critiques, the final concept, research, and sources

The entire process runs without human intervention and completes in ~30–90 seconds.

---

## Repo Structure

```
n8n-llm-pipeline/
├── workflows/
│   ├── 4-llm-business-pipeline.json   ← Import this into n8n
│   └── README.md                       ← Import instructions
├── docs/
│   ├── setup-guide.md                  ← Full step-by-step setup
│   └── architecture.md                 ← Pipeline design & data flow
├── schemas/
│   └── google-sheets-schema.md         ← 13-column Sheets schema
├── .github/
│   └── workflows/
│       └── validate-json.yml           ← CI: validates workflow JSON on push
├── .gitignore
└── README.md
```

---

## Quick Start

**Prerequisites:** A running n8n instance (v1.0+), API keys for OpenAI, Google AI Studio, Anthropic, and Perplexity.

1. Set up your Google Sheet using [`schemas/google-sheets-schema.md`](schemas/google-sheets-schema.md)
2. Create 5 credentials in n8n (see [`docs/setup-guide.md`](docs/setup-guide.md))
3. Import `workflows/4-llm-business-pipeline.json` into n8n
4. Attach credentials and set your Spreadsheet ID in the two Google Sheets nodes
5. Verify the loop-back connection on the `IF - Exit Loop` node
6. Activate the workflow and test:

```bash
curl -X POST https://YOUR_N8N_HOST/webhook/business-pipeline \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A subscription box for local artisan food products"}'
```

---

## Credentials Required

| n8n Credential Name | Type | API |
|--------------------|------|-----|
| `OpenAI API Key` | HTTP Header Auth | OpenAI |
| `Google AI Studio API Key` | HTTP Header Auth | Google AI Studio |
| `Anthropic API Key` | HTTP Header Auth | Anthropic |
| `Perplexity API Key` | HTTP Header Auth | Perplexity |
| `Google Sheets OAuth2` | Google Sheets OAuth2 API | Google |

---

## Estimated Cost Per Run

~$0.09 per pipeline execution (Claude is the primary cost driver at ~$0.08 with 2 avg cycles).

---

## License

MIT
