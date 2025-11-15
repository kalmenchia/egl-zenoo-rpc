---
title: "MCP Prompts Reference"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "End Users"
---

# MCP Prompts Reference

## Overview

MCP Prompts are pre-configured templates that guide AI assistants in performing common Odoo tasks. They provide structured workflows for complex operations.

**Available Prompts**: 3 main prompt types
- Data analysis templates
- Report generation workflows  
- Workflow optimization suggestions

---

## Available Prompts

### 1. analyze_data

Generate data analysis workflows for Odoo models.

**Usage**:
```
User: Analyze customer data
Claude: [Uses analyze_data prompt with model="res.partner"]
```

**Parameters**:
- `model` (default: `res.partner`) - Model to analyze
- `analysis_type` (default: `summary`) - Type: `summary`, `trends`, `comparison`

**Example Output**:
```
Analyzing res.partner data:

1. Total Records: 1,247
2. Distribution:
   - Companies: 342 (27%)
   - Individuals: 905 (73%)
3. Top Countries:
   - United States: 456
   - Canada: 123
   - Mexico: 89
4. Trends:
   - New customers this month: 47
   - Growth rate: +12%
```

### 2. generate_report_query

Create report generation queries.

**Usage**:
```
User: Generate a sales report
Claude: [Uses generate_report_query prompt]
```

**Example Output**:
```sql
-- Sales Report Query
SELECT 
    partner_id,
    COUNT(*) as order_count,
    SUM(amount_total) as total_revenue,
    AVG(amount_total) as avg_order_value
FROM sale_order
WHERE state = 'sale'
  AND date_order >= '2025-01-01'
GROUP BY partner_id
ORDER BY total_revenue DESC
LIMIT 20;
```

### 3. suggest_workflow

Analyze and suggest workflow improvements.

**Usage**:
```
User: How can I improve my sales process?
Claude: [Uses suggest_workflow prompt]
```

**Example Output**:
```
Workflow Analysis:

Current State:
- Manual order entry
- Paper-based approvals
- Email confirmations

Suggested Improvements:
1. Automate order import from website
2. Implement digital approval workflow
3. Set up automatic email notifications
4. Add real-time inventory checks

Expected Benefits:
- 50% reduction in order processing time
- 90% fewer data entry errors
- Better customer experience
```

---

## Using Prompts in Claude Desktop

Prompts work automatically when you ask relevant questions:

```
User: "Analyze my customer data"
→ Claude uses analyze_data prompt

User: "How can I improve efficiency?"
→ Claude uses suggest_workflow prompt

User: "Create a sales report"
→ Claude uses generate_report_query prompt
```

---

**Last Updated**: 2025-11-15
**Maintained By**: Zenoo RPC Development Team
