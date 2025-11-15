---
title: "Zenoo RPC MCP Server Documentation"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
---

# Zenoo RPC MCP Server Documentation

## Overview

The **Zenoo RPC MCP Server** enables AI assistants like Claude Desktop, ChatGPT, and other MCP-compatible tools to interact directly with your Odoo ERP system through the Model Context Protocol (MCP). This powerful integration allows AI assistants to search, create, update, and analyze Odoo data using natural language.

**Key Benefits**:
- 🤖 **AI-Native Integration**: Natural language queries to Odoo data
- 🔒 **Secure**: Built-in authentication, authorization, and rate limiting
- ⚡ **High Performance**: Async operations with intelligent caching
- 🛡️ **Type Safe**: Full Pydantic validation for all operations
- 🔄 **Transaction Support**: ACID-compliant operations with rollback
- 📊 **Rich Analytics**: Built-in aggregation and reporting tools

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Documentation Sections](#documentation-sections)
3. [Quick Links](#quick-links)
4. [Use Cases](#use-cases)

---

## Getting Started

### For Claude Desktop Users

Get started in 3 steps:

1. **Install Zenoo RPC with MCP support**:
   ```bash
   pip install zenoo-rpc[mcp]
   ```

2. **Configure Claude Desktop** (see [Claude Desktop Integration](./claude-desktop-integration.md))

3. **Start using natural language** with your Odoo data:
   ```
   User: "Find all customers in California who made purchases last month"
   Claude: [Uses MCP tools to query Odoo and analyze data]
   ```

### For Developers

Integrate Zenoo RPC MCP Server into your workflow:

1. **Quick Start**: [Installation & Setup](./installation-setup.md)
2. **Configuration**: [Configuration Guide](./configuration.md)
3. **Security**: [Security & Authentication](./security.md)
4. **Deployment**: [Deployment Guide](./deployment.md)

---

## Documentation Sections

### 📚 Core Documentation

| Document | Description | Audience |
|----------|-------------|----------|
| [**01. Overview & Architecture**](./01-overview-architecture.md) | System architecture, components, and design | All |
| [**02. Installation & Setup**](./02-installation-setup.md) | Complete installation guide | Administrators |
| [**03. Configuration**](./03-configuration.md) | Server configuration and environment variables | Administrators |
| [**04. Claude Desktop Integration**](./04-claude-desktop-integration.md) | Step-by-step Claude Desktop setup | End Users, Administrators |
| [**05. MCP Tools Reference**](./05-tools-reference.md) | Complete reference for all MCP tools | Developers, End Users |
| [**06. MCP Resources Reference**](./06-resources-reference.md) | Resource URIs and data access | Developers |
| [**07. MCP Prompts Reference**](./07-prompts-reference.md) | Pre-built prompt templates | End Users |
| [**08. Security & Authentication**](./08-security-authentication.md) | Security best practices and auth methods | Administrators, Developers |
| [**09. Deployment Guide**](./09-deployment.md) | Production deployment strategies | Administrators, DevOps |
| [**10. Troubleshooting**](./10-troubleshooting.md) | Common issues and solutions | All |

### 🔧 Advanced Topics

| Document | Description | Audience |
|----------|-------------|----------|
| [**11. Advanced Topics**](./11-advanced/README.md) | Customization and extension guides | Developers |
| [**11.1 Custom MCP Clients**](./11-advanced/custom-clients.md) | Build your own MCP clients | Developers |
| [**11.2 Extending the Server**](./11-advanced/extending-server.md) | Add custom tools and plugins | Developers |

### 🐛 Issues & Planning

| Document | Description | Audience |
|----------|-------------|----------|
| [**12. Known Issues**](./12-known-issues/README.md) | Current limitations and workarounds | All |
| [**13. Roadmap**](./13-roadmap/README.md) | Planned features and timeline | All |

### 🎯 Use Case Guides

| Guide | Description |
|-------|-------------|
| [**Customer Service Automation**](./use-cases/customer-service.md) | Using AI to assist with customer queries |
| [**Sales Analytics**](./use-cases/sales-analytics.md) | AI-powered sales data analysis |
| [**Inventory Management**](./use-cases/inventory-management.md) | Smart inventory queries and insights |
| [**Report Generation**](./use-cases/report-generation.md) | Automated report creation with AI |

### 🔧 Advanced Topics

| Topic | Description |
|-------|-------------|
| [**Custom MCP Clients**](./advanced/custom-clients.md) | Building custom MCP clients |
| [**Performance Optimization**](./advanced/performance.md) | Optimizing MCP server performance |
| [**Extending the Server**](./advanced/extending.md) | Adding custom tools and resources |

---

## Quick Links

### New to MCP Server?
1. Start with [Overview & Architecture](./01-overview-architecture.md)
2. Follow [Installation & Setup](./02-installation-setup.md)
3. Try [Claude Desktop Integration](./04-claude-desktop-integration.md)
4. Explore [Tools Reference](./05-tools-reference.md)

### Already Using the MCP Server?
- **Looking for a specific tool?** → [Tools Reference](./05-tools-reference.md)
- **Need to configure security?** → [Security Guide](./08-security-authentication.md)
- **Having issues?** → [Troubleshooting](./10-troubleshooting.md)
- **Want to deploy to production?** → [Deployment Guide](./09-deployment.md)

### For Administrators
1. [Configuration Guide](./03-configuration.md)
2. [Security & Authentication](./08-security-authentication.md)
3. [Deployment Guide](./09-deployment.md)
4. [Troubleshooting](./10-troubleshooting.md)

### For End Users
1. [Claude Desktop Integration](./04-claude-desktop-integration.md)
2. [MCP Tools Reference](./05-tools-reference.md)
3. [Use Case Guides](./use-cases/)
4. [Prompts Reference](./07-prompts-reference.md)

### For Developers
1. [Overview & Architecture](./01-overview-architecture.md)
2. [Tools Reference](./05-tools-reference.md)
3. [Resources Reference](./06-resources-reference.md)
4. [Custom Clients](./advanced/custom-clients.md)

---

## Use Cases

### 🎯 Real-World Scenarios

#### Customer Service
```
User: "Show me all open support tickets for customers in New York"
Claude: [Searches res.partner and helpdesk.ticket models]
        "Found 23 open tickets. Here's a summary..."
```

#### Sales Analysis
```
User: "What were our top 5 products last quarter?"
Claude: [Analyzes sale.order.line data with aggregation]
        "Here are your top 5 products by revenue..."
```

#### Inventory Management
```
User: "Which products are running low on stock?"
Claude: [Queries product.product with stock quantity filters]
        "Found 12 products below reorder point..."
```

#### Report Generation
```
User: "Generate a monthly sales report for the marketing team"
Claude: [Uses analytics_query tool with date ranges and grouping]
        "I've analyzed the data. Here's your report..."
```

---

## What is MCP?

**Model Context Protocol (MCP)** is an open protocol that enables AI assistants to interact with external systems and data sources. Think of it as a standardized API that AI tools use to:

- **Execute Actions** (Tools): Perform operations like search, create, update
- **Access Data** (Resources): Retrieve information from your systems
- **Use Templates** (Prompts): Pre-configured queries and workflows

### Why MCP for Odoo?

Traditional Odoo integrations require:
- Writing custom code for each operation
- Managing authentication and sessions
- Handling Odoo's complex domain syntax
- Building UI for data access

**With Zenoo RPC MCP Server**, AI assistants can:
- Use natural language to query Odoo
- Automatically handle authentication
- Translate requests to Odoo domains
- Present data in user-friendly formats

---

## Architecture Overview

```
┌─────────────────────┐
│   Claude Desktop    │
│   or AI Assistant   │
└──────────┬──────────┘
           │ MCP Protocol
           │
┌──────────▼──────────┐
│  Zenoo RPC MCP      │
│  Server             │
│  ┌───────────────┐  │
│  │ Security      │  │
│  │ & Auth        │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │ MCP Tools     │  │
│  │ (search,      │  │
│  │  create,      │  │
│  │  analytics)   │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │ Zenoo RPC     │  │
│  │ Client        │  │
│  └───────────────┘  │
└──────────┬──────────┘
           │ Odoo RPC/JSON-RPC
           │
┌──────────▼──────────┐
│   Odoo Server       │
│   (Any Version)     │
└─────────────────────┘
```

---

## MCP Server Features

### 🛠️ Tools (8 Available)

**Basic Operations**:
- `search_records` - Search any Odoo model
- `get_record` - Get specific record by ID
- `create_record` - Create new records
- `update_record` - Update existing records
- `delete_record` - Delete records

**Advanced Operations**:
- `complex_search` - Advanced queries with relationships
- `batch_operation` - High-performance bulk operations
- `analytics_query` - Aggregation and reporting

See [Tools Reference](./05-tools-reference.md) for complete documentation.

### 📦 Resources (3 Categories)

- `odoo://models` - List all available Odoo models
- `odoo://model/{name}` - Get model schema and fields
- `odoo://record/{model}/{id}` - Get record data

See [Resources Reference](./06-resources-reference.md) for details.

### 📝 Prompts (Templates)

- `analyze_data` - Data analysis templates
- `generate_report_query` - Report generation
- `suggest_workflow` - Workflow optimization

See [Prompts Reference](./07-prompts-reference.md) for examples.

---

## Supported Transport Types

The MCP server supports multiple transport protocols:

| Transport | Use Case | Configuration |
|-----------|----------|---------------|
| **stdio** | Claude Desktop, local tools | Default, no network required |
| **HTTP** | Web applications, REST APIs | Requires host/port configuration |
| **SSE** | Server-sent events, streaming | For real-time updates |

See [Configuration Guide](./03-configuration.md) for transport setup.

---

## System Requirements

### Minimum Requirements
- Python 3.8 or higher
- Odoo 14.0 or higher (other versions untested)
- Network access to Odoo server
- 512MB RAM minimum

### Recommended Requirements
- Python 3.10 or higher
- Odoo 16.0 or higher
- Redis for caching (optional)
- 1GB+ RAM for production

### Supported Platforms
- ✅ Linux (Ubuntu, Debian, CentOS, RHEL)
- ✅ macOS (10.15+)
- ✅ Windows 10/11
- ✅ Docker containers

---

## Security Features

- 🔐 **Multiple Auth Methods**: API keys, OAuth, session tokens
- 🛡️ **Rate Limiting**: Prevent abuse and DoS
- 🔒 **Input Validation**: Pydantic-based validation
- 📊 **Audit Logging**: Track all operations
- 🚫 **Permission Control**: Odoo-based access control
- 🔑 **Session Management**: Secure session handling

See [Security Guide](./08-security-authentication.md) for complete details.

---

## Getting Help

### Documentation
- **This Guide**: Comprehensive MCP server documentation
- **Python Library**: [User Guide](../user-guide/README.md)
- **API Reference**: [API Documentation](../api-reference/)
- **Examples**: [Real-World Examples](../examples/real-world/)

### Support Channels
- **GitHub Issues**: [Report bugs](https://github.com/tuanle96/zenoo-rpc/issues)
- **GitHub Discussions**: [Ask questions](https://github.com/tuanle96/zenoo-rpc/discussions)
- **Troubleshooting**: [Common Issues](./10-troubleshooting.md)

### Community
- **Discord**: [Join our community](#) *(coming soon)*
- **Twitter**: [@zenoorpc](#) *(coming soon)*

---

## Next Steps

### For First-Time Users
1. Read [Overview & Architecture](./01-overview-architecture.md)
2. Follow [Installation & Setup](./02-installation-setup.md)
3. Configure [Claude Desktop Integration](./04-claude-desktop-integration.md)
4. Try example queries from [Tools Reference](./05-tools-reference.md)

### For Administrators
1. Review [Configuration Guide](./03-configuration.md)
2. Set up [Security & Authentication](./08-security-authentication.md)
3. Plan [Production Deployment](./09-deployment.md)
4. Monitor using deployment guides

### For Developers
1. Understand [Architecture](./01-overview-architecture.md)
2. Explore [Tools API](./05-tools-reference.md)
3. Build [Custom Clients](./advanced/custom-clients.md)
4. Contribute improvements

---

## Related Documentation

- **[Python Library User Guide](../user-guide/README.md)** - Using Zenoo RPC as a library
- **[API Reference](../api-reference/)** - Complete API documentation
- **[Examples](../examples/)** - Code examples and patterns
- **[AI Features](../ai-features.md)** - AI-powered capabilities

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team

---

**Ready to get started?** → [Installation & Setup](./02-installation-setup.md)
