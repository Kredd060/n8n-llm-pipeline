# Workflows

This directory contains all n8n workflow JSON files. Each file can be imported directly into n8n.

---

## Importing a Workflow into n8n

1. In n8n, click **Workflows** in the left sidebar
2. Click **Add Workflow** → **Import from File**
3. Select the `.json` file from this directory
4. The workflow opens in the canvas editor
5. Attach your credentials (see [`../docs/setup-guide.md`](../docs/setup-guide.md))
6. Replace any placeholder values (Spreadsheet IDs, etc.)
7. Click the **Active** toggle to enable the workflow

---

## Available Workflows

| File | Description | LLMs Used |
|------|-------------|-----------|
| `4-llm-business-pipeline.json` | 5-stage autonomous pipeline: webhook → idea gen → format → critique loop → research | GPT-4o Mini, Gemini 1.5 Flash, Claude, Perplexity Sonar |

---

## Workflow Naming Convention

When adding new workflows, follow this pattern:

```
{purpose}-{version}.json
```

Examples:
- `4-llm-business-pipeline-v2.json`
- `email-triage-pipeline-v1.json`
- `content-generation-pipeline-v1.json`

---

## Notes on Version Control

- Workflow JSON files are committed **without embedded credentials** — all sensitive values are stored in n8n's credential system and referenced by ID
- The `REPLACE_WITH_*` placeholders in node parameters must be filled in n8n after import — they are intentionally left blank here for security
- The `.gitignore` excludes `local-dev-*.json` — use that prefix for personal development versions you don't want committed
