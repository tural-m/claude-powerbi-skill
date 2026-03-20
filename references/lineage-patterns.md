# Data Lineage & Dependency Patterns

Use this file during Block 9 (Data Lineage & Dependency Mapping) when tracing column origins, measure dependencies, or assessing impact of schema changes.

---

## Lineage Trace Format

When a user asks "where does [column/measure] come from?" — produce this output:

```
Lineage: Sales[NetRevenue]
──────────────────────────────────────────────
Source:
  SQL Server → dbo.SalesTransactions → [gross_amount], [discount_amount]

Power Query (Sales query):
  Step 1: Connect to SQL source
  Step 2: Filter Status = "Completed"           ← removes cancelled orders
  Step 3: AddColumn "NetRevenue" =
          [gross_amount] - [discount_amount]     ← transformation origin
  Step 4: RenameColumns → "NetRevenue"
  Step 5: Remove source columns

Model column:
  Sales[NetRevenue] — Decimal, summarize as Sum

Measures referencing this column:
  → [TotalRevenue]: SUM(Sales[NetRevenue])
  → [YoYRevenueGrowth]: uses [TotalRevenue]
  → [RevenueShareByRegion]: uses [TotalRevenue]

Visuals using these measures:
  → Page "Executive Summary": Card visual (TotalRevenue)
  → Page "Sales Analysis": Bar chart (RevenueShareByRegion)
  → Page "Trends": Line chart (YoYRevenueGrowth)
──────────────────────────────────────────────
```

---

## Impact Analysis Format

Before any schema change, produce this output:

```
Impact Analysis: Rename Sales[NetRevenue] → Sales[Revenue]
──────────────────────────────────────────────────────────
BREAKING (will error immediately):
  ❌ Measure [TotalRevenue] — references Sales[NetRevenue] directly
  ❌ Measure [YoYRevenueGrowth] — depends on [TotalRevenue]
  ❌ Measure [RevenueShareByRegion] — depends on [TotalRevenue]

DEGRADED (will return blank or wrong value):
  ⚠️  Power Query step "AddColumn NetRevenue" — source column renamed
  ⚠️  Any RLS rules filtering on this column

COSMETIC (display only, no calculation impact):
  ℹ️  Column name in visual tooltips
  ℹ️  Field list display name

SAFE TO CHANGE:
  ✅ Report page titles (text only)
  ✅ Unrelated tables

Recommended fix order:
  1. Update Power Query step name (cosmetic)
  2. Update Sales[NetRevenue] → Sales[Revenue] in model
  3. Update [TotalRevenue] measure formula
  4. [YoYRevenueGrowth] and [RevenueShareByRegion] update automatically
     (they reference [TotalRevenue], not the column directly)
──────────────────────────────────────────────────────────
```

---

## Measure Dependency Map Format

```
Measure Dependency Tree: [YoYRevenueGrowth]
──────────────────────────────────────────────
[YoYRevenueGrowth]
  └── depends on: [TotalRevenue]
        └── depends on: Sales[NetRevenue]  (column)
              └── derived from: Power Query AddColumn step
                    └── source: Sales[gross_amount] - Sales[discount_amount]

Time intelligence dependency:
  └── requires: Calendar[Date] table
        └── marked as Date Table: ✅ confirmed
        └── contiguous dates: ✅ confirmed
        └── relationship to Sales: Sales[OrderDate] → Calendar[Date] (active)
──────────────────────────────────────────────
```

---

## Common Lineage Patterns in Power BI

### Pattern 1: Calculated column vs measure — when each is appropriate

| Use case | Use calculated column | Use measure |
|----------|-----------------------|-------------|
| Row-level computation needed in slicer | ✅ | ❌ |
| Aggregation that changes with filter context | ❌ | ✅ |
| Text categorization (bucketing) | ✅ | ❌ |
| KPI dependent on user filter selection | ❌ | ✅ |
| Used as relationship key | ✅ | ❌ |

**Lineage implication:** Calculated columns are stored in the model and computed at refresh time. Measures are computed at query time. Changing a source column breaks both, but calculated columns may also break downstream measures.

---

### Pattern 2: Snowflake to star schema lineage risk

In a snowflake schema, dimension tables chain together:

```
Sales → Product → ProductSubcategory → ProductCategory
```

Measures filtering at category level must propagate through all joins.

**Risk:** If a relationship in the chain is inactive or many-to-many, filters may not propagate correctly.

**Lineage check:** When tracing a measure that filters by category, confirm every relationship in the chain is active and single-direction.

---

### Pattern 3: Shared dimension (role-playing dimension)

A date table used for multiple fact relationships:

```
Calendar[Date] ─── (active) ───→ Sales[OrderDate]
Calendar[Date] ─── (inactive) → Sales[ShipDate]
Calendar[Date] ─── (inactive) → Sales[DeliveryDate]
```

Measures using USERELATIONSHIP() activate the inactive relationships.

**Lineage flag:** If user asks about "delivery date analysis" — confirm USERELATIONSHIP is used in the measure, not the default relationship.

---

### Pattern 4: Calculation group interaction

If calculation groups exist, they intercept all measures by default.

**Lineage impact:** A measure [TotalRevenue] may behave differently depending on which calculation group item is selected.

**Detection:** Query MCP for calculation group tables. If found — document:
- Calculation group name
- Items (e.g. "Actual", "Budget", "Variance")
- Which measures are affected (typically all, unless excluded with ISSELECTEDMEASURE())

---

## Circular Dependency Detection

Circular dependencies in DAX cause model errors. Common accidental patterns:

```dax
-- ❌ Circular: TableA[ColA] references TableA[ColB], which references TableA[ColA]
TableA[ColA] = TableA[ColB] * 2
TableA[ColB] = TableA[ColA] + 10
```

**Detection approach:**
1. List all calculated columns
2. For each column, extract referenced columns
3. Build dependency graph
4. Flag any cycle (A → B → A)

**Flag message:**
> "Circular dependency detected: [ColA] → [ColB] → [ColA]. This will cause a model error. One of these columns must use a different source."

---

## Lineage Documentation Template

Auto-generate this for Block 10 documentation output:

```markdown
## Data Lineage Summary

### Fact Tables
| Table | Source | Key transformation | Row count |
|-------|--------|-------------------|-----------|
| Sales | SQL Server dbo.SalesTransactions | Filter Status=Completed, Add NetRevenue | 1.8M |

### Dimension Tables
| Table | Source | Notes |
|-------|--------|-------|
| Calendar | Calculated table (DAX) | Contiguous 2020–2030, marked as Date Table |
| Customer | CSV import | Refreshed weekly |
| Product | SQL Server dbo.Products | Joined with ProductCategory (snowflake — review) |

### Measures (dependencies)
| Measure | Depends on columns | Depends on measures | Used in visuals |
|---------|-------------------|---------------------|-----------------|
| TotalRevenue | Sales[NetRevenue] | — | Executive Summary (Card), Sales Analysis (Bar) |
| YoYRevenueGrowth | Sales[NetRevenue] | TotalRevenue | Trends (Line) |

### Known Lineage Risks
- Sales[NetRevenue] is computed in Power Query — column rename will break 3 measures
- Product table uses snowflake join — category filters require chain validation
```
