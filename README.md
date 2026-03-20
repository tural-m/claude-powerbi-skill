# Power BI AI Analytics Assistant — Claude Skill

> A structured, self-validating AI analytics assistant for Claude that covers the full Power BI lifecycle — from environment detection to a scored session summary with ranked business insights.

## Overview

| | |
|---|---|
| **Domain** | Business Intelligence · Data Analytics |
| **Tools** | Claude AI · Power BI MCP · DAX · Power Query · VS Code |
| **Trigger** | Any Power BI, DAX, data model, or analytics workflow in Claude |
| **Structure** | 9 blocks · 45 tasks · sequential workflow |

---

## The Problem This Solves

Working with Claude AI + MCP + Power BI on a real project, a lot of questions came up fast:

- How should an AI handle data quality *before* running analysis?
- What stops it from generating DAX against columns that don't exist?
- How do you ensure insights are backed by evidence, not just pattern-matched guesses?

No existing skill handled this. So this one was built from scratch.

---

## Workflow — 9 Blocks in Sequence

```
BLOCK 1  Environment Awareness       ← always run first
BLOCK 2  Data Model Validation       ← always run before analysis
BLOCK 3  Data Quality Audit          ← gates analysis if critical issues found ⛔
BLOCK 4  Analytical Assistance       ← on user request
BLOCK 5  Insight Generation          ← after analysis
BLOCK 6  Validation & Safety         ← runs continuously throughout
BLOCK 7  Performance & Refresh       ← advanced
BLOCK 8  Data Lineage & Dependencies ← advanced
BLOCK 9  Documentation & Summary     ← always run at session end
```

---

## Key Capabilities

**Data Model Validation**
Classifies tables (fact / dimension / bridge), detects schema type (star / snowflake / flat), audits all relationships, flags many-to-many, and computes a **Model Health Score (0–100)** across schema quality, data quality, performance, and documentation.

**Data Quality Audit with Gating**
Checks nulls, duplicates, inconsistent category labels, and outliers. If a critical issue is found (e.g. >25% missing values on a key column), analysis is blocked until it's resolved — not just warned about.

**Natural Language → DAX with Guardrails**
Before generating any DAX, the assistant clarifies ambiguous KPI definitions ("revenue" → gross or net? includes returns?), confirms every table and column name exists via MCP, and annotates generated code with inline comments.

**Hallucination Prevention**
Every column and table reference is confirmed via MCP before it appears in any generated code. If confirmation fails, the assistant stops and states why — it does not guess.

**Insight Scoring**
Every finding includes: absolute magnitude, % contribution to total, driving segment, trend validation (only labeled a "trend" if consistent across ≥3 periods and exceeds historical volatility), and a confidence score.

**Session Summary**
Every session closes with a full AI Assistant Summary: Model Health Score, ranked insights, data quality issues, performance risks, assumption log, and auto-generated documentation (data dictionary, table descriptions, measure definitions).

---

## Repo Structure

```
powerbi-analytics-assistant/
├── SKILL.md                          # Main skill file
├── references/
│   ├── dax-optimization.md           # DAX anti-patterns and rewrites
│   ├── lineage-patterns.md           # Common dependency structures
│   └── kpi-definitions.md            # KPI disambiguation examples
└── diagram/
    └── powerbi-skill-diagram.html    # Visual workflow diagram
```

---

## How to Use

1. Go to **Claude.ai → Settings → Skills**
2. Upload the `powerbi-analytics-assistant` folder as a skill
3. Start a new conversation — the skill activates automatically when you mention Power BI, DAX, data models, or analytics workflows

The skill runs blocks in sequence by default. You can also call any block directly — for example: *"Run a data quality audit on my current model"* triggers Block 3 only.

---

## Demo Minimum (3 things that must work)

1. Natural language → DAX with KPI guardrails (Blocks 4 + 6)
2. Data quality audit with gated report (Block 3)
3. AI Assistant Summary with ranked insights + confidence scores (Blocks 5 + 9)

---

## Author

**Tural Mansimov**  
[LinkedIn](https://linkedin.com/in/tural-m)
