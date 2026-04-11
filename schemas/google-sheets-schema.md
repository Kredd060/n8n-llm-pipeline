# Google Sheets Hub — Schema Reference

## Sheet Name: `Pipeline`

This sheet acts as the central database for the 4-LLM pipeline. Every execution appends one row, then updates it in-place as the pipeline progresses.

---

## Column Definitions (13 Columns)

| # | Column Name | Type | Set By | Description |
|---|-------------|------|--------|-------------|
| A | **Row ID** | String | Initialize Pipeline | Unique identifier. Format: `ROW-{unix_timestamp}`. Used as the match key for updates. |
| B | **Timestamp** | String (ISO 8601) | Initialize Pipeline | UTC datetime of pipeline start. |
| C | **Original Prompt** | String | Initialize Pipeline | The raw business prompt submitted to the webhook. |
| D | **ChatGPT Idea** | String | Gemini 1.5 Flash | The Gemini-formatted version of the GPT-4o Mini idea, including category and confidence score tags. |
| E | **Cycle 1 Critique** | String | Parse Claude Decision (Cycle 1) | Claude's critique from cycle 1. |
| F | **Cycle 2 Critique** | String | Parse Claude Decision (Cycle 2) | Claude's critique from cycle 2. Empty if fewer than 2 cycles ran. |
| G | **Cycle 3 Critique** | String | Parse Claude Decision (Cycle 3) | Claude's critique from cycle 3. Empty if fewer than 3 cycles ran. |
| H | **Final Concept** | String | Parse Claude Decision (final cycle) | The approved/refined business concept. |
| I | **Status** | String | Multiple stages | `Processing` → `Completed` |
| J | **Cycles Used** | Number | Google Sheets Update | Total Claude critique cycles executed (1, 2, or 3). |
| K | **Research Summary** | String | Parse Perplexity Response | Perplexity Sonar's executive research summary. |
| L | **Sources** | String | Parse Perplexity Response | Competitor names and citation URLs. Pipe-delimited. |
| M | **Notes** | String | Multiple stages | Initial: `Pipeline started`. Final: Perplexity viability verdict. |

---

## Setup Instructions

### 1. Create the Spreadsheet
1. Go to [Google Sheets](https://sheets.google.com) and create a new spreadsheet.
2. Rename the default sheet tab to **`Pipeline`** (exactly — case-sensitive).
3. Copy your Spreadsheet ID from the URL:
   ```
   https://docs.google.com/spreadsheets/d/YOUR_SPREADSHEET_ID/edit
   ```

### 2. Add the Header Row
In row 1, enter these exact column headers (A through M):

```
Row ID | Timestamp | Original Prompt | ChatGPT Idea | Cycle 1 Critique | Cycle 2 Critique | Cycle 3 Critique | Final Concept | Status | Cycles Used | Research Summary | Sources | Notes
```

> **Important:** Column names must match exactly — the n8n workflow maps to them by name.

### 3. Recommended Formatting
- **Column A (Row ID):** Freeze column, bold header
- **Column B (Timestamp):** Format as plain text (not date)
- **Columns E-G (Critiques):** Set wrap text, row height ~60px
- **Column I (Status):** Conditional formatting: `Processing` -> Yellow, `Completed` -> Green
- **Column J (Cycles Used):** Center-align, number format

### 4. Update the n8n Workflow
Replace `REPLACE_WITH_YOUR_SPREADSHEET_ID` in both Google Sheets nodes with your actual Spreadsheet ID.

---

## Example Row (Completed Pipeline Run)

| Column | Example Value |
|--------|---------------|
| Row ID | `ROW-1712672580123` |
| Timestamp | `2026-04-09T14:23:00.123Z` |
| Original Prompt | `A subscription service that helps small restaurants reduce food waste` |
| ChatGPT Idea | `**Core Idea:** AI-powered inventory management... [Category: FoodTech] [Complexity: Medium] [Confidence: 78%]` |
| Cycle 1 Critique | `Solid concept but lacks clear monetization. | Key Risks: Low switching urgency | Decision: Needs clarity` |
| Cycle 2 Critique | `Tiered pricing addresses the gap. | Key Risks: SMB churn | Decision: Approve` |
| Cycle 3 Critique | *(empty — approved in cycle 2)* |
| Final Concept | `A SaaS platform for AI-driven food waste analytics, priced at $149/mo.` |
| Status | `Completed` |
| Cycles Used | `2` |
| Research Summary | `Restaurant food waste SaaS is a $2.1B market. Market Size: $2.1B TAM | Trends: Cost-conscious operators...` |
| Sources | `Competitors: Winnow, Leanpath | https://...` |
| Notes | `Viability: Viable` |
