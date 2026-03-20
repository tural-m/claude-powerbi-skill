# DAX Optimization Reference

Use this file during Block 7 (Performance & Refresh Optimization) when detecting and rewriting inefficient DAX patterns.

---

## Anti-Pattern 1: Nested CALCULATE

**Problem:** CALCULATE inside CALCULATE causes redundant context transitions and is hard to debug.

```dax
-- ❌ Anti-pattern
TotalRevenue =
CALCULATE(
    CALCULATE(
        SUM(Sales[Revenue]),
        Sales[Status] = "Completed"
    ),
    DATEYTD(Calendar[Date])
)
```

```dax
-- ✅ Optimized: merge filter arguments
TotalRevenue =
CALCULATE(
    SUM(Sales[Revenue]),
    Sales[Status] = "Completed",
    DATEYTD(Calendar[Date])
)
```

---

## Anti-Pattern 2: FILTER() over full table instead of boolean

**Problem:** FILTER() iterates the entire table. For simple conditions, use boolean filter arguments instead.

```dax
-- ❌ Anti-pattern: iterates every row of Sales
SlowMeasure =
CALCULATE(
    SUM(Sales[Revenue]),
    FILTER(Sales, Sales[Region] = "North")
)
```

```dax
-- ✅ Optimized: boolean filter — no row iteration
FastMeasure =
CALCULATE(
    SUM(Sales[Revenue]),
    Sales[Region] = "North"
)
```

Use FILTER() only when you need row-level logic that cannot be expressed as a simple boolean (e.g. comparing two columns from the same row).

---

## Anti-Pattern 3: Iterator over large unreduced table

**Problem:** SUMX/AVERAGEX iterating large tables without prior filtering causes full scans.

```dax
-- ❌ Anti-pattern: iterates all 1.8M rows of Sales
Margin =
SUMX(
    Sales,
    Sales[Revenue] - Sales[Cost]
)
```

```dax
-- ✅ Option A: pre-filter before iterating
Margin =
SUMX(
    FILTER(Sales, Sales[Status] = "Completed"),
    Sales[Revenue] - Sales[Cost]
)

-- ✅ Option B: add calculated column at model level if margin is always needed
-- Sales[Margin] = Sales[Revenue] - Sales[Cost]
-- Then: SUM(Sales[Margin])
```

---

## Anti-Pattern 4: Bidirectional relationships on large tables

**Problem:** Bidirectional cross-filter allows filters to propagate both ways, causing ambiguous results and performance degradation.

**Detection:** Query MCP for relationships where `crossFilteringBehavior = "bothDirections"` on tables with >100K rows.

**Fix options:**
- Switch to single-direction and use CROSSFILTER() in specific measures only
- Use bridge tables to handle many-to-many without bidirectional filters

---

## Anti-Pattern 5: High-cardinality column in relationships

**Problem:** Joining on a text column with millions of unique string values is slow.

**Detection:** Flag relationship keys where `distinctCount > 500,000` and data type is text.

```dax
-- ❌ Joining on: Sales[OrderID] as text (e.g. "ORD-2024-00184729")
-- ✅ Add integer surrogate key: Sales[OrderKey] as INT
```

**Fix:** Add integer surrogate key in Power Query, update relationship.

---

## Anti-Pattern 6: Using ALL() when REMOVEFILTERS() is clearer

```dax
-- ❌ Ambiguous intent
ShareOfTotal =
DIVIDE(
    SUM(Sales[Revenue]),
    CALCULATE(SUM(Sales[Revenue]), ALL(Sales))
)

-- ✅ Explicit intent
ShareOfTotal =
DIVIDE(
    SUM(Sales[Revenue]),
    CALCULATE(SUM(Sales[Revenue]), REMOVEFILTERS(Sales))
)
```

---

## Anti-Pattern 7: Time intelligence without a proper date table

**Problem:** DATEYTD, SAMEPERIODLASTYEAR, etc. require a contiguous date table marked as a date table in the model.

**Detection checks:**
- Is there a dedicated Calendar/Date table?
- Is it marked as a Date Table in Power BI?
- Does it have no gaps in dates?
- Does the relationship to fact table use the date column (not datetime)?

If any check fails — flag before generating time intelligence DAX.

---

## Query Folding Anti-Patterns (Power Query)

These Power Query transformations break query folding, causing the entire table to be downloaded and processed locally:

| Transformation | Breaks folding? | Alternative |
|----------------|-----------------|-------------|
| Table.AddColumn with custom function | Yes | Use native column operations |
| List.Contains() in row filter | Yes | Use Table.SelectRows with direct comparison |
| Type conversion after filter | Sometimes | Convert before filtering |
| Merging queries from different sources | Yes | Accept — document as known limitation |

**How to check:** Right-click any step in Power Query → "View Native Query". If grayed out, folding is broken at that step.

---

## Performance Diagnostic Checklist

Run through this during Block 7:

- [ ] Row count per table retrieved from MCP
- [ ] DirectQuery tables identified — latency risk flagged if >500K rows
- [ ] Nested CALCULATE patterns scanned
- [ ] Iterator functions (SUMX, AVERAGEX, RANKX) on large tables flagged
- [ ] Bidirectional relationships on large tables flagged
- [ ] High-cardinality relationship keys identified
- [ ] Date table validity confirmed (contiguous, marked, no gaps)
- [ ] Query folding broken steps identified in Power Query
- [ ] Incremental refresh opportunity identified for large fact tables
