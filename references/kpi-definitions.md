# KPI Definitions & Guardrail Reference

Use this file when validating KPI semantics before generating DAX measures, or when the user requests a KPI Definition Card ("show KPI card", "define this metric formally"). On-demand — not automatic.

---

## Core Principle

Never generate a measure for an ambiguous KPI term without confirming the definition.
Ask one clarifying question. Wait for confirmation. Then generate.

Format for clarification:
> "Before I generate this measure — when you say [term], do you mean:
> A) [definition A]
> B) [definition B]
> Which is correct, or is it something else?"

---

## Common Ambiguous Terms

### Revenue

| Term used | Possible meanings | Clarify |
|-----------|-------------------|---------|
| Revenue | Gross revenue (before returns/discounts) | Includes returns? Discounts deducted? |
| Net Revenue | After returns and discounts | Confirm discount treatment |
| Recognized Revenue | Accounting-period adjusted | Is this accrual or cash basis? |

**DAX impact:** SUM vs SUM minus returns table vs CALCULATE with adjustment filter

---

### Profit / Margin

| Term used | Possible meanings | Clarify |
|-----------|-------------------|---------|
| Profit | Gross profit (revenue - COGS) | Or operating profit (includes overhead)? |
| Margin | Gross margin % | Or net margin? Which costs included? |
| Contribution | Revenue minus variable costs only | Fixed costs excluded? |

**DAX impact:** Different columns, different tables, different filter contexts

---

### Count

| Term used | Possible meanings | Clarify |
|-----------|-------------------|---------|
| Count customers | COUNT (includes nulls + duplicates) | Or DISTINCTCOUNT (unique only)? |
| Count orders | Total rows in Orders table | Or distinct order_id? |
| Active customers | All customers | Or only those with activity in last N days? |

```dax
-- COUNT: counts all non-blank values (may include duplicates)
COUNT(Sales[CustomerID])

-- DISTINCTCOUNT: unique values only
DISTINCTCOUNT(Sales[CustomerID])

-- COUNTROWS: counts rows (use over filtered table)
COUNTROWS(FILTER(Customers, Customers[LastPurchaseDate] >= DATE(2024,1,1)))
```

---

### Growth

| Term used | Possible meanings | Clarify |
|-----------|-------------------|---------|
| Growth | vs prior month | Or vs same period last year? Or vs budget? |
| YoY growth | Calendar year | Or fiscal year? |
| QoQ growth | Sequential quarters | Which calendar? |

**Always confirm:** comparison period AND calendar type (fiscal vs calendar).

```dax
-- Month over Month
MoM_Growth =
DIVIDE(
    [TotalRevenue] - CALCULATE([TotalRevenue], DATEADD(Calendar[Date], -1, MONTH)),
    CALCULATE([TotalRevenue], DATEADD(Calendar[Date], -1, MONTH))
)

-- Year over Year (requires date table marked as Date Table)
YoY_Growth =
DIVIDE(
    [TotalRevenue] - CALCULATE([TotalRevenue], SAMEPERIODLASTYEAR(Calendar[Date])),
    CALCULATE([TotalRevenue], SAMEPERIODLASTYEAR(Calendar[Date]))
)
```

---

### Churn

| Term used | Possible meanings | Clarify |
|-----------|-------------------|---------|
| Churn | Customers who cancelled subscription | Or customers inactive for N days? |
| Churn rate | % of customers lost vs start of period | Which period? What defines "lost"? |
| At-risk | Customers approaching churn threshold | What is the threshold? |

**Minimum fields required for churn analysis:**
- `customer_id`
- `last_activity_date` or `subscription_end_date`
- Clear definition of "churned" (e.g. no activity in 60 days)

---

### Retention

| Term used | Possible meanings | Clarify |
|-----------|-------------------|---------|
| Retention rate | % of customers retained vs prior period | Cohort-based or snapshot? |
| Day-N retention | % of users active N days after acquisition | Requires event-level data |

**Minimum fields required:**
- `customer_id`
- `acquisition_date` or `first_purchase_date`
- `activity_date` or `last_purchase_date`

---

### Sales

| Term used | Possible meanings | Clarify |
|-----------|-------------------|---------|
| Sales | Total invoiced amount | Or shipped? Or delivered? |
| Units sold | Quantity column | Or number of transactions? |
| Average order value | Revenue / order count | Or revenue / line item count? |

---

## Currency & Aggregation Guards

Before summing any monetary column:

1. **Check for multi-currency** — does the dataset include multiple currencies?
   - If yes: sum requires currency conversion first
   - Never SUM(Revenue) across different currencies without conversion

2. **Check aggregation direction** — can this column be meaningfully summed?
   - Revenue: ✅ summable
   - Price per unit: ❌ not summable — use AVERAGE or SUMX(table, price * quantity)
   - Percentage columns: ❌ not summable — recalculate from base metrics

3. **Check for negative values** — returns, refunds, adjustments
   - May need to be subtracted, not added
   - Confirm treatment before generating SUM

---

## Clarification Script Templates

Use these when detecting ambiguity during DAX generation:

**For count ambiguity:**
> "When you say 'number of customers' — should I count all customer records, or only distinct unique customers? These can differ if a customer appears in multiple rows."

**For period ambiguity:**
> "When you say 'last quarter' — do you mean the most recently completed calendar quarter, or the last 90 days rolling? Also, does your business use a fiscal calendar different from the standard calendar year?"

**For metric ambiguity:**
> "When you say 'revenue growth' — should I compare to the same quarter last year, or to the previous quarter? And should I use gross revenue or net revenue after returns?"

**For scope ambiguity:**
> "When you say 'top customers' — top by total revenue, by number of orders, by profit margin, or by recency of purchase?"

---

## Pre-Generation Checklist

Before writing any DAX measure, confirm:

- [ ] KPI term unambiguous (or confirmed with user)
- [ ] Comparison period confirmed (if growth/trend measure)
- [ ] Currency single or conversion handled
- [ ] Aggregation type appropriate (SUM / AVERAGE / DISTINCTCOUNT / COUNTROWS)
- [ ] Calendar type confirmed (fiscal vs calendar)
- [ ] Column names confirmed via MCP
- [ ] Filter context understood and commented in DAX
