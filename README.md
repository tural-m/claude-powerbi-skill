# claude-powerbi-skill

> A lean, intent-driven AI analytics assistant for Claude — built to catch data problems before they corrupt analysis, then get out of the way.

## Overview

| | |
|---|---|
| **Domain** | Business Intelligence · Data Analytics |
| **Tools** | Claude AI · Power BI MCP · DAX · Power Query · VS Code |
| **Trigger** | Any Power BI, DAX, data model, or analytics workflow in Claude |
| **Design** | Expert-speed · intent-driven · proceed-and-warn |

---

## The Problem This Solves

Working with Claude AI + MCP + Power BI on a real project, a lot of questions came up fast:

- How should an AI handle data quality before running analysis?
- What stops it from generating DAX against columns that don't exist?
- How do you ensure insights are backed by evidence, not just pattern-matched guesses?

I searched for an existing skill that handled this. Nothing out there. So I built one from scratch — and then rebuilt it three more times until it actually matched how real BI work happens.

---

## How It Works

No pipeline. No sequential blocks. No mode selection.

```
User asks anything
      ↓
Detect intent
      ↓
Run Data Clarity Check (Light)   ← always, runs in background
      ↓
Issues found? → auto-upgrade to Deep check
      ↓
Respond: answer + inline warnings + 3-line micro-summary + one soft suggestion
```

Every response is self-contained. Heavy modules (SQL validation, performance audit, lineage, documentation) are on demand — never automatic.

---

## Core Feature: Data Clarity Check

The single thing this skill does better than vanilla Claude.

**Light check (always runs)** — null %, key duplicates, relationship risks, MCP schema confirmation. One line inline. Takes seconds.

**Deep check (auto-triggered)** — runs automatically if Light finds a critical issue. Adds fuzzy label detection, full column profiling, outlier detection, filter context audit.

```
⚠️ Data clarity: customer_region 28% null [🔴 affects this analysis] · order_id 142 dupes [🟡]
```

**Warning levels:**
- 🔴 Critical — affects KPI correctness directly
- 🟡 Warning — may affect interpretation
- 🔵 Info — minor, noted once

No blocking. Ever. Analysts work with imperfect data intentionally — the skill warns and proceeds.

---

## DAX Generation

- Confirms schema via MCP before generating — if unavailable, makes smart labeled assumption
- Inline comments on every measure explaining logic
- Flags performance risks and filter context issues inline
- KPI Definition Card available on request — never forced

```dax
-- [assumed: Sales[revenue] = gross revenue — MCP returned no definition]
-- ⚠️ crosses many-to-many (Orders ↔ Products) — verify result
Total Revenue = CALCULATE(SUM(Sales[revenue]), Sales[status] = "Completed")
```

---

## Insight Generation — Two Layers

**Layer 1 `[data]`** — only what the numbers directly support. Evidence, magnitude, confidence score.

**Layer 2 `[hypothesis]`** — analyst inference, clearly labeled, always includes a validation suggestion.

```
[data]       Electronics revenue +22% QoQ
             $4.2M · 41% of total · 1,840 orders · conf: 0.82

[hypothesis] Likely seasonal effect — Q4 pattern visible in 2023 and 2024.
             → Validate: compare Q4 2022, check category mix shift
```

Stakeholders get something actionable. Credibility stays intact.

---

## Micro-Summary

Every response ends with three lines:

```
──────────────────────────────
Data:     1 critical (customer_region nulls), 1 warning (dupes)
Finding:  Electronics revenue +22% QoQ — high confidence
Next:     Want SQL validation or performance audit on this model?
──────────────────────────────
```

---

## On-Demand Modules

Triggered by user request — never automatic.

| Module | How to trigger |
|---|---|
| Consistency Check | *"check this measure"* / *"verify this result"* |
| SQL Validation | *"validate with SQL"* / *"cross-check against source"* |
| Performance Audit | *"audit performance"* / *"why is this slow"* |
| Lineage Trace | *"trace this column"* / *"what uses this measure"* |
| KPI Definition Card | *"show KPI card"* / *"define this metric formally"* |
| Documentation | *"document this model"* / *"generate data dictionary"* |

---

## Repo Structure

```
claude-powerbi-skill/
├── README.md
├── SKILL.md                          # Main skill file (v4)
├── references/
│   ├── dax-optimization.md           # DAX anti-patterns and rewrites
│   ├── lineage-patterns.md           # Common dependency structures
│   └── kpi-definitions.md            # KPI disambiguation examples
└── diagram/
    └── powerbi-skill-diagram.html    # Visual workflow diagram
```

---

## How to Install

1. Go to **Claude.ai → Settings → Skills**
2. Upload the `claude-powerbi-skill` folder
3. Start a conversation — the skill activates automatically when you mention Power BI, DAX, data models, or analytics workflows

---

## Version History

| Version | Design |
|---|---|
| v1 | Sequential pipeline · 9 blocks · 45 tasks · hard gates |
| v2 | + KPI Definition Cards · + causal claim rules · + filter context audit |
| v3 | + Fast mode · + MCP degraded mode · + SQL validation layer · + honest validation labels |
| v4 | Full rethink → intent-driven · proceed-and-warn · expert-speed · lean core |

---

## Author

**Tural Mansimov** 
[LinkedIn](https://linkedin.com/in/tural-m) · [GitHub](https://github.com/tural-m)
