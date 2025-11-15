---
title: "MCP Tools Reference"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "Developers, End Users"
---

# MCP Tools Reference

## Overview

This document provides a complete reference for all MCP tools available in the Zenoo RPC MCP Server. Tools are functions that AI assistants can execute to interact with your Odoo system.

**Available Tools**: 8
- 5 Basic CRUD operations
- 3 Advanced operations (complex search, batch, analytics)

---

## Table of Contents

### Basic Tools
1. [search_records](#1-search_records)
2. [get_record](#2-get_record)
3. [create_record](#3-create_record)
4. [update_record](#4-update_record)
5. [delete_record](#5-delete_record)

### Advanced Tools
6. [complex_search](#6-complex_search)
7. [batch_operation](#7-batch_operation)
8. [analytics_query](#8-analytics_query)

---

## Tool Categories

| Category | Tools | Use Case |
|----------|-------|----------|
| **Read** | `search_records`, `get_record`, `complex_search` | Querying data |
| **Write** | `create_record`, `update_record`, `delete_record` | Modifying data |
| **Bulk** | `batch_operation` | High-performance operations |
| **Analytics** | `analytics_query` | Aggregation and reporting |

---

## Basic Tools

### 1. search_records

Search for records in any Odoo model using domain filters.

#### Signature

```python
async def search_records(
    model: str,
    domain: List = None,
    fields: List[str] = None,
    limit: int = 100,
    offset: int = 0,
    order: str = None
) -> Dict[str, Any]
```

#### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `model` | string | ✅ Yes | - | Odoo model name (e.g., `res.partner`, `sale.order`) |
| `domain` | array | ❌ No | `[]` | Odoo domain filter (e.g., `[['name', 'ilike', 'John']]`) |
| `fields` | array | ❌ No | all | List of fields to retrieve |
| `limit` | integer | ❌ No | 100 | Maximum number of records |
| `offset` | integer | ❌ No | 0 | Number of records to skip |
| `order` | string | ❌ No | `id DESC` | Sort order (e.g., `name ASC`, `create_date DESC`) |

#### Returns

```json
{
  "records": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      ...
    }
  ],
  "count": 47,
  "model": "res.partner",
  "has_more": true,
  "domain": [["is_company", "=", false]],
  "fields": "all"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `records` | array | List of matching records |
| `count` | integer | Number of records returned |
| `model` | string | Model name that was searched |
| `has_more` | boolean | True if more records exist beyond the limit |
| `domain` | array | The domain filter that was applied |
| `fields` | string/array | Fields that were retrieved |

#### Examples

**Example 1: Simple Search**

Natural language:
```
Find all customers in the system
```

Tool call:
```json
{
  "tool": "search_records",
  "arguments": {
    "model": "res.partner",
    "domain": [["customer_rank", ">", 0]],
    "limit": 100
  }
}
```

**Example 2: Search with Field Selection**

Natural language:
```
Show me names and emails of all companies
```

Tool call:
```json
{
  "tool": "search_records",
  "arguments": {
    "model": "res.partner",
    "domain": [["is_company", "=", true]],
    "fields": ["id", "name", "email", "phone"],
    "limit": 50
  }
}
```

**Example 3: Complex Domain**

Natural language:
```
Find customers in California or Texas with credit limit over $10,000
```

Tool call:
```json
{
  "tool": "search_records",
  "arguments": {
    "model": "res.partner",
    "domain": [
      ["customer_rank", ">", 0],
      "|",
      ["state", "=", "CA"],
      ["state", "=", "TX"],
      ["credit_limit", ">", 10000]
    ],
    "order": "credit_limit DESC",
    "limit": 20
  }
}
```

**Example 4: Pagination**

Natural language:
```
Show me the next 20 sales orders (starting from position 40)
```

Tool call:
```json
{
  "tool": "search_records",
  "arguments": {
    "model": "sale.order",
    "offset": 40,
    "limit": 20,
    "order": "date_order DESC"
  }
}
```

#### Domain Filter Syntax

Odoo domains use Polish notation:

| Operator | Syntax | Example |
|----------|--------|---------|
| Equal | `['field', '=', value]` | `['name', '=', 'John']` |
| Not Equal | `['field', '!=', value]` | `['state', '!=', 'draft']` |
| Greater Than | `['field', '>', value]` | `['amount', '>', 100]` |
| Less Than | `['field', '<', value]` | `['qty', '<', 10]` |
| Like | `['field', 'like', 'pattern']` | `['name', 'like', '%Corp%']` |
| iLike | `['field', 'ilike', 'pattern']` | `['email', 'ilike', '%@gmail.com']` |
| In | `['field', 'in', [list]]` | `['id', 'in', [1, 2, 3]]` |
| AND | No operator (default) | `[['a', '=', 1], ['b', '=', 2]]` |
| OR | `'\|'` prefix | `['\|', ['a', '=', 1], ['b', '=', 2]]` |
| NOT | `'!'` prefix | `['!', ['state', '=', 'draft']]` |

#### File Reference

**Implementation**: `src/zenoo_rpc/mcp_server/server.py:234-263`

```python
@self.mcp_server.tool()
async def search_records(
    model: str,
    domain: List = None,
    fields: List[str] = None,
    limit: int = 100,
    offset: int = 0,
    order: str = None
) -> Dict[str, Any]:
    """Search for records in an Odoo model."""
    return await self._execute_tool("search_records", {
        "model": model,
        "domain": domain or [],
        "fields": fields,
        "limit": limit,
        "offset": offset,
        "order": order
    })
```

**Handler**: `src/zenoo_rpc/mcp_server/server.py:499-530`

---

### 2. get_record

Get a specific record by ID with all or selected fields.

#### Signature

```python
async def get_record(
    model: str,
    record_id: int,
    fields: List[str] = None
) -> Dict[str, Any]
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | ✅ Yes | Odoo model name |
| `record_id` | integer | ✅ Yes | Record ID to retrieve |
| `fields` | array | ❌ No | List of fields to retrieve (default: all) |

#### Returns

```json
{
  "record": {
    "id": 15,
    "name": "Azure Interior",
    "email": "azure@example.com",
    "phone": "+1-555-0123",
    "is_company": true,
    "country_id": [233, "United States"]
  },
  "model": "res.partner",
  "id": 15
}
```

#### Examples

**Example 1: Get Full Record**

Natural language:
```
Show me all details for partner ID 15
```

Tool call:
```json
{
  "tool": "get_record",
  "arguments": {
    "model": "res.partner",
    "record_id": 15
  }
}
```

**Example 2: Get Specific Fields**

Natural language:
```
Get the name and email for sales order 42
```

Tool call:
```json
{
  "tool": "get_record",
  "arguments": {
    "model": "sale.order",
    "record_id": 42,
    "fields": ["id", "name", "partner_id", "amount_total", "state"]
  }
}
```

**Example 3: Check Record Exists**

Natural language:
```
Does product ID 999 exist?
```

Tool call:
```json
{
  "tool": "get_record",
  "arguments": {
    "model": "product.product",
    "record_id": 999,
    "fields": ["id", "name"]
  }
}
```

#### Error Handling

If record doesn't exist:
```json
{
  "error": "Record 999 not found in model product.product"
}
```

#### File Reference

**Implementation**: `src/zenoo_rpc/mcp_server/server.py:265-285`
**Handler**: `src/zenoo_rpc/mcp_server/server.py:561-601`

---

### 3. create_record

Create a new record in any Odoo model.

#### Signature

```python
async def create_record(
    model: str,
    values: Dict[str, Any]
) -> Dict[str, Any]
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | ✅ Yes | Odoo model name |
| `values` | object | ✅ Yes | Field values for the new record |

#### Returns

```json
{
  "record": {
    "id": 156,
    "name": "New Customer",
    "email": "new@example.com",
    "create_date": "2025-11-15 10:30:00"
  },
  "model": "res.partner",
  "created": true,
  "id": 156
}
```

#### Examples

**Example 1: Create Customer**

Natural language:
```
Create a new customer:
- Name: Acme Corporation
- Email: contact@acme.com
- Phone: +1-555-0199
```

Tool call:
```json
{
  "tool": "create_record",
  "arguments": {
    "model": "res.partner",
    "values": {
      "name": "Acme Corporation",
      "email": "contact@acme.com",
      "phone": "+1-555-0199",
      "is_company": true,
      "customer_rank": 1
    }
  }
}
```

**Example 2: Create Sales Order**

Natural language:
```
Create a sales order for partner 15 with these lines:
- Product 10, quantity 5
- Product 20, quantity 3
```

Tool call:
```json
{
  "tool": "create_record",
  "arguments": {
    "model": "sale.order",
    "values": {
      "partner_id": 15,
      "order_line": [
        [0, 0, {
          "product_id": 10,
          "product_uom_qty": 5
        }],
        [0, 0, {
          "product_id": 20,
          "product_uom_qty": 3
        }]
      ]
    }
  }
}
```

#### Relationship Fields

Odoo uses special syntax for relationship fields:

| Relationship | Syntax | Example |
|--------------|--------|---------|
| **Many2one** | `field_id: integer` | `"partner_id": 15` |
| **One2many/Many2many** (create) | `[[0, 0, {values}]]` | `[[0, 0, {"name": "Line 1"}]]` |
| **One2many/Many2many** (link) | `[[4, id]]` | `[[4, 5], [4, 10]]` (link IDs 5 and 10) |
| **One2many/Many2many** (replace) | `[[6, 0, [ids]]]` | `[[6, 0, [1, 2, 3]]]` |

#### Validation

Values are validated using Pydantic schemas:
- Required fields must be provided
- Field types must match
- Relationships must reference valid records

#### File Reference

**Implementation**: `src/zenoo_rpc/mcp_server/server.py:287-304`
**Handler**: `src/zenoo_rpc/mcp_server/server.py:603-639`

---

### 4. update_record

Update an existing record.

#### Signature

```python
async def update_record(
    model: str,
    record_id: int,
    values: Dict[str, Any]
) -> Dict[str, Any]
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | ✅ Yes | Odoo model name |
| `record_id` | integer | ✅ Yes | ID of record to update |
| `values` | object | ✅ Yes | Fields to update with new values |

#### Returns

```json
{
  "record": {
    "id": 50,
    "name": "Updated Name",
    "email": "updated@example.com",
    "write_date": "2025-11-15 14:30:00"
  },
  "model": "res.partner",
  "updated": true,
  "transaction_id": "tx_abc123"
}
```

#### Examples

**Example 1: Update Customer**

Natural language:
```
Update partner 50 to set email to newemail@example.com
```

Tool call:
```json
{
  "tool": "update_record",
  "arguments": {
    "model": "res.partner",
    "record_id": 50,
    "values": {
      "email": "newemail@example.com"
    }
  }
}
```

**Example 2: Update Multiple Fields**

Natural language:
```
For sales order 42, update the state to 'confirmed' and add a note
```

Tool call:
```json
{
  "tool": "update_record",
  "arguments": {
    "model": "sale.order",
    "record_id": 42,
    "values": {
      "state": "sale",
      "note": "Confirmed by MCP on 2025-11-15"
    }
  }
}
```

#### Transaction Support

Updates are executed within a transaction:
- ✅ Automatic commit on success
- ✅ Automatic rollback on error
- ✅ Transaction ID returned for audit

#### File Reference

**Implementation**: `src/zenoo_rpc/mcp_server/server.py:306-326`
**Handler**: `src/zenoo_rpc/mcp_server/server.py:641-685`

---

### 5. delete_record

Delete a record from Odoo.

#### Signature

```python
async def delete_record(
    model: str,
    record_id: int
) -> Dict[str, Any]
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | ✅ Yes | Odoo model name |
| `record_id` | integer | ✅ Yes | ID of record to delete |

#### Returns

```json
{
  "id": 99,
  "model": "res.partner",
  "deleted": true,
  "transaction_id": "tx_xyz789"
}
```

#### Examples

**Example 1: Delete Record**

Natural language:
```
Delete partner ID 99
```

Tool call:
```json
{
  "tool": "delete_record",
  "arguments": {
    "model": "res.partner",
    "record_id": 99
  }
}
```

#### Important Notes

⚠️ **WARNING**: Deletion is permanent and cannot be undone!

**Safety Considerations**:
- Odoo may prevent deletion if record has dependencies
- Some models have `active` field instead - consider using `update_record` to set `active: false`
- Transactions provide rollback on error but not after success

**Best Practice**:
Use archive instead of delete when possible:
```json
{
  "tool": "update_record",
  "arguments": {
    "model": "res.partner",
    "record_id": 99,
    "values": {"active": false}
  }
}
```

#### File Reference

**Implementation**: `src/zenoo_rpc/mcp_server/server.py:328-345`
**Handler**: `src/zenoo_rpc/mcp_server/server.py:687-725`

---

## Advanced Tools

### 6. complex_search

Advanced search using QueryBuilder with complex filters and relationships.

#### Signature

```python
async def complex_search(
    model: str,
    filters: Dict[str, Any],
    order_by: str = None,
    limit: int = 100,
    include_relationships: bool = False
) -> Dict[str, Any]
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | ✅ Yes | Odoo model name |
| `filters` | object | ✅ Yes | Complex filters using Django-like syntax |
| `order_by` | string | ❌ No | Sort field (prefix `-` for descending) |
| `limit` | integer | ❌ No | Maximum records (default: 100) |
| `include_relationships` | boolean | ❌ No | Include related record data |

#### Filter Syntax

Django-like lookup syntax:

| Lookup | Example | Odoo Equivalent |
|--------|---------|-----------------|
| `field` | `{"name": "John"}` | `[('name', '=', 'John')]` |
| `field__ilike` | `{"name__ilike": "john"}` | `[('name', 'ilike', 'john')]` |
| `field__gt` | `{"amount__gt": 1000}` | `[('amount', '>', 1000)]` |
| `field__gte` | `{"amount__gte": 1000}` | `[('amount', '>=', 1000)]` |
| `field__lt` | `{"qty__lt": 10}` | `[('qty', '<', 10)]` |
| `field__lte` | `{"qty__lte": 10}` | `[('qty', '<=', 10)]` |
| `field__in` | `{"id__in": [1,2,3]}` | `[('id', 'in', [1,2,3])]` |

#### Returns

```json
{
  "records": [
    {
      "id": 10,
      "name": "Azure Interior",
      "amount_total": 15000.00,
      "_relationships": {
        "partner_id": {
          "id": 15,
          "name": "Customer Name"
        }
      }
    }
  ],
  "count": 23,
  "model": "sale.order",
  "filters_applied": {"amount_total__gt": 10000},
  "order_by": "-amount_total",
  "includes_relationships": true
}
```

#### Examples

**Example 1: Complex Filters**

Natural language:
```
Find sales orders with amount over $10,000 from this year,
ordered by amount descending
```

Tool call:
```json
{
  "tool": "complex_search",
  "arguments": {
    "model": "sale.order",
    "filters": {
      "amount_total__gt": 10000,
      "date_order__gte": "2025-01-01"
    },
    "order_by": "-amount_total",
    "limit": 50
  }
}
```

**Example 2: With Relationships**

Natural language:
```
Show me all invoices with customer details included
```

Tool call:
```json
{
  "tool": "complex_search",
  "arguments": {
    "model": "account.move",
    "filters": {
      "move_type": "out_invoice",
      "state": "posted"
    },
    "include_relationships": true,
    "limit": 20
  }
}
```

**Example 3: Multiple Conditions**

Natural language:
```
Find products that are:
- In 'Electronics' category
- Price between $100 and $500
- In stock (qty > 0)
- Name contains 'laptop'
```

Tool call:
```json
{
  "tool": "complex_search",
  "arguments": {
    "model": "product.product",
    "filters": {
      "categ_id__name__ilike": "electronics",
      "list_price__gte": 100,
      "list_price__lte": 500,
      "qty_available__gt": 0,
      "name__ilike": "laptop"
    },
    "order_by": "list_price",
    "limit": 30
  }
}
```

#### File Reference

**Implementation**: `src/zenoo_rpc/mcp_server/server.py:348-374`
**Handler**: `src/zenoo_rpc/mcp_server/server.py:831-905`

---

### 7. batch_operation

Perform bulk create, update, or delete operations efficiently.

#### Signature

```python
async def batch_operation(
    operation: str,
    model: str,
    records: List[Dict[str, Any]]
) -> Dict[str, Any]
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | string | ✅ Yes | Operation type: `create`, `update`, or `delete` |
| `model` | string | ✅ Yes | Odoo model name |
| `records` | array | ✅ Yes | List of record data |

#### Returns

```json
{
  "operation": "create",
  "model": "res.partner",
  "processed_count": 50,
  "results": [
    {"id": 200, "name": "Customer 1", ...},
    {"id": 201, "name": "Customer 2", ...}
  ],
  "transaction_id": "tx_batch_123"
}
```

#### Examples

**Example 1: Batch Create**

Natural language:
```
Create these 3 customers in one operation:
1. Alpha Corp - alpha@corp.com
2. Beta Inc - beta@inc.com
3. Gamma LLC - gamma@llc.com
```

Tool call:
```json
{
  "tool": "batch_operation",
  "arguments": {
    "operation": "create",
    "model": "res.partner",
    "records": [
      {"name": "Alpha Corp", "email": "alpha@corp.com", "is_company": true},
      {"name": "Beta Inc", "email": "beta@inc.com", "is_company": true},
      {"name": "Gamma LLC", "email": "gamma@llc.com", "is_company": true}
    ]
  }
}
```

**Example 2: Batch Update**

Natural language:
```
Update all these sales orders to 'confirmed' status:
- Order 10, 15, 20, 25, 30
```

Tool call:
```json
{
  "tool": "batch_operation",
  "arguments": {
    "operation": "update",
    "model": "sale.order",
    "records": [
      {"id": 10, "state": "sale"},
      {"id": 15, "state": "sale"},
      {"id": 20, "state": "sale"},
      {"id": 25, "state": "sale"},
      {"id": 30, "state": "sale"}
    ]
  }
}
```

**Example 3: Batch Delete**

Natural language:
```
Delete these test partners: IDs 500, 501, 502, 503
```

Tool call:
```json
{
  "tool": "batch_operation",
  "arguments": {
    "operation": "delete",
    "model": "res.partner",
    "records": [
      {"id": 500},
      {"id": 501},
      {"id": 502},
      {"id": 503}
    ]
  }
}
```

#### Performance

Batch operations are optimized:
- All records processed in single transaction
- Reduced RPC calls compared to individual operations
- 10-100x faster for large datasets

**Benchmarks**:
| Records | Individual Ops | Batch Op | Speedup |
|---------|---------------|----------|---------|
| 10 | 5s | 0.5s | 10x |
| 100 | 50s | 2s | 25x |
| 1000 | 500s | 15s | 33x |

#### File Reference

**Implementation**: `src/zenoo_rpc/mcp_server/server.py:376-396`
**Handler**: `src/zenoo_rpc/mcp_server/server.py:907-958`

---

### 8. analytics_query

Perform aggregation queries with grouping and date ranges.

#### Signature

```python
async def analytics_query(
    model: str,
    group_by: List[str],
    aggregates: Dict[str, str],
    filters: Dict[str, Any] = None,
    date_range: Dict[str, str] = None
) -> Dict[str, Any]
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | ✅ Yes | Odoo model name |
| `group_by` | array | ✅ Yes | Fields to group by |
| `aggregates` | object | ✅ Yes | Aggregation functions: `sum`, `count`, `avg`, `max`, `min` |
| `filters` | object | ❌ No | Optional filters |
| `date_range` | object | ❌ No | Date range: `{field, start, end}` |

#### Returns

```json
{
  "model": "sale.order",
  "group_by": ["partner_id", "state"],
  "aggregates": {"total": "sum", "count": "count"},
  "results": [
    {
      "partner_id": 15,
      "state": "sale",
      "total": 25000.00,
      "count": 12
    },
    {
      "partner_id": 20,
      "state": "sale",
      "total": 18000.00,
      "count": 8
    }
  ],
  "count": 25
}
```

#### Examples

**Example 1: Sales by Month**

Natural language:
```
Show me total sales amount grouped by month for this year
```

Tool call:
```json
{
  "tool": "analytics_query",
  "arguments": {
    "model": "sale.order",
    "group_by": ["date_order:month"],
    "aggregates": {
      "total_revenue": "sum",
      "order_count": "count"
    },
    "date_range": {
      "field": "date_order",
      "start": "2025-01-01",
      "end": "2025-12-31"
    }
  }
}
```

**Example 2: Product Performance**

Natural language:
```
What are my top products by revenue?
```

Tool call:
```json
{
  "tool": "analytics_query",
  "arguments": {
    "model": "sale.order.line",
    "group_by": ["product_id"],
    "aggregates": {
      "revenue": "sum",
      "quantity_sold": "sum",
      "avg_price": "avg"
    },
    "filters": {
      "order_id.state": "sale"
    }
  }
}
```

**Example 3: Customer Analysis**

Natural language:
```
Analyze customers by country: count, total sales, average order value
```

Tool call:
```json
{
  "tool": "analytics_query",
  "arguments": {
    "model": "sale.order",
    "group_by": ["partner_id.country_id"],
    "aggregates": {
      "total_sales": "sum",
      "order_count": "count",
      "avg_order_value": "avg"
    },
    "filters": {
      "state": "sale"
    }
  }
}
```

#### Supported Aggregations

| Function | Description | Example |
|----------|-------------|---------|
| `sum` | Sum of values | Total revenue |
| `count` | Count of records | Number of orders |
| `avg` | Average value | Average order amount |
| `max` | Maximum value | Highest sale amount |
| `min` | Minimum value | Lowest sale amount |

#### File Reference

**Implementation**: `src/zenoo_rpc/mcp_server/server.py:398-424`
**Handler**: `src/zenoo_rpc/mcp_server/server.py:960-1024`

---

## Tool Usage Patterns

### Pattern 1: Search → Get Details

```
1. search_records to find matching records
2. get_record to get full details of specific record
```

Example:
```
User: "Find customer 'Acme Corp' and show me all their details"

Step 1: search_records(model="res.partner", domain=[["name", "ilike", "Acme Corp"]])
        → Returns: [{id: 50, name: "Acme Corp", ...}]

Step 2: get_record(model="res.partner", record_id=50)
        → Returns full record with all fields
```

### Pattern 2: Create → Update

```
1. create_record to create new record
2. update_record to modify additional fields
```

### Pattern 3: Analytics → Drill Down

```
1. analytics_query to get aggregated overview
2. complex_search to drill down into specific segment
```

Example:
```
User: "Show me sales by state, then details for California"

Step 1: analytics_query(group_by=["partner_id.state"], aggregates={"total": "sum"})
        → Returns summary by state

Step 2: complex_search(filters={"partner_id.state": "CA"})
        → Returns detailed California orders
```

---

## Related Documentation

- **[Claude Desktop Integration](./04-claude-desktop-integration.md)** - Using tools via Claude
- **[Resources Reference](./06-resources-reference.md)** - Data resources
- **[Security Guide](./08-security-authentication.md)** - Tool permissions
- **[Python Library Query Guide](../user-guide/queries.md)** - QueryBuilder details

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team
