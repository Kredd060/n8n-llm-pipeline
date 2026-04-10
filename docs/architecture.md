# Pipeline Architecture

## Overview

The 4-LLM Autonomous Business Pipeline is a 5-stage autonomous workflow that takes a raw business prompt and returns a researched, critiqued, and validated business concept — without human intervention between stages.

```
WEBHOOK INPUT
     │
     ▼
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 0 — Initialize                                               │
│  Node: Initialize Pipeline                                          │
│  • Validates prompt                                                 │
│  • Creates Row ID (ROW-{timestamp})                                 │
│  • Seeds all 13 pipeline state fields                               │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 1 — Idea Generation (GPT-4o Mini)                            │
│  Nodes: GPT-4o Mini → Parse GPT Response                            │
│  • Generates structured business idea from prompt                  │
│  • Output fields: core_idea, target_market, value_proposition,     │
│    revenue_model, implementation_steps                              │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 2 — Format & Log (Gemini 1.5 Flash + Google Sheets)          │
│  Nodes: Gemini → Parse Gemini → Google Sheets (Append)             │
│  • Gemini reformats and scores the GPT idea                        │
│  • Output: formatted_concept, category, complexity, confidence     │
│  • Google Sheets: Creates initial row with 13 columns              │
│    (critique columns empty, status = "Processing")                 │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 3 — Critique Loop (Claude) — MAX 3 CYCLES                   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Increment Cycle Counter                                    │   │
│  │       ↓                                                     │   │
│  │  Build Claude Prompt (with all prior context)               │   │
│  │       ↓                                                     │   │
│  │  Claude - Critique & Decide                                 │   │
│  │  • Receives: prompt, GPT idea, Gemini format, prior cycles  │   │
│  │  • Returns: approved (bool), critique, final_concept,       │   │
│  │             key_risks, approval_reason                      │   │
│  │       ↓                                                     │   │
│  │  Parse Claude Decision                                      │   │
│  │  • Stores critique in critique1/2/3 slot                   │   │
│  │       ↓                                                     │   │
│  │  IF: approved=true OR cycleCount>=3 ?                       │   │
│  │     YES ──────────────────────────────────────► continue   │   │
│  │     NO  ──────────────────────────────────────► (loop) ↑   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Loop Rules:                                                        │
│  • Cycle 1: Claude critiques fresh concept                         │
│  • Cycle 2: Claude sees Cycle 1 critique, iterates               │
│  • Cycle 3: FORCED FINAL DECISION — Claude must approve           │
└─────────────────────────┬───────────────────────────────────────────┘
                          │ (approved concept)
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 4 — Research & Validation (Perplexity Sonar)                 │
│  Nodes: Perplexity Sonar → Parse Perplexity                        │
│  • Receives: final approved concept + original prompt              │
│  • Returns: research_summary, market_size, key_competitors,        │
│             market_trends, viability_verdict, sources              │
└─────────────────────────┬───────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────┐
│  STAGE 5 — Final Write-Back + Response                              │
│  Nodes: Google Sheets (Update) → Respond to Webhook                │
│  • Updates the existing Sheets row (matched by Row ID)             │
│  • Returns JSON response to original HTTP caller                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## LLM Role Summary

| LLM | Model | Stage | Primary Role | Output |
|-----|-------|-------|-------------|--------|
| **GPT-4o Mini** | `gpt-4o-mini` | 1 | Creative ideation from prompt | Structured business idea JSON |
| **Gemini 1.5 Flash** | `gemini-1.5-flash` | 2 | Format, categorize, and score idea | Executive summary + metadata |
| **Claude** | `claude-opus-4-6` | 3 | Critical evaluation + final approval | Critique text, approved flag, refined concept |
| **Perplexity Sonar** | `sonar` | 4 | Real-time market research | Research summary, competitors, sources |

---

## State Object

The pipeline carries a single JSON state object through all nodes:

```json
{
  "rowId":           "ROW-1712672580123",
  "timestamp":       "2026-04-09T14:23:00.000Z",
  "prompt":          "The original user prompt",
  "gptIdea":         "Full GPT-4o Mini idea text",
  "geminiFormatted": "Gemini-structured summary [Category: X] [Confidence: 78%]",
  "cycleCount":      2,
  "critique1":       "Cycle 1 critique text | Key Risks: ... | Decision: ...",
  "critique2":       "Cycle 2 critique text | ...",
  "critique3":       "",
  "approved":        true,
  "finalConcept":    "The refined, Claude-approved concept statement",
  "researchSummary": "Market research from Perplexity",
  "sources":         "Competitors: X, Y | https://...",
  "notes":           "Viability: Viable",
  "status":          "Completed",
  "sheetsRowNum":    5
}
```

---

## Key Design Decisions

**Why `Restore Pipeline State` after Google Sheets?**
n8n's Google Sheets Append node replaces the incoming data with Sheets response metadata. The Restore node re-injects the full pipeline state so the critique loop has all necessary context.

**Why does the IF node use `combinator: "any"` (OR)?**
The loop exits when EITHER condition is true: Claude approves, OR max cycles are reached. Claude always makes the final concept decision — even on a forced exit at cycle 3.
