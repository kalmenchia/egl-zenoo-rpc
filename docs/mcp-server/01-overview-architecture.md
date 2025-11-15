---
title: "MCP Server Overview & Architecture"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "All"
---

# MCP Server Overview & Architecture

## Overview

The Zenoo RPC MCP Server is a production-ready implementation of the **Model Context Protocol (MCP)** that bridges AI assistants with Odoo ERP systems. It enables natural language interaction with Odoo data through a secure, high-performance, and type-safe interface.

**Purpose**: Enable AI tools like Claude Desktop, ChatGPT, and custom MCP clients to query, analyze, and manipulate Odoo data using natural language.

**Key Benefits**:
- 🤖 Natural language queries to Odoo
- 🛡️ Type-safe operations with Pydantic validation
- ⚡ High performance with async operations and caching
- 🔒 Enterprise-grade security and authentication
- 🔄 ACID-compliant transactions
- 📊 Built-in analytics and aggregation

---

## Table of Contents

1. [What is MCP?](#what-is-mcp)
2. [System Architecture](#system-architecture)
3. [Core Components](#core-components)
4. [Data Flow](#data-flow)
5. [Deployment Architectures](#deployment-architectures)
6. [Performance Characteristics](#performance-characteristics)
7. [Security Model](#security-model)

---

## What is MCP?

### Model Context Protocol Explained

The **Model Context Protocol (MCP)** is an open standard that allows AI models to interact with external systems and data sources in a structured, secure way.

Think of MCP as a **universal adapter** between AI assistants and your data:

```
┌─────────────┐         ┌──────────────┐         ┌────────────┐
│   User      │────────>│  AI Model    │────────>│    MCP     │
│ (Natural    │  Text   │  (Claude,    │  MCP    │   Server   │
│  Language)  │         │   ChatGPT)   │Protocol │            │
└─────────────┘         └──────────────┘         └──────┬─────┘
                                                          │
                                                          │
                                                    ┌─────▼────┐
                                                    │  Odoo    │
                                                    │  Server  │
                                                    └──────────┘
```

### MCP Capabilities

MCP provides three main capabilities:

#### 1. **Tools** (Actions)
Functions the AI can execute:
- `search_records` - Query Odoo models
- `create_record` - Create new data
- `analytics_query` - Run aggregations

#### 2. **Resources** (Data)
Read-only data the AI can access:
- `odoo://models` - List of models
- `odoo://model/res.partner` - Model schema
- `odoo://record/sale.order/123` - Specific record

#### 3. **Prompts** (Templates)
Pre-configured workflows:
- `analyze_data` - Data analysis template
- `generate_report` - Report generation template

### Why MCP for Odoo?

**Traditional Approach**:
```python
# Manual code required for each query
from odoorpc import ODOO
odoo = ODOO('localhost', port=8069)
odoo.login('db', 'user', 'password')
Partner = odoo.env['res.partner']
ids = Partner.search([('is_company', '=', True)])
partners = Partner.browse(ids)  # Multiple RPC calls!
```

**With MCP**:
```
User: "Find all companies in California"
Claude: [Uses MCP to execute query automatically]
        "Found 47 companies in California..."
```

---

## System Architecture

### High-Level Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                         AI Client Layer                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Claude     │  │   ChatGPT    │  │   Custom     │         │
│  │   Desktop    │  │   (via MCP)  │  │   MCP Client │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
└─────────┼──────────────────┼──────────────────┼────────────────┘
          │                  │                  │
          │            MCP Protocol              │
          │        (stdio / HTTP / SSE)          │
          │                  │                  │
┌─────────▼──────────────────▼──────────────────▼────────────────┐
│                    Zenoo RPC MCP Server                         │
│  ┌────────────────────────────────────────────────────────┐    │
│  │              MCP Server Layer (FastMCP)                │    │
│  │  • Protocol handling                                   │    │
│  │  • Request validation                                  │    │
│  │  • Response formatting                                 │    │
│  └───────────────────────┬────────────────────────────────┘    │
│                          │                                      │
│  ┌───────────────────────▼────────────────────────────────┐    │
│  │              Security & Validation Layer               │    │
│  │  • Authentication (API keys, OAuth)                    │    │
│  │  • Authorization (permissions check)                   │    │
│  │  • Rate limiting                                       │    │
│  │  • Input sanitization (Pydantic validation)           │    │
│  │  • Audit logging                                       │    │
│  └───────────────────────┬────────────────────────────────┘    │
│                          │                                      │
│  ┌───────────────────────▼────────────────────────────────┐    │
│  │                  Tool Handler Layer                    │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐  │    │
│  │  │   Basic      │  │   Advanced   │  │  Analytics  │  │    │
│  │  │   CRUD       │  │   Search     │  │  Queries    │  │    │
│  │  │   Tools      │  │   Tools      │  │             │  │    │
│  │  └──────────────┘  └──────────────┘  └─────────────┘  │    │
│  └───────────────────────┬────────────────────────────────┘    │
│                          │                                      │
│  ┌───────────────────────▼────────────────────────────────┐    │
│  │               Zenoo RPC Client Layer                   │    │
│  │  • Type-safe models (Pydantic)                         │    │
│  │  • Query builder                                       │    │
│  │  • Transaction manager                                 │    │
│  │  • Cache manager                                       │    │
│  │  • Batch operations                                    │    │
│  │  • Async HTTP transport                                │    │
│  └───────────────────────┬────────────────────────────────┘    │
└──────────────────────────┼─────────────────────────────────────┘
                           │
                    Odoo JSON-RPC / XML-RPC
                           │
┌──────────────────────────▼─────────────────────────────────────┐
│                      Odoo Server                                │
│  • Business logic                                               │
│  • Database (PostgreSQL)                                        │
│  • Security (ACL, Record Rules)                                 │
│  • Workflows                                                    │
└─────────────────────────────────────────────────────────────────┘
```

### Component Diagram

```
┌──────────────────────────────────────────────────────────┐
│              Zenoo RPC MCP Server                        │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │  src/zenoo_rpc/mcp_server/                        │ │
│  │                                                    │ │
│  │  ┌──────────────┐  ┌──────────────┐              │ │
│  │  │  server.py   │  │   cli.py     │              │ │
│  │  │  (Core MCP)  │  │  (CLI Entry) │              │ │
│  │  └──────┬───────┘  └──────┬───────┘              │ │
│  │         │                  │                      │ │
│  │  ┌──────▼──────────────────▼───────┐             │ │
│  │  │        config.py                │             │ │
│  │  │  (Configuration Management)     │             │ │
│  │  └──────────────────────────────────┘             │ │
│  │                                                    │ │
│  │  ┌──────────────┐  ┌──────────────┐              │ │
│  │  │ security.py  │  │exceptions.py │              │ │
│  │  │ (Auth & ACL) │  │ (Error Types)│              │ │
│  │  └──────────────┘  └──────────────┘              │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │  src/zenoo_rpc/ (Core Library)                    │ │
│  │                                                    │ │
│  │  ├── client.py          (Async Odoo client)       │ │
│  │  ├── models/            (Pydantic models)         │ │
│  │  ├── query/             (Query builder)           │ │
│  │  ├── transaction/       (Transaction support)     │ │
│  │  ├── cache/             (Caching layer)           │ │
│  │  ├── batch/             (Batch operations)        │ │
│  │  └── transport/         (HTTP transport)          │ │
│  └────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. MCP Server Core (`server.py`)

**File**: `src/zenoo_rpc/mcp_server/server.py`

**Responsibilities**:
- Implement MCP protocol using FastMCP
- Register and manage tools, resources, prompts
- Handle tool execution and response formatting
- Manage Odoo connection lifecycle
- Coordinate security and validation

**Key Classes**:
```python
class ZenooMCPServer:
    """Main MCP server implementation.

    Exposes Odoo operations through MCP protocol.
    """

    def __init__(self, config: MCPServerConfig):
        """Initialize server with configuration."""

    async def start(self) -> None:
        """Start MCP server with configured transport."""

    def _register_tools(self) -> None:
        """Register all available MCP tools."""

    def _register_resources(self) -> None:
        """Register MCP resources."""

    async def _execute_tool(self, tool_name: str, args: Dict) -> Dict:
        """Execute tool with security validation."""
```

**MCP Tools Registered** (8 tools):
- `search_records` - Search Odoo models
- `get_record` - Get specific record
- `create_record` - Create new record
- `update_record` - Update existing record
- `delete_record` - Delete record
- `complex_search` - Advanced search with filters
- `batch_operation` - Bulk operations
- `analytics_query` - Aggregation queries

### 2. CLI Interface (`cli.py`)

**File**: `src/zenoo_rpc/mcp_server/cli.py`

**Responsibilities**:
- Command-line interface for server management
- Configuration loading (file, env vars, CLI args)
- Logging setup
- Signal handling for graceful shutdown

**Usage**:
```bash
# Start with default settings
python -m zenoo_rpc.mcp_server.cli

# Custom configuration
python -m zenoo_rpc.mcp_server.cli --config config.json

# HTTP transport
python -m zenoo_rpc.mcp_server.cli --transport http --port 8080
```

### 3. Configuration Manager (`config.py`)

**File**: `src/zenoo_rpc/mcp_server/config.py`

**Responsibilities**:
- Load configuration from multiple sources
- Validate configuration
- Provide typed configuration objects

**Configuration Sources** (priority order):
1. CLI arguments (highest priority)
2. Configuration file (JSON)
3. Environment variables
4. Default values (lowest priority)

**Key Configuration**:
```python
class MCPServerConfig:
    # Server settings
    name: str = "zenoo-mcp-server"
    transport_type: MCPTransportType = MCPTransportType.STDIO
    host: str = "localhost"
    port: int = 8000

    # Odoo connection
    odoo_url: str = "http://localhost:8069"
    odoo_database: str = "demo"
    odoo_username: str = "admin"
    odoo_password: str = "admin"

    # Security
    security: MCPSecurityConfig

    # Features
    features: MCPFeaturesConfig
```

### 4. Security Manager (`security.py`)

**File**: `src/zenoo_rpc/mcp_server/security.py`

**Responsibilities**:
- Authentication (API keys, OAuth, session)
- Authorization (permission checks)
- Rate limiting
- Session management
- Audit logging

**Security Features**:
- Multiple authentication methods
- Per-user rate limiting
- Session expiration
- Request validation
- Audit trails

### 5. Zenoo RPC Client Integration

The MCP server leverages the full power of the Zenoo RPC library:

**Features Used**:
- ✅ **Type Safety**: Pydantic models for validation
- ✅ **Async Operations**: Non-blocking I/O
- ✅ **Query Builder**: Fluent, Django-like queries
- ✅ **Transactions**: ACID-compliant operations
- ✅ **Caching**: Intelligent cache management
- ✅ **Batch Operations**: High-performance bulk ops
- ✅ **Retry Logic**: Automatic retry with backoff

---

## Data Flow

### Request Flow (Search Records Example)

```
┌──────────┐
│  Claude  │ User: "Find all customers in California"
└────┬─────┘
     │ 1. Natural language → MCP request
     │
┌────▼──────────────────────────────────────────┐
│  MCP Protocol Layer                           │
│  Request: {                                    │
│    "tool": "search_records",                   │
│    "arguments": {                              │
│      "model": "res.partner",                   │
│      "domain": [["state", "=", "CA"]]          │
│    }                                           │
│  }                                             │
└────┬──────────────────────────────────────────┘
     │ 2. MCP request validation
     │
┌────▼──────────────────────────────────────────┐
│  Security Layer                                │
│  • Authenticate request (API key validation)   │
│  • Authorize operation (permission check)      │
│  • Rate limit check (requests/minute)          │
│  • Validate input (Pydantic schemas)           │
└────┬──────────────────────────────────────────┘
     │ 3. Security passed
     │
┌────▼──────────────────────────────────────────┐
│  Tool Handler Layer                            │
│  • Route to search_records handler             │
│  • Build Odoo search parameters                │
│  • Apply field selection and limits            │
└────┬──────────────────────────────────────────┘
     │ 4. Execute via Zenoo RPC
     │
┌────▼──────────────────────────────────────────┐
│  Zenoo RPC Client                              │
│  • Check cache (if enabled)                    │
│  • Build optimized query                       │
│  • Execute async search_read                   │
│  • Type validation (Pydantic)                  │
└────┬──────────────────────────────────────────┘
     │ 5. Odoo JSON-RPC call
     │
┌────▼──────────────────────────────────────────┐
│  Odoo Server                                   │
│  • Authenticate session                        │
│  • Check record rules (ACL)                    │
│  • Execute database query                      │
│  • Format response                             │
└────┬──────────────────────────────────────────┘
     │ 6. Return results
     │
┌────▼──────────────────────────────────────────┐
│  Response Processing                           │
│  • Cache results (if configured)               │
│  • Format for MCP protocol                     │
│  • Add metadata (count, has_more, etc.)        │
└────┬──────────────────────────────────────────┘
     │ 7. MCP response
     │
┌────▼──────────────────────────────────────────┐
│  Claude Desktop                                │
│  Response: {                                   │
│    "records": [...47 customers...],            │
│    "count": 47,                                │
│    "has_more": false                           │
│  }                                             │
└────┬──────────────────────────────────────────┘
     │ 8. AI formats for user
     │
┌────▼─────┐
│  User    │ "Found 47 customers in California..."
└──────────┘
```

### Transaction Flow (Create Record)

```
User Request → MCP Server → Start Transaction
                   ↓
            Validate Input (Pydantic)
                   ↓
            Execute Create (Zenoo RPC)
                   ↓
            Odoo RPC Call
                   ↓
         [Success] → Commit Transaction → Return Result
                ↓
         [Error] → Rollback Transaction → Return Error
```

---

## Deployment Architectures

### Architecture 1: Local Development (stdio)

```
┌────────────────────┐
│  Claude Desktop    │
│  (Local)           │
└────────┬───────────┘
         │ stdio
         │ (stdin/stdout)
         │
┌────────▼───────────┐
│  MCP Server        │
│  (Local Process)   │
└────────┬───────────┘
         │ HTTP/RPC
         │
┌────────▼───────────┐
│  Odoo Server       │
│  (localhost:8069)  │
└────────────────────┘
```

**Use Case**: Development, testing, personal use

**Pros**:
- Simple setup
- No network configuration
- Fast communication

**Cons**:
- Single user
- No remote access
- Limited scalability

### Architecture 2: HTTP Server (Multi-User)

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Claude      │    │  ChatGPT     │    │  Custom      │
│  Desktop     │    │  Client      │    │  Client      │
└──────┬───────┘    └──────┬───────┘    └──────┬───────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │ HTTPS (TLS)
                           │
                    ┌──────▼──────┐
                    │   Nginx/    │
                    │   Proxy     │
                    └──────┬──────┘
                           │
                    ┌──────▼──────────────┐
                    │  MCP Server         │
                    │  (HTTP Transport)   │
                    │  • Load balanced    │
                    │  • Redis cache      │
                    └──────┬──────────────┘
                           │
                    ┌──────▼──────────────┐
                    │  Odoo Server        │
                    │  (Production)       │
                    └─────────────────────┘
```

**Use Case**: Production, multi-user, enterprise

**Pros**:
- Multiple concurrent users
- Horizontal scaling
- Caching and load balancing
- Remote access

**Cons**:
- More complex setup
- Security configuration required
- Infrastructure overhead

### Architecture 3: Docker Deployment

```
┌─────────────────────────────────────────────────┐
│             Docker Host                         │
│                                                 │
│  ┌────────────────┐      ┌──────────────────┐  │
│  │  MCP Server    │      │  Redis Cache     │  │
│  │  Container     │◄────►│  Container       │  │
│  └────────┬───────┘      └──────────────────┘  │
│           │                                     │
│           │              ┌──────────────────┐  │
│           └─────────────►│  Odoo Container  │  │
│                          └──────────────────┘  │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Use Case**: Cloud deployment, containerized apps

**Pros**:
- Easy deployment
- Consistent environment
- Scalable (Kubernetes, etc.)
- Portable

**Cons**:
- Docker knowledge required
- Resource overhead

---

## Performance Characteristics

### Throughput

| Operation | Requests/Second | Notes |
|-----------|----------------|-------|
| `search_records` (cached) | 1000+ | In-memory cache |
| `search_records` (uncached) | 50-100 | Depends on Odoo |
| `get_record` | 200-500 | Single record lookup |
| `create_record` | 50-100 | Database writes |
| `batch_operation` | 10-50 | Bulk operations |

### Latency

| Component | Latency | Notes |
|-----------|---------|-------|
| MCP protocol overhead | <1ms | FastMCP processing |
| Security validation | <5ms | Pydantic validation |
| Zenoo RPC processing | <10ms | Query building |
| Odoo RPC call | 10-500ms | Network + Odoo processing |
| Total (typical) | 20-600ms | End-to-end |

### Optimization Strategies

1. **Caching**:
   ```python
   # Enable Redis caching
   await client.setup_cache_manager(
       backend="redis",
       url="redis://localhost:6379/0"
   )
   ```

2. **Batch Operations**:
   ```python
   # Process multiple records in single transaction
   async with client.batch() as batch:
       records = await batch.create_many(Model, data_list)
   ```

3. **Field Selection**:
   ```python
   # Only fetch needed fields
   records = await query.values('id', 'name', 'email').all()
   ```

4. **Connection Pooling**:
   ```python
   # Reuse HTTP connections
   client = ZenooClient(url, pool_limits=(10, 100))
   ```

---

## Security Model

### Authentication Methods

```
┌──────────────────────────────────────────┐
│        Authentication Flow               │
│                                          │
│  Request → Identify Method → Validate    │
│                                          │
│  ┌──────────────┐  ┌──────────────┐     │
│  │  API Key     │  │   OAuth      │     │
│  │  Header      │  │   Token      │     │
│  └──────┬───────┘  └──────┬───────┘     │
│         │                 │             │
│         └────────┬────────┘             │
│                  │                      │
│           ┌──────▼──────┐               │
│           │  Validate   │               │
│           │  & Extract  │               │
│           │  Identity   │               │
│           └──────┬──────┘               │
│                  │                      │
│           ┌──────▼──────┐               │
│           │  Session    │               │
│           │  Manager    │               │
│           └──────┬──────┘               │
│                  │                      │
│           ┌──────▼──────┐               │
│           │  Authorized │               │
│           │  Request    │               │
│           └─────────────┘               │
└──────────────────────────────────────────┘
```

### Authorization Layers

1. **MCP Server Level**: API key validation
2. **Odoo Session Level**: User authentication
3. **Odoo Model Level**: Access rights (ACL)
4. **Odoo Record Level**: Record rules

### Security Features

| Feature | Implementation | Purpose |
|---------|---------------|---------|
| **Rate Limiting** | Token bucket algorithm | Prevent DoS |
| **Input Validation** | Pydantic schemas | Prevent injection |
| **Session Management** | TTL-based tokens | Secure access |
| **Audit Logging** | Structured logs | Compliance |
| **TLS/SSL** | HTTPS transport | Encrypted transit |

---

## Related Documentation

- **[Installation & Setup](./02-installation-setup.md)** - Get started with MCP server
- **[Configuration Guide](./03-configuration.md)** - Configure the server
- **[Security Guide](./08-security-authentication.md)** - Security best practices
- **[Tools Reference](./05-tools-reference.md)** - Available MCP tools
- **[Python Library](../user-guide/README.md)** - Zenoo RPC library docs

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team
