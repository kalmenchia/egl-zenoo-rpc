---
title: "Building Custom MCP Clients"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "Developers"
---

# Building Custom MCP Clients

## Overview

Learn how to build custom MCP clients that connect to the Zenoo RPC MCP Server for specialized integrations.

---

## MCP Protocol Basics

### Protocol Structure

```
Client                          Server
  │                               │
  ├─── Initialize Request ───────>│
  │<─── Initialize Response ──────┤
  │                               │
  ├─── Tool Call Request ────────>│
  │<─── Tool Call Response ───────┤
  │                               │
  ├─── Resource Request ─────────>│
  │<─── Resource Response ────────┤
```

### Message Format

**Tool Call Request**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search_records",
    "arguments": {
      "model": "res.partner",
      "limit": 10
    }
  }
}
```

**Tool Call Response**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "records": [...],
    "count": 10
  }
}
```

---

## Python MCP Client

### Basic Client Implementation

```python
import asyncio
import json
from typing import Any, Dict
import httpx

class ZenooMCPClient:
    """Custom MCP client for Zenoo RPC server."""

    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url
        self.api_key = api_key
        self.client = httpx.AsyncClient(
            base_url=base_url,
            headers={"Authorization": f"Bearer {api_key}"}
        )
        self._request_id = 0

    async def call_tool(
        self,
        tool_name: str,
        arguments: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Call an MCP tool."""
        self._request_id += 1

        request = {
            "jsonrpc": "2.0",
            "id": self._request_id,
            "method": "tools/call",
            "params": {
                "name": tool_name,
                "arguments": arguments
            }
        }

        response = await self.client.post("/mcp", json=request)
        response.raise_for_status()

        result = response.json()
        if "error" in result:
            raise Exception(result["error"])

        return result["result"]

    async def search_records(
        self,
        model: str,
        domain: list = None,
        limit: int = 100
    ) -> Dict[str, Any]:
        """Search Odoo records."""
        return await self.call_tool("search_records", {
            "model": model,
            "domain": domain or [],
            "limit": limit
        })

    async def close(self):
        """Close the client."""
        await self.client.aclose()

# Usage
async def main():
    client = ZenooMCPClient(
        base_url="http://localhost:8000",
        api_key="your-api-key"
    )

    try:
        # Search partners
        result = await client.search_records(
            model="res.partner",
            domain=[["is_company", "=", True]],
            limit=10
        )

        print(f"Found {result['count']} companies")
        for record in result['records']:
            print(f"- {record['name']}")

    finally:
        await client.close()

if __name__ == "__main__":
    asyncio.run(main())
```

---

## JavaScript/TypeScript Client

### TypeScript Implementation

```typescript
interface MCPRequest {
  jsonrpc: "2.0";
  id: number;
  method: string;
  params: {
    name: string;
    arguments: Record<string, any>;
  };
}

interface MCPResponse {
  jsonrpc: "2.0";
  id: number;
  result?: any;
  error?: {
    code: number;
    message: string;
  };
}

class ZenooMCPClient {
  private baseUrl: string;
  private apiKey: string;
  private requestId: number = 0;

  constructor(baseUrl: string, apiKey: string) {
    this.baseUrl = baseUrl;
    this.apiKey = apiKey;
  }

  async callTool(
    toolName: string,
    args: Record<string, any>
  ): Promise<any> {
    this.requestId++;

    const request: MCPRequest = {
      jsonrpc: "2.0",
      id: this.requestId,
      method: "tools/call",
      params: {
        name: toolName,
        arguments: args,
      },
    };

    const response = await fetch(`${this.baseUrl}/mcp`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${this.apiKey}`,
      },
      body: JSON.stringify(request),
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const result: MCPResponse = await response.json();

    if (result.error) {
      throw new Error(result.error.message);
    }

    return result.result;
  }

  async searchRecords(
    model: string,
    domain: any[] = [],
    limit: number = 100
  ): Promise<any> {
    return this.callTool("search_records", {
      model,
      domain,
      limit,
    });
  }
}

// Usage
async function main() {
  const client = new ZenooMCPClient(
    "http://localhost:8000",
    "your-api-key"
  );

  const result = await client.searchRecords(
    "res.partner",
    [["is_company", "=", true]],
    10
  );

  console.log(`Found ${result.count} companies`);
  result.records.forEach((record: any) => {
    console.log(`- ${record.name}`);
  });
}

main();
```

---

## Authentication

### API Key Authentication

```python
# Include in headers
headers = {
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json"
}
```

### Session-Based Authentication

```python
async def authenticate(username: str, password: str) -> str:
    """Authenticate and get session token."""
    response = await client.post("/auth/login", json={
        "username": username,
        "password": password
    })

    result = response.json()
    return result["token"]

# Use token in subsequent requests
token = await authenticate("user", "password")
headers = {"Authorization": f"Bearer {token}"}
```

---

## Error Handling

### Robust Error Handling

```python
from typing import Optional

class MCPError(Exception):
    """Base MCP error."""
    pass

class MCPConnectionError(MCPError):
    """Connection error."""
    pass

class MCPAuthenticationError(MCPError):
    """Authentication error."""
    pass

class MCPToolError(MCPError):
    """Tool execution error."""
    pass

async def call_tool_safe(
    client: ZenooMCPClient,
    tool_name: str,
    arguments: Dict[str, Any],
    retries: int = 3
) -> Optional[Dict[str, Any]]:
    """Call tool with retry logic."""

    for attempt in range(retries):
        try:
            return await client.call_tool(tool_name, arguments)

        except httpx.HTTPStatusError as e:
            if e.response.status_code == 401:
                raise MCPAuthenticationError("Invalid API key")
            elif e.response.status_code == 429:
                # Rate limited, wait and retry
                wait_time = 2 ** attempt
                await asyncio.sleep(wait_time)
                continue
            else:
                raise MCPToolError(f"Tool failed: {e}")

        except httpx.ConnectError:
            if attempt < retries - 1:
                await asyncio.sleep(1)
                continue
            raise MCPConnectionError("Cannot connect to server")

    return None
```

---

## Best Practices

### 1. Connection Pooling

```python
# Reuse HTTP client
client = httpx.AsyncClient(
    base_url=base_url,
    limits=httpx.Limits(
        max_keepalive_connections=5,
        max_connections=10
    )
)
```

### 2. Request Timeout

```python
response = await client.post(
    "/mcp",
    json=request,
    timeout=30.0  # 30 seconds
)
```

### 3. Rate Limiting

```python
from asyncio import Semaphore

# Limit concurrent requests
semaphore = Semaphore(10)

async def call_tool_limited(...):
    async with semaphore:
        return await client.call_tool(...)
```

---

## Related Documentation

- **[Tools Reference](../05-tools-reference.md)** - Available tools
- **[Security Guide](../08-security-authentication.md)** - Authentication methods
- **[API Reference](../../api-reference/)** - Core library API

---

**Last Updated**: 2025-11-15
**Maintained By**: Zenoo RPC Development Team
