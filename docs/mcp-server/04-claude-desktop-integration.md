---
title: "Claude Desktop Integration Guide"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "End Users, Administrators"
---

# Claude Desktop Integration Guide

## Overview

This guide shows you how to integrate the Zenoo RPC MCP Server with **Claude Desktop**, enabling you to interact with your Odoo ERP system using natural language through Claude.

**What You'll Achieve**:
- ✅ Connect Claude Desktop to your Odoo server
- ✅ Query Odoo data using natural language
- ✅ Create, update, and analyze Odoo records via chat
- ✅ Generate reports and insights from Odoo data

**Estimated Time**: 5-10 minutes
**Difficulty**: Beginner

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Configuration Steps](#configuration-steps)
3. [Testing the Connection](#testing-the-connection)
4. [Example Queries](#example-queries)
5. [Advanced Usage](#advanced-usage)
6. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Software

1. **Claude Desktop** (latest version)
   - Download from: https://claude.ai/desktop
   - Supported: macOS, Windows, Linux

2. **Zenoo RPC MCP Server** installed
   ```bash
   pip install zenoo-rpc[mcp]
   ```
   See [Installation Guide](./02-installation-setup.md) if not yet installed

3. **Access to Odoo Server**
   - Odoo URL (e.g., `http://localhost:8069`)
   - Database name
   - Valid username and password

### System Requirements

- Python 3.8+ (with `zenoo-rpc[mcp]` installed)
- Network access to Odoo server
- Claude Desktop subscription (Free or Pro)

---

## Configuration Steps

### Step 1: Locate Claude Desktop Configuration

Claude Desktop configuration is stored in a JSON file:

**macOS**:
```
~/.config/claude/claude_desktop_config.json
```

**Windows**:
```
%APPDATA%\Claude\claude_desktop_config.json
```

**Linux**:
```
~/.config/claude/claude_desktop_config.json
```

### Step 2: Create/Edit Configuration File

If the file doesn't exist, create it:

```bash
# macOS/Linux
mkdir -p ~/.config/claude
nano ~/.config/claude/claude_desktop_config.json

# Windows (PowerShell)
mkdir -Force $env:APPDATA\Claude
notepad $env:APPDATA\Claude\claude_desktop_config.json
```

### Step 3: Add Zenoo RPC MCP Server Configuration

**Basic Configuration** (minimum required):

```json
{
  "mcpServers": {
    "odoo": {
      "command": "python",
      "args": ["-m", "zenoo_rpc.mcp_server.cli"],
      "env": {
        "ODOO_URL": "http://localhost:8069",
        "ODOO_DATABASE": "demo",
        "ODOO_USERNAME": "admin",
        "ODOO_PASSWORD": "admin"
      }
    }
  }
}
```

**Advanced Configuration** (with additional options):

```json
{
  "mcpServers": {
    "odoo-production": {
      "command": "python",
      "args": ["-m", "zenoo_rpc.mcp_server.cli"],
      "env": {
        "ODOO_URL": "https://your-company.odoo.com",
        "ODOO_DATABASE": "production",
        "ODOO_USERNAME": "admin",
        "ODOO_PASSWORD": "your-secure-password",
        "MCP_SERVER_NAME": "My Company Odoo",
        "MCP_LOG_LEVEL": "INFO",
        "MCP_ENABLE_CACHING": "true"
      }
    }
  }
}
```

**Multiple Odoo Instances**:

```json
{
  "mcpServers": {
    "odoo-dev": {
      "command": "python",
      "args": ["-m", "zenoo_rpc.mcp_server.cli"],
      "env": {
        "ODOO_URL": "http://localhost:8069",
        "ODOO_DATABASE": "development",
        "ODOO_USERNAME": "admin",
        "ODOO_PASSWORD": "admin",
        "MCP_SERVER_NAME": "Odoo Development"
      }
    },
    "odoo-prod": {
      "command": "python",
      "args": ["-m", "zenoo_rpc.mcp_server.cli"],
      "env": {
        "ODOO_URL": "https://prod.example.com",
        "ODOO_DATABASE": "production",
        "ODOO_USERNAME": "admin",
        "ODOO_PASSWORD": "secure-password",
        "MCP_SERVER_NAME": "Odoo Production"
      }
    }
  }
}
```

### Step 4: Configuration Parameters Explained

| Parameter | Required | Description | Example |
|-----------|----------|-------------|---------|
| `ODOO_URL` | ✅ Yes | Your Odoo server URL | `http://localhost:8069` |
| `ODOO_DATABASE` | ✅ Yes | Odoo database name | `demo` or `production` |
| `ODOO_USERNAME` | ✅ Yes | Odoo username | `admin` |
| `ODOO_PASSWORD` | ✅ Yes | Odoo password | `admin` |
| `MCP_SERVER_NAME` | ❌ No | Friendly name in Claude | `My Company Odoo` |
| `MCP_LOG_LEVEL` | ❌ No | Logging verbosity | `INFO`, `DEBUG`, `WARNING` |
| `MCP_ENABLE_CACHING` | ❌ No | Enable caching | `true` or `false` |

### Step 5: Restart Claude Desktop

After saving the configuration:

1. **Quit Claude Desktop completely**
   - macOS: `Cmd + Q`
   - Windows: Right-click taskbar icon → Quit
   - Linux: Close all windows

2. **Restart Claude Desktop**

3. **Verify MCP Server Loaded**
   - Look for the 🔌 (plug) icon next to the input box
   - Click the icon to see available MCP servers
   - "odoo" (or your custom name) should appear in the list

---

## Testing the Connection

### Visual Confirmation

When Claude Desktop successfully connects to the MCP server, you'll see:

1. **🔌 Icon**: Appears next to the message input box
2. **Server Listed**: "odoo" appears in the MCP servers list when you click the icon
3. **Tools Available**: Green dot indicates tools are ready

### Test Query 1: Basic Search

Try this simple query to test connectivity:

```
Can you search for the first 5 partners in Odoo?
```

**Expected Response**:
```
Claude: I'll search for partners in your Odoo system.

[Uses search_records tool]

I found 5 partners:
1. YourCompany (ID: 1)
2. Administrator (ID: 2)
3. Mitchell Admin (ID: 3)
4. Azure Interior (ID: 4)
5. Brandon Freeman (ID: 5)

Would you like more details about any of these partners?
```

### Test Query 2: Specific Search

```
Show me all companies (not individuals) in Odoo
```

**Expected Response**:
```
Claude: I'll search for company partners (where is_company is True).

[Uses search_records with domain filter]

I found 23 companies in your Odoo system:
- Azure Interior
- Gemini Furniture
- Ready Mat
...

Would you like me to show more details or filter further?
```

### Test Query 3: Create Record

```
Create a new partner named "Test Customer" with email test@example.com
```

**Expected Response**:
```
Claude: I'll create a new partner with those details.

[Uses create_record tool]

✅ Successfully created partner:
- Name: Test Customer
- Email: test@example.com
- ID: 156

Would you like to add more information to this partner?
```

---

## Example Queries

### Customer Management

#### Search Customers
```
Find all customers in California
```

```
Show me customers who made purchases in the last 30 days
```

```
List all partners with email addresses ending in @gmail.com
```

#### Create/Update Customers
```
Create a new customer:
- Name: Acme Corporation
- Email: contact@acme.com
- Phone: +1-555-0123
- Country: United States
```

```
Update partner ID 50 to set their phone to +1-555-9999
```

### Sales Analysis

```
What were our top 5 selling products last month?
```

```
Show me all sales orders from this week
```

```
How many orders are in "draft" status?
```

```
What's the total revenue from sales orders this quarter?
```

### Inventory Management

```
Which products have stock quantity less than 10?
```

```
Show me all products in the "Electronics" category
```

```
What's the current stock level for product "Office Chair"?
```

### Reporting

```
Generate a summary of sales by salesperson for this month
```

```
Show me the top 10 customers by total purchase amount
```

```
What percentage of our sales orders are in "confirmed" status?
```

### Data Analysis

```
Analyze customer distribution by country and show me the top 5 countries
```

```
Compare sales revenue between Q1 and Q2 of this year
```

```
Show me trends in customer acquisition over the last 6 months
```

---

## Advanced Usage

### Using Multiple Tools in One Query

Claude can chain multiple tool calls:

```
Find the customer "Azure Interior", show me their details,
and list all their sales orders from this year
```

Claude will:
1. Use `search_records` to find the customer
2. Use `get_record` to get full details
3. Use `complex_search` to find related sales orders

### Analytics Queries

```
Group all sales orders by month for this year and show me the
count and total amount for each month
```

Claude will use `analytics_query` tool with:
- Grouping by month
- Aggregations: count and sum

### Batch Operations

```
Create the following 3 customers in one operation:
1. Alpha Corp - alpha@corp.com
2. Beta Inc - beta@inc.com
3. Gamma LLC - gamma@llc.com
```

Claude will use `batch_operation` for efficient bulk creation.

### Natural Language to Odoo Domain

Claude automatically translates natural language to Odoo domain syntax:

| Your Query | Odoo Domain Claude Uses |
|------------|------------------------|
| "customers in California" | `[('state', '=', 'CA'), ('customer', '=', True)]` |
| "companies, not individuals" | `[('is_company', '=', True)]` |
| "sales orders from last week" | `[('date_order', '>=', '2025-11-08'), ('date_order', '<=', '2025-11-15')]` |
| "products with low stock" | `[('qty_available', '<', 10)]` |

---

## Configuration Best Practices

### Security Considerations

1. **Use Dedicated User**:
   ```json
   {
     "env": {
       "ODOO_USERNAME": "mcp_readonly",
       "ODOO_PASSWORD": "strong-password-here"
     }
   }
   ```
   Create a dedicated Odoo user with only necessary permissions

2. **Avoid Hardcoding Passwords**:
   ```bash
   # Store password in keychain/credential manager
   # Reference via environment variable
   export ODOO_PASSWORD=$(security find-generic-password -a mcp -w)
   ```

3. **Use HTTPS for Production**:
   ```json
   {
     "env": {
       "ODOO_URL": "https://secure.example.com"
     }
   }
   ```

### Performance Optimization

1. **Enable Caching**:
   ```json
   {
     "env": {
       "MCP_ENABLE_CACHING": "true",
       "REDIS_URL": "redis://localhost:6379/0"
     }
   }
   ```

2. **Adjust Log Level**:
   ```json
   {
     "env": {
       "MCP_LOG_LEVEL": "WARNING"  // Less verbose in production
     }
   }
   ```

### Multi-Environment Setup

**Development + Production**:

```json
{
  "mcpServers": {
    "odoo-dev": {
      "command": "python",
      "args": ["-m", "zenoo_rpc.mcp_server.cli"],
      "env": {
        "ODOO_URL": "http://localhost:8069",
        "ODOO_DATABASE": "dev",
        "ODOO_USERNAME": "admin",
        "ODOO_PASSWORD": "admin",
        "MCP_SERVER_NAME": "Odoo Dev 🔧"
      }
    },
    "odoo-prod": {
      "command": "python",
      "args": ["-m", "zenoo_rpc.mcp_server.cli"],
      "env": {
        "ODOO_URL": "https://prod.example.com",
        "ODOO_DATABASE": "production",
        "ODOO_USERNAME": "mcp_user",
        "ODOO_PASSWORD": "secure-password",
        "MCP_SERVER_NAME": "Odoo Production 🚀"
      }
    }
  }
}
```

Now in Claude Desktop:
- Select "Odoo Dev 🔧" for development queries
- Select "Odoo Production 🚀" for production queries

---

## Troubleshooting

### Issue: MCP Server Not Appearing in Claude Desktop

**Symptoms**:
- 🔌 icon doesn't appear
- Server not listed when clicking icon

**Solutions**:

1. **Check Configuration File Location**:
   ```bash
   # macOS/Linux
   ls -la ~/.config/claude/claude_desktop_config.json

   # Windows (PowerShell)
   Test-Path $env:APPDATA\Claude\claude_desktop_config.json
   ```

2. **Validate JSON Syntax**:
   ```bash
   # Use online JSON validator or:
   python -m json.tool ~/.config/claude/claude_desktop_config.json
   ```

3. **Check Python/Zenoo RPC Installation**:
   ```bash
   # Test MCP server can start
   python -m zenoo_rpc.mcp_server.cli --validate-config
   ```

4. **Review Claude Desktop Logs**:
   - macOS: `~/Library/Logs/Claude/`
   - Windows: `%APPDATA%\Claude\logs\`
   - Linux: `~/.config/claude/logs/`

### Issue: Connection to Odoo Fails

**Error in Claude**:
```
Error: Could not connect to Odoo server
```

**Solutions**:

1. **Test Odoo Connectivity**:
   ```bash
   curl http://localhost:8069/web/database/selector
   ```

2. **Verify Credentials**:
   ```bash
   python3 << EOF
   from zenoo_rpc import ZenooClient
   import asyncio

   async def test():
       async with ZenooClient("http://localhost:8069") as client:
           await client.login("demo", "admin", "admin")
           print("✅ Connection successful!")

   asyncio.run(test())
   EOF
   ```

3. **Check Configuration**:
   ```json
   {
     "env": {
       "ODOO_URL": "http://localhost:8069",  // Correct protocol (http/https)?
       "ODOO_DATABASE": "demo",               // Database exists?
       "ODOO_USERNAME": "admin",              // User exists?
       "ODOO_PASSWORD": "admin"               // Password correct?
     }
   }
   ```

### Issue: Tools Not Working

**Error in Claude**:
```
Tool execution failed: [error message]
```

**Solutions**:

1. **Enable Debug Logging**:
   ```json
   {
     "env": {
       "MCP_LOG_LEVEL": "DEBUG"
     }
   }
   ```

2. **Check Odoo User Permissions**:
   - Ensure user has access to models being queried
   - Check Odoo access rights and record rules

3. **Test Tool Directly**:
   ```python
   # Test search_records tool
   from zenoo_rpc import ZenooClient
   import asyncio

   async def test():
       async with ZenooClient("http://localhost:8069") as client:
           await client.login("demo", "admin", "admin")
           results = await client.search_read("res.partner", [], limit=5)
           print(f"Found {len(results)} records")

   asyncio.run(test())
   ```

### Issue: Slow Performance

**Symptoms**:
- Queries take >5 seconds
- Claude shows "thinking" for extended periods

**Solutions**:

1. **Enable Caching**:
   ```json
   {
     "env": {
       "MCP_ENABLE_CACHING": "true"
     }
   }
   ```

2. **Optimize Odoo Performance**:
   - Add database indexes
   - Optimize Odoo server resources
   - Check network latency

3. **Limit Result Sets**:
   ```
   Show me only the first 10 customers
   ```

### Issue: Python Command Not Found

**Error**:
```
Command 'python' not found
```

**Solutions**:

1. **Use Full Python Path**:
   ```json
   {
     "command": "/usr/local/bin/python3",  // or your Python path
     "args": ["-m", "zenoo_rpc.mcp_server.cli"]
   }
   ```

2. **Find Python Path**:
   ```bash
   # macOS/Linux
   which python3

   # Windows (PowerShell)
   (Get-Command python).Path
   ```

3. **Use Virtual Environment**:
   ```json
   {
     "command": "/path/to/venv/bin/python",
     "args": ["-m", "zenoo_rpc.mcp_server.cli"]
   }
   ```

---

## Related Documentation

- **[Installation & Setup](./02-installation-setup.md)** - Install MCP server
- **[Configuration Guide](./03-configuration.md)** - Advanced configuration
- **[Tools Reference](./05-tools-reference.md)** - Available MCP tools
- **[Troubleshooting](./10-troubleshooting.md)** - Common issues
- **[Use Cases](./use-cases/)** - Real-world examples

---

## Tips for Best Results

### Effective Prompting

**✅ Good Prompts**:
```
Find all companies in California with more than 100 employees
Create a customer named "Acme Corp" with email acme@example.com
Show me sales orders from last quarter grouped by salesperson
```

**❌ Less Effective Prompts**:
```
Find stuff
Create a customer
Show me things
```

### Context Matters

Provide context for better results:
```
I'm looking for customers who haven't made a purchase in 6 months
so I can reach out to them with a special offer
```

### Iterative Refinement

Start broad, then refine:
```
1. "Show me all sales orders"
2. "Filter those to only show orders from last month"
3. "Group them by salesperson and show total amounts"
```

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team

---

**Ready to start querying?** Try the example queries above or explore [Use Cases](./use-cases/) for more ideas!
