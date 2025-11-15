---
title: "Extending the MCP Server"
version: "1.0"
last_updated: "2025-11-15"
status: "Active"
audience: "Developers"
---

# Extending the MCP Server

## Overview

Learn how to extend the Zenoo RPC MCP Server with custom functionality, tools, and integrations.

---

## Adding Custom Tools

### Register a Custom Tool

**File**: `custom_tools.py`

```python
from zenoo_rpc.mcp_server.server import ZenooMCPServer
from typing import Dict, Any

def register_custom_tools(server: ZenooMCPServer):
    """Register custom MCP tools."""

    @server.mcp_server.tool()
    async def calculate_customer_lifetime_value(
        customer_id: int
    ) -> Dict[str, Any]:
        """Calculate customer lifetime value (CLV).

        Args:
            customer_id: Partner ID

        Returns:
            CLV metrics and predictions
        """
        # Get all orders for customer
        orders = await server.zenoo_client.search_read(
            "sale.order",
            domain=[
                ["partner_id", "=", customer_id],
                ["state", "=", "sale"]
            ],
            fields=["amount_total", "date_order"]
        )

        # Calculate metrics
        total_revenue = sum(o["amount_total"] for o in orders)
        order_count = len(orders)
        avg_order_value = total_revenue / order_count if order_count > 0 else 0

        # Simple CLV calculation (can be enhanced with ML)
        clv = total_revenue * 1.5  # Assume 50% future value

        return {
            "customer_id": customer_id,
            "total_revenue": total_revenue,
            "order_count": order_count,
            "average_order_value": avg_order_value,
            "lifetime_value_estimate": clv,
            "orders": orders
        }

    @server.mcp_server.tool()
    async def product_recommendation(
        customer_id: int,
        limit: int = 5
    ) -> Dict[str, Any]:
        """Get product recommendations for customer.

        Args:
            customer_id: Partner ID
            limit: Number of recommendations

        Returns:
            Recommended products
        """
        # Get customer's purchase history
        order_lines = await server.zenoo_client.search_read(
            "sale.order.line",
            domain=[["order_id.partner_id", "=", customer_id]],
            fields=["product_id"]
        )

        purchased_products = {line["product_id"][0] for line in order_lines}

        # Find products often bought together (simplified)
        # In production, use ML-based recommendations
        recommendations = await server.zenoo_client.search_read(
            "product.product",
            domain=[
                ["id", "not in", list(purchased_products)],
                ["sale_ok", "=", True]
            ],
            limit=limit,
            fields=["name", "list_price", "categ_id"]
        )

        return {
            "customer_id": customer_id,
            "recommendations": recommendations,
            "count": len(recommendations)
        }
```

### Load Custom Tools on Startup

**File**: `server_with_extensions.py`

```python
from zenoo_rpc.mcp_server.server import ZenooMCPServer
from zenoo_rpc.mcp_server.config import MCPServerConfig
from custom_tools import register_custom_tools

async def start_extended_server():
    """Start MCP server with custom tools."""
    config = MCPServerConfig.from_env()
    server = ZenooMCPServer(config)

    # Register custom tools
    register_custom_tools(server)

    # Start server
    await server.start()

if __name__ == "__main__":
    import asyncio
    asyncio.run(start_extended_server())
```

---

## Adding Custom Resources

```python
def register_custom_resources(server: ZenooMCPServer):
    """Register custom MCP resources."""

    @server.mcp_server.resource("odoo://analytics/dashboard")
    async def analytics_dashboard() -> str:
        """Get analytics dashboard data."""
        import json

        # Aggregate various metrics
        metrics = {
            "sales_today": await get_sales_today(server),
            "open_orders": await get_open_orders_count(server),
            "low_stock_items": await get_low_stock_count(server),
            "pending_invoices": await get_pending_invoices(server)
        }

        return json.dumps(metrics)

    @server.mcp_server.resource("odoo://reports/{report_name}")
    async def custom_report(report_name: str) -> str:
        """Generate custom report."""
        # Implement custom report generation
        pass
```

---

## Middleware Pattern

### Adding Request Logging

```python
from functools import wraps
import time
import logging

logger = logging.getLogger(__name__)

def log_tool_execution(func):
    """Decorator to log tool execution."""
    @wraps(func)
    async def wrapper(*args, **kwargs):
        start_time = time.time()
        tool_name = func.__name__

        logger.info(f"Tool '{tool_name}' started")

        try:
            result = await func(*args, **kwargs)
            duration = time.time() - start_time
            logger.info(f"Tool '{tool_name}' completed in {duration:.2f}s")
            return result

        except Exception as e:
            duration = time.time() - start_time
            logger.error(
                f"Tool '{tool_name}' failed after {duration:.2f}s: {e}"
            )
            raise

    return wrapper

# Apply to tools
@log_tool_execution
async def my_custom_tool(...):
    pass
```

---

## Plugin System

### Plugin Interface

```python
from abc import ABC, abstractmethod
from typing import Any, Dict

class MCPPlugin(ABC):
    """Base class for MCP server plugins."""

    @abstractmethod
    def get_name(self) -> str:
        """Get plugin name."""
        pass

    @abstractmethod
    async def initialize(self, server: ZenooMCPServer):
        """Initialize plugin."""
        pass

    @abstractmethod
    async def cleanup(self):
        """Cleanup plugin resources."""
        pass

# Example plugin
class AnalyticsPlugin(MCPPlugin):
    """Analytics enhancement plugin."""

    def get_name(self) -> str:
        return "analytics"

    async def initialize(self, server: ZenooMCPServer):
        """Register analytics tools."""
        self.server = server

        @server.mcp_server.tool()
        async def advanced_analytics(**kwargs):
            return await self.run_analytics(kwargs)

    async def run_analytics(self, params: Dict[str, Any]):
        """Run analytics query."""
        # Implementation
        pass

    async def cleanup(self):
        """Cleanup resources."""
        pass

# Load plugins
async def load_plugins(server: ZenooMCPServer, plugins: list[MCPPlugin]):
    """Load all plugins."""
    for plugin in plugins:
        await plugin.initialize(server)
        logger.info(f"Loaded plugin: {plugin.get_name()}")
```

---

## Event Hooks

### Implementing Hooks

```python
from typing import Callable, List

class EventHooks:
    """Event hook system for MCP server."""

    def __init__(self):
        self._hooks: Dict[str, List[Callable]] = {}

    def register(self, event: str, handler: Callable):
        """Register event handler."""
        if event not in self._hooks:
            self._hooks[event] = []
        self._hooks[event].append(handler)

    async def trigger(self, event: str, *args, **kwargs):
        """Trigger event handlers."""
        if event in self._hooks:
            for handler in self._hooks[event]:
                await handler(*args, **kwargs)

# Usage
hooks = EventHooks()

# Register hooks
@hooks.register("tool_executed")
async def on_tool_executed(tool_name: str, result: Any):
    logger.info(f"Tool {tool_name} executed successfully")

@hooks.register("tool_failed")
async def on_tool_failed(tool_name: str, error: Exception):
    logger.error(f"Tool {tool_name} failed: {error}")

# Trigger hooks
await hooks.trigger("tool_executed", "search_records", result)
```

---

## Related Documentation

- **[Custom Clients](./custom-clients.md)** - Building MCP clients
- **[Custom Tools](./custom-tools.md)** - Detailed tool development
- **[Configuration](../03-configuration.md)** - Server configuration

---

**Last Updated**: 2025-11-15
**Maintained By**: Zenoo RPC Development Team
