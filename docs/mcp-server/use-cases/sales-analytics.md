---
title: "Sales Analytics with AI"
version: "1.0"
last_updated: "2025-11-15"
status: "Active"
---

# Sales Analytics with AI

## Business Scenario

Sales managers need to analyze performance, identify trends, and make data-driven decisions. Using Claude Desktop with MCP, they can get instant insights without building complex reports.

---

## Example Queries

### Query 1: Top Performing Products

**User Question**:
```
What are our top 5 products by revenue this month?
```

**Claude's Actions**:
1. Uses `analytics_query` with grouping by product
2. Filters by date range (this month)
3. Sorts by revenue

**Result**:
```
Top 5 Products (November 2025):

1. Office Chair Pro
   Revenue: $45,200
   Units Sold: 156
   Avg Price: $290

2. Standing Desk Electric
   Revenue: $38,900
   Units Sold: 89
   Avg Price: $437

3. Conference Table Large
   Revenue: $34,500
   Units Sold: 23
   Avg Price: $1,500

4. LED Desk Lamp
   Revenue: $28,700
   Units Sold: 358
   Avg Price: $80

5. Ergonomic Keyboard
   Revenue: $22,100
   Units Sold: 442
   Avg Price: $50

Total Revenue (Top 5): $169,400 (67% of total sales)
```

---

### Query 2: Sales by Salesperson

**User Question**:
```
Compare sales performance by salesperson this quarter
```

**Claude's Actions**:
1. Uses `analytics_query` grouped by salesperson
2. Calculates totals and averages
3. Provides comparative analysis

**Result**:
```
Q4 2025 Sales Performance:

1. Sarah Johnson
   Orders: 87
   Revenue: $245,600
   Avg Deal: $2,823
   Growth: +15% vs Q3

2. Mike Chen
   Orders: 76
   Revenue: $198,400
   Avg Deal: $2,611
   Growth: +8% vs Q3

3. Emily Rodriguez
   Orders: 65
   Revenue: $167,300
   Avg Deal: $2,574
   Growth: +22% vs Q3

4. David Kim
   Orders: 54
   Revenue: $145,900
   Avg Deal: $2,702
   Growth: +5% vs Q3

Team Total: 282 orders, $757,200 revenue
Team Average Deal Size: $2,686
```

---

### Query 3: Monthly Trends

**User Question**:
```
Show me sales trends for the past 6 months
```

**Claude's Actions**:
1. Uses `analytics_query` with date grouping
2. Calculates month-over-month growth
3. Identifies patterns

**Result**:
```
Sales Trends (May - October 2025):

May:      $198,400  (baseline)
June:     $215,600  (+8.7%)
July:     $234,200  (+8.6%)
August:   $228,900  (-2.3%) ⚠️
September: $256,700  (+12.1%)
October:  $287,300  (+11.9%)

Key Insights:
✅ Strong upward trend (+44.8% growth over 6 months)
✅ Consistent month-over-month growth except August
⚠️ August dip likely due to summer vacation season
📈 Average monthly growth: +7.5%
📊 Projected November: $320,000 (if trend continues)

Recommendation: Investigate August patterns to prevent
future seasonal dips. Consider promotions or campaigns
during slow months.
```

---

### Query 4: Customer Segmentation

**User Question**:
```
Segment our customers by total purchase amount and show me the distribution
```

**Claude's Actions**:
1. Uses `analytics_query` to group customers
2. Creates value segments
3. Calculates distribution

**Result**:
```
Customer Segmentation by Lifetime Value:

VIP (>$50,000):
- Count: 12 customers (2.4%)
- Revenue: $845,600 (38.2% of total)
- Avg: $70,467 per customer

High-Value ($20,000 - $50,000):
- Count: 45 customers (9.1%)
- Revenue: $1,125,400 (50.9% of total)
- Avg: $25,009 per customer

Medium-Value ($5,000 - $20,000):
- Count: 156 customers (31.4%)
- Revenue: $198,700 (9.0% of total)
- Avg: $1,274 per customer

Low-Value (<$5,000):
- Count: 284 customers (57.1%)
- Revenue: $42,300 (1.9% of total)
- Avg: $149 per customer

Insights:
💎 Top 11.5% of customers generate 89.1% of revenue
📈 Focus on nurturing high-value relationships
🎯 Opportunity: Move medium-value to high-value tier
```

---

## MCP Tools Used

| Tool | Purpose |
|------|---------|
| `analytics_query` | Aggregations, grouping, trends |
| `complex_search` | Filtered data queries |
| `search_records` | Historical data retrieval |

---

## Benefits

✅ **Instant Insights**: No waiting for reports
✅ **Ad-hoc Analysis**: Ask any question, anytime
✅ **Natural Language**: No SQL or complex queries
✅ **Actionable Recommendations**: AI-powered insights

---

## Best Practices

1. **Be Specific with Timeframes**: "this month", "Q4 2025", "last 6 months"
2. **Ask for Comparisons**: "compare vs last year", "trend over time"
3. **Request Insights**: "what do you recommend?", "what patterns do you see?"
4. **Follow Up**: Drill down into interesting findings

---

**Last Updated**: 2025-11-15
**Maintained By**: Zenoo RPC Development Team
