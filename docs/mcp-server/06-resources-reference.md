---
title: "MCP Resources Reference"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "Developers, End Users"
---

# MCP Resources Reference

## Overview

MCP Resources provide read-only access to Odoo data through URI-based addressing. Unlike tools (which are actions), resources allow AI assistants to browse and discover Odoo data structures.

**Available Resources**: 3 main resource types
- Model listings
- Model schema information
- Record data

---

## Table of Contents

1. [What are MCP Resources?](#what-are-mcp-resources)
2. [Resource URI Scheme](#resource-uri-scheme)
3. [Available Resources](#available-resources)
4. [Usage Examples](#usage-examples)
5. [Best Practices](#best-practices)

---

## What are MCP Resources?

### Resources vs Tools

| Aspect | Resources | Tools |
|--------|-----------|-------|
| **Purpose** | Data access (read-only) | Actions (read/write) |
| **Addressing** | URI-based | Function calls |
| **Caching** | Highly cacheable | Not cached |
| **Use Case** | Discovery, browsing | Operations, modifications |

### Resource Characteristics

✅ **Read-Only**: Resources never modify data
✅ **Discoverable**: Clients can browse available resources
✅ **Cacheable**: Results can be cached aggressively
✅ **Structured**: Consistent JSON format

---

## Resource URI Scheme

All resources use the `odoo://` URI scheme:

```
odoo://<resource-type>/<parameters>
```

**Syntax**:
- Protocol: `odoo://`
- Resource type: `models`, `model`, `record`, `search`
- Parameters: Depends on resource type

---

## Available Resources

### 1. List Models (`odoo://models`)

Get a list of all available Odoo models.

#### URI

```
odoo://models
```

#### Response Format

```json
{
  "resource": "list_models",
  "count": 847,
  "models": [
    {
      "model": "res.partner",
      "name": "Contact",
      "info": "Partners (customers, suppliers, addresses)"
    },
    {
      "model": "sale.order",
      "name": "Sales Order",
      "info": "Sales orders management"
    },
    {
      "model": "product.product",
      "name": "Product",
      "info": "Product variants"
    }
    // ... more models (limited to first 50)
  ]
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `resource` | string | Resource type identifier |
| `count` | integer | Total number of models |
| `models` | array | List of model information (max 50) |
| `models[].model` | string | Technical model name |
| `models[].name` | string | Human-readable name |
| `models[].info` | string | Model description |

#### Use Cases

**1. Discovery**
```
User: "What models are available in Odoo?"
Claude: [Fetches odoo://models]
        "Your Odoo has 847 models. Here are the main ones:
        - res.partner (Contacts)
        - sale.order (Sales Orders)
        - product.product (Products)
        ..."
```

**2. Model Search**
```
User: "Is there a model for invoices?"
Claude: [Fetches odoo://models]
        "Yes, there are several invoice-related models:
        - account.move (Invoices)
        - account.move.line (Invoice Lines)
        ..."
```

#### Claude Desktop Example

```
User: What Odoo models are available for customer management?

Claude: [Accesses resource odoo://models]
I found several customer management models in your Odoo:

1. **res.partner** - Main contact/customer model
2. **res.partner.category** - Customer categorization
3. **res.partner.bank** - Customer bank accounts
4. **res.partner.title** - Customer titles (Mr., Mrs., etc.)

Would you like details about any of these models?
```

---

### 2. Model Information (`odoo://model/{model_name}`)

Get detailed information about a specific Odoo model, including fields and schema.

#### URI

```
odoo://model/{model_name}
```

**Parameters**:
- `{model_name}`: Technical name of the model (e.g., `res.partner`, `sale.order`)

#### Examples

```
odoo://model/res.partner
odoo://model/sale.order
odoo://model/product.product
```

#### Response Format

```json
{
  "resource": "get_model_info",
  "model_name": "res.partner",
  "model_info": {
    "model": "res.partner",
    "name": "Contact",
    "info": "Partners (customers, suppliers, addresses)"
  },
  "fields": [
    {
      "name": "name",
      "field_description": "Name",
      "ttype": "char",
      "required": true
    },
    {
      "name": "email",
      "field_description": "Email",
      "ttype": "char",
      "required": false
    },
    {
      "name": "phone",
      "field_description": "Phone",
      "ttype": "char",
      "required": false
    },
    {
      "name": "is_company",
      "field_description": "Is a Company",
      "ttype": "boolean",
      "required": false
    },
    {
      "name": "country_id",
      "field_description": "Country",
      "ttype": "many2one",
      "required": false
    }
    // ... more fields (limited to first 20)
  ]
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `resource` | string | Resource type identifier |
| `model_name` | string | Model technical name |
| `model_info` | object | Model metadata |
| `fields` | array | List of model fields (max 20) |
| `fields[].name` | string | Field technical name |
| `fields[].field_description` | string | Field label |
| `fields[].ttype` | string | Field type (char, integer, boolean, many2one, etc.) |
| `fields[].required` | boolean | Whether field is required |

#### Field Types

| Type | Description | Example |
|------|-------------|---------|
| `char` | Text field | Name, email |
| `text` | Long text | Notes, description |
| `integer` | Whole number | Quantity, count |
| `float` | Decimal number | Price, amount |
| `boolean` | True/False | is_company, active |
| `date` | Date only | Birth date |
| `datetime` | Date and time | Created date |
| `selection` | Dropdown | State, type |
| `many2one` | Link to one record | Country, category |
| `one2many` | Link to many records | Order lines |
| `many2many` | Multiple links | Tags, categories |

#### Use Cases

**1. Understanding Model Structure**
```
User: "What fields does the customer model have?"
Claude: [Fetches odoo://model/res.partner]
        "The res.partner model has these main fields:
        - name (text, required)
        - email (text)
        - phone (text)
        - is_company (boolean)
        - country_id (link to country)
        ..."
```

**2. Building Queries**
```
User: "How do I filter customers by country?"
Claude: [Fetches odoo://model/res.partner]
        "The customer model has a 'country_id' field (many2one).
        You can filter using:
        domain=[['country_id', '=', country_id]]"
```

#### Claude Desktop Example

```
User: What fields are available in the sales order model?

Claude: [Accesses resource odoo://model/sale.order]
The sale.order model has these key fields:

**Basic Info**:
- name (text, required) - Order reference
- partner_id (many2one, required) - Customer
- date_order (datetime) - Order date
- state (selection) - Status

**Amounts**:
- amount_total (float) - Total amount
- amount_tax (float) - Tax amount
- amount_untaxed (float) - Untaxed amount

**Lines**:
- order_line (one2many) - Order lines

Would you like to see more fields or query sales orders?
```

---

### 3. Record Data (`odoo://record/{model_name}/{record_id}`)

Get detailed data for a specific record.

#### URI

```
odoo://record/{model_name}/{record_id}
```

**Parameters**:
- `{model_name}`: Technical model name
- `{record_id}`: Record ID (integer)

#### Examples

```
odoo://record/res.partner/15
odoo://record/sale.order/42
odoo://record/product.product/100
```

#### Response Format

```json
{
  "resource": "get_record_resource",
  "model_name": "res.partner",
  "record_id": "15",
  "record": {
    "id": 15,
    "name": "Azure Interior",
    "email": "azure@example.com",
    "phone": "+1-555-0123",
    "is_company": true,
    "street": "123 Main St",
    "city": "Los Angeles",
    "state_id": [5, "California"],
    "country_id": [233, "United States"],
    "customer_rank": 1,
    "active": true,
    "create_date": "2024-01-15 10:30:00",
    "write_date": "2025-11-10 14:20:00"
  }
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `resource` | string | Resource type identifier |
| `model_name` | string | Model technical name |
| `record_id` | string | Record ID |
| `record` | object | Full record data with all fields |

#### Use Cases

**1. Record Details**
```
User: "Show me details for partner ID 15"
Claude: [Fetches odoo://record/res.partner/15]
        "Partner #15: Azure Interior
        - Email: azure@example.com
        - Phone: +1-555-0123
        - Location: Los Angeles, California, United States
        - Type: Company
        - Customer since: 2024-01-15"
```

**2. Related Data Exploration**
```
User: "What's in sales order 42?"
Claude: [Fetches odoo://record/sale.order/42]
        "Sales Order SO00042:
        - Customer: Azure Interior (#15)
        - Date: 2025-11-01
        - Total: $5,250.00
        - Status: Confirmed
        - Lines: 3 items"
```

#### Claude Desktop Example

```
User: Get full details for customer ID 15

Claude: [Accesses resource odoo://record/res.partner/15]
Here are the complete details for customer #15:

**Azure Interior**
- ID: 15
- Type: Company
- Email: azure@example.com
- Phone: +1-555-0123

**Address**:
123 Main St
Los Angeles, California
United States

**Status**:
- Active: Yes
- Customer Rank: 1
- Created: 2024-01-15
- Last Updated: 2025-11-10

Would you like to see their sales orders or invoices?
```

---

## Usage Examples

### Example 1: Discovering Available Data

**Conversation Flow**:
```
User: What data is available in my Odoo system?

Step 1: Claude fetches odoo://models
Result: List of all models

Step 2: Claude summarizes by category
Response: "Your Odoo has data for:
- Sales (sale.order, sale.order.line)
- Customers (res.partner, res.partner.category)
- Products (product.product, product.template)
- Inventory (stock.picking, stock.move)
..."
```

### Example 2: Understanding Data Structure

**Conversation Flow**:
```
User: How is product data structured?

Step 1: Claude fetches odoo://model/product.product
Result: Product model fields

Step 2: Claude explains structure
Response: "Products have:
- Basic info (name, description, type)
- Pricing (list_price, standard_price)
- Stock (qty_available, virtual_available)
- Categorization (categ_id, tag_ids)
- Images (image_1920)
..."
```

### Example 3: Exploring Specific Records

**Conversation Flow**:
```
User: Show me what's in sales order 100

Step 1: Claude searches for order 100 using tools
Result: Confirms order exists

Step 2: Claude fetches odoo://record/sale.order/100
Result: Complete order details

Step 3: Claude formats for user
Response: "Sales Order SO00100:
[formatted order details]
Would you like to see the order lines?"
```

---

## Resource Patterns

### Pattern 1: Model → Fields → Records

```
1. odoo://models
   → Discover available models

2. odoo://model/res.partner
   → Understand structure

3. odoo://record/res.partner/15
   → Get specific data
```

### Pattern 2: Search → Detail

```
1. Use search_records tool to find records

2. odoo://record/{model}/{id}
   → Get full details for found records
```

### Pattern 3: Schema Discovery

```
1. odoo://models
   → List models related to topic

2. odoo://model/{model_name}
   → Understand each model's structure

3. Plan queries based on discovered schema
```

---

## Caching Behavior

### Resource Caching

Resources are designed to be cached:

| Resource | Cache Duration | Reason |
|----------|---------------|---------|
| `odoo://models` | Long (hours) | Model list rarely changes |
| `odoo://model/{name}` | Long (hours) | Schema rarely changes |
| `odoo://record/{model}/{id}` | Short (minutes) | Data changes frequently |

### Cache Control

Resources include cache hints:
```json
{
  "resource": "list_models",
  "cache_control": {
    "max_age": 3600,
    "stale_while_revalidate": 300
  },
  "data": {...}
}
```

---

## Best Practices

### 1. Use Resources for Discovery

✅ **DO**:
```
User: "What customer data do we have?"

Claude: [Uses odoo://models then odoo://model/res.partner]
Better understanding of data structure before querying
```

❌ **DON'T**:
```
User: "What customer data do we have?"

Claude: [Immediately uses search_records blindly]
No understanding of available fields
```

### 2. Combine Resources with Tools

✅ **DO**:
```
Step 1: odoo://model/res.partner (understand structure)
Step 2: search_records with appropriate fields
Step 3: odoo://record/{id} for details if needed
```

### 3. Respect Caching

Resources can be cached aggressively since they're read-only. This improves performance.

---

## Resource Limitations

### Current Limitations

1. **Result Limits**:
   - `odoo://models`: Returns max 50 models
   - `odoo://model/{name}`: Returns max 20 fields
   - Use tools for complete data access

2. **Read-Only**:
   - Resources cannot modify data
   - Use tools for create/update/delete

3. **No Filtering**:
   - Resources return standard views
   - Use tools with domains for filtered results

---

## Related Documentation

- **[Tools Reference](./05-tools-reference.md)** - MCP tools for data operations
- **[Prompts Reference](./07-prompts-reference.md)** - Pre-configured prompts
- **[Claude Desktop Integration](./04-claude-desktop-integration.md)** - Using resources in Claude

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team
