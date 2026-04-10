# Setup Guide — 4-LLM Autonomous Business Pipeline

Complete this guide in order. Estimated setup time: 20–30 minutes.

---

## Prerequisites

- A running self-hosted n8n instance (v1.0+)
- Google account with Google Sheets access
- API accounts for: OpenAI, Google AI Studio, Anthropic, Perplexity

---

## Step 1 — Get Your API Keys

### OpenAI (GPT-4o Mini)
1. Go to [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Create a new secret key
3. Ensure your account has access to `gpt-4o-mini`
4. Copy the key — you won't see it again

### Google AI Studio (Gemini 1.5 Flash)
1. Go to [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)
2. Click **Create API Key**
3. Copy the key

### Anthropic (Claude)
1. Go to [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys)
2. Create a new API key
3. Copy the key

### Perplexity (Sonar)
1. Go to [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)
2. Generate an API key
3. Copy the key — it begins with `pplx-`

---

## Step 2 — Set Up Google Sheets

Follow the full instructions in [`schemas/google-sheets-schema.md`](../schemas/google-sheets-schema.md).

Summary:
1. Create a new Google Sheet
2. Rename tab to `Pipeline`
3. Add 13 column headers in row 1 (exact names required — see schema doc)
4. Copy your Spreadsheet ID from the URL

---

## Step 3 — Create n8n Credentials

In your n8n instance, go to **Settings → Credentials → Add Credential**.

Create the following 5 credentials:

### 3a. OpenAI API Key
- Type: **HTTP Header Auth**
- Name: `OpenAI API Key`
- Header Name: `Authorization`
- Header Value: `Bearer sk-YOUR_OPENAI_KEY`

### 3b. Google AI Studio API Key
- Type: **HTTP Header Auth**
- Name: `Google AI Studio API Key`
- Header Name: `x-goog-api-key`
- Header Value: `YOUR_GOOGLE_AI_STUDIO_KEY`

### 3c. Anthropic API Key
- Type: **HTTP Header Auth**
- Name: `Anthropic API Key`
- Header Name: `x-api-key`
- Header Value: `YOUR_ANTHROPIC_KEY`

### 3d. Perplexity API Key
- Type: **HTTP Header Auth**
- Name: `Perplexity API Key`
- Header Name: `Authorization`
- Header Value: `Bearer pplx-YOUR_PERPLEXITY_KEY`

### 3e. Google Sheets OAuth2
- Type: **Google Sheets OAuth2 API**
- Name: `Google Sheets OAuth2`
- Follow n8n's OAuth flow — you'll be redirected to sign in with Google

---

## Step 4 — Import the Workflow

1. In n8n, click **Workflows → Import from File**
2. Select `workflows/4-llm-business-pipeline.json`
3. The workflow will open in the editor

---

## Step 5 — Configure the Workflow

### 5a. Attach Credentials
For each HTTP Request node, open it and select the correct credential from the dropdown:
- `GPT-4o Mini - Generate Idea` → **OpenAI API Key**
- `Gemini 1.5 Flash - Format & Structure` → **Google AI Studio API Key**
- `Claude - Critique & Decide` → **Anthropic API Key**
- `Perplexity Sonar - Research & Validate` → **Perplexity API Key**

For each Google Sheets node:
- `Google Sheets - Log Initial Entry` → **Google Sheets OAuth2**
- `Google Sheets - Update Research Results` → **Google Sheets OAuth2**

### 5b. Set Your Spreadsheet ID
Open both Google Sheets nodes and replace `REPLACE_WITH_YOUR_SPREADSHEET_ID` with your actual Spreadsheet ID.

### 5c. Verify the Loop Connection
After import, verify that the **FALSE output** of `IF - Exit Loop` connects **back** to `Critique Loop - Increment Cycle`. This loop-back is the critique cycle mechanism. If it's broken, draw the connection manually.

The connection appears as a curved arrow going backwards in the canvas.

---

## Step 6 — Activate & Test

### Test with Curl
Once activated, your webhook URL will appear in the Webhook Trigger node. Test it:

```bash
curl -X POST https://YOUR_N8N_HOST/webhook/business-pipeline \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A mobile app that helps freelancers track their tax deductions automatically"}'
```

### Expected Response (JSON)
```json
{
  "success": true,
  "rowId": "ROW-1712672580123",
  "cyclesUsed": 2,
  "approved": true,
  "finalConcept": "A SaaS app that...",
  "researchSummary": "Market size...",
  "status": "Completed"
}
```

### Check Google Sheets
After the run, open your spreadsheet — a new row should appear with all 13 columns populated.

---

## Troubleshooting

| Issue | Likely Cause | Fix |
|-------|-------------|-----|
| Webhook returns 404 | Workflow not activated | Click the toggle to activate |
| OpenAI node fails | Wrong credential or billing | Check key, check OpenAI billing |
| Gemini returns 400 | API key in wrong header | Must be `x-goog-api-key`, not `Authorization` |
| Claude loop doesn't exit | Loop-back connection missing | Re-draw FALSE output of IF node → Increment node |
| Sheets node fails | OAuth not connected or wrong sheet name | Re-authenticate, check tab is named `Pipeline` |
| Perplexity returns 401 | Key format wrong | Value must be `Bearer pplx-...` (include "Bearer ") |

---

## Estimated API Costs Per Run

| Service | Model | Avg Tokens | Est. Cost |
|---------|-------|-----------|-----------|
| OpenAI | gpt-4o-mini | ~600 in/out | ~$0.001 |
| Google AI Studio | gemini-1.5-flash | ~800 in/out | ~$0.001 |
| Anthropic | claude-opus-4-6 | ~2,000 in/out × 2 avg cycles | ~$0.08 |
| Perplexity | sonar | ~1,500 in/out | ~$0.005 |
| **Total per run** | | | **~$0.09** |

> Costs are estimates and vary with prompt length and cycles used. Claude is the primary cost driver.
