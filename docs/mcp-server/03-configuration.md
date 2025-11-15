---
title: "MCP Server Configuration Guide"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "Administrators, Developers"
---

# MCP Server Configuration Guide

## Overview

This guide covers all configuration options for the Zenoo RPC MCP Server. Learn how to configure the server for different environments, optimize performance, and customize behavior.

**Topics Covered**:
- Configuration methods (environment variables, files, CLI)
- Server settings
- Odoo connection configuration
- Security settings
- Performance tuning
- Feature toggles

---

## Table of Contents

1. [Configuration Methods](#configuration-methods)
2. [Configuration Sources Priority](#configuration-sources-priority)
3. [Server Configuration](#server-configuration)
4. [Odoo Connection](#odoo-connection)
5. [Security Configuration](#security-configuration)
6. [Performance Settings](#performance-settings)
7. [Feature Configuration](#feature-configuration)
8. [Transport Configuration](#transport-configuration)
9. [Logging Configuration](#logging-configuration)
10. [Complete Configuration Example](#complete-configuration-example)

---

## Configuration Methods

The MCP server supports three configuration methods that can be combined:

### Method 1: Environment Variables

Set environment variables in your shell or `.env` file:

```bash
# Odoo Connection
export ODOO_URL=http://localhost:8069
export ODOO_DATABASE=demo
export ODOO_USERNAME=admin
export ODOO_PASSWORD=admin

# Server Settings
export MCP_SERVER_NAME=zenoo-mcp-server
export MCP_TRANSPORT_TYPE=stdio
export MCP_LOG_LEVEL=INFO
```

**Pros**:
- Simple and straightforward
- Works well with Docker/containers
- Easy to manage in CI/CD

**Cons**:
- Can clutter environment
- Hard to version control sensitive data

### Method 2: Configuration File

Create a JSON configuration file:

```bash
# Create config file
cat > mcp-config.json << 'EOF'
{
  "name": "my-odoo-server",
  "odoo_url": "http://localhost:8069",
  "odoo_database": "production",
  "odoo_username": "admin",
  "odoo_password": "admin"
}
EOF

# Start server with config file
python -m zenoo_rpc.mcp_server.cli --config mcp-config.json
```

**Pros**:
- Organized and structured
- Easy to version control (excluding secrets)
- Supports complex configurations

**Cons**:
- Requires file management
- Secrets in file need special handling

### Method 3: CLI Arguments

Pass configuration directly via command line:

```bash
python -m zenoo_rpc.mcp_server.cli \
  --odoo-url http://localhost:8069 \
  --odoo-database demo \
  --odoo-username admin \
  --odoo-password admin \
  --transport stdio \
  --log-level INFO
```

**Pros**:
- Explicit and visible
- No file or env management
- Good for testing

**Cons**:
- Verbose command lines
- Secrets visible in process list
- Hard to maintain

---

## Configuration Sources Priority

When multiple configuration sources are present, they are applied in this priority order (highest to lowest):

```
1. CLI Arguments (highest priority)
   ↓
2. Configuration File
   ↓
3. Environment Variables
   ↓
4. Default Values (lowest priority)
```

**Example**:
```bash
# Environment variable
export ODOO_DATABASE=development

# Config file has: "odoo_database": "staging"

# CLI argument
python -m zenoo_rpc.mcp_server.cli \
  --config config.json \
  --odoo-database production

# Result: Uses "production" (CLI argument wins)
```

---

## Server Configuration

### Basic Server Settings

**Configuration File** (`mcp-config.json`):
```json
{
  "name": "zenoo-mcp-server",
  "version": "1.0",
  "description": "MCP Server for Odoo Integration",
  "transport_type": "stdio",
  "host": "localhost",
  "port": 8000
}
```

**Environment Variables**:
```bash
export MCP_SERVER_NAME=zenoo-mcp-server
export MCP_SERVER_VERSION=1.0
export MCP_TRANSPORT_TYPE=stdio
export MCP_HOST=localhost
export MCP_PORT=8000
```

**CLI Arguments**:
```bash
python -m zenoo_rpc.mcp_server.cli \
  --name zenoo-mcp-server \
  --transport stdio \
  --host localhost \
  --port 8000
```

### Server Settings Reference

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `name` | string | `zenoo-mcp-server` | Server name shown in MCP clients |
| `version` | string | `1.0` | Server version |
| `description` | string | Auto-generated | Server description |
| `transport_type` | enum | `stdio` | Transport: `stdio`, `http`, `sse` |
| `host` | string | `localhost` | Host for HTTP/SSE transport |
| `port` | integer | `8000` | Port for HTTP/SSE transport |

---

## Odoo Connection

### Basic Odoo Connection

**Configuration File**:
```json
{
  "odoo_url": "http://localhost:8069",
  "odoo_database": "demo",
  "odoo_username": "admin",
  "odoo_password": "admin"
}
```

**Environment Variables**:
```bash
export ODOO_URL=http://localhost:8069
export ODOO_DATABASE=demo
export ODOO_USERNAME=admin
export ODOO_PASSWORD=admin
```

### Odoo Connection Settings

| Setting | Type | Required | Description |
|---------|------|----------|-------------|
| `odoo_url` | string | ✅ Yes | Odoo server URL (include protocol) |
| `odoo_database` | string | ✅ Yes | Odoo database name |
| `odoo_username` | string | ✅ Yes | Odoo username |
| `odoo_password` | string | ✅ Yes | Odoo password |

### Connection Pool Settings

For high-performance scenarios, configure connection pooling:

```json
{
  "odoo_url": "http://localhost:8069",
  "connection_pool": {
    "min_connections": 5,
    "max_connections": 20,
    "connection_timeout": 30,
    "idle_timeout": 300
  }
}
```

**Environment Variables**:
```bash
export ODOO_POOL_MIN_CONNECTIONS=5
export ODOO_POOL_MAX_CONNECTIONS=20
export ODOO_POOL_CONNECTION_TIMEOUT=30
export ODOO_POOL_IDLE_TIMEOUT=300
```

---

## Security Configuration

### Authentication Methods

**Configuration File**:
```json
{
  "security": {
    "auth_method": "api_key",
    "api_keys": [
      "your-secret-key-1",
      "your-secret-key-2"
    ],
    "session_timeout": 3600,
    "require_https": true
  }
}
```

**Environment Variables**:
```bash
export MCP_AUTH_METHOD=api_key
export MCP_API_KEYS=key1,key2,key3
export MCP_SESSION_TIMEOUT=3600
export MCP_REQUIRE_HTTPS=true
```

### Security Settings Reference

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `auth_method` | enum | `api_key` | Auth method: `api_key`, `oauth`, `session` |
| `api_keys` | array | `[]` | List of valid API keys |
| `session_timeout` | integer | `3600` | Session timeout in seconds |
| `require_https` | boolean | `false` | Require HTTPS for HTTP transport |

### Rate Limiting

**Configuration File**:
```json
{
  "security": {
    "rate_limit_enabled": true,
    "rate_limit_requests": 100,
    "rate_limit_window": 60,
    "rate_limit_by": "api_key"
  }
}
```

**Environment Variables**:
```bash
export MCP_RATE_LIMIT_ENABLED=true
export MCP_RATE_LIMIT_REQUESTS=100
export MCP_RATE_LIMIT_WINDOW=60
export MCP_RATE_LIMIT_BY=api_key
```

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `rate_limit_enabled` | boolean | `true` | Enable rate limiting |
| `rate_limit_requests` | integer | `100` | Max requests per window |
| `rate_limit_window` | integer | `60` | Time window in seconds |
| `rate_limit_by` | enum | `api_key` | Rate limit by: `api_key`, `ip`, `user` |

### Input Validation

**Configuration File**:
```json
{
  "security": {
    "validate_input": true,
    "max_record_limit": 1000,
    "max_batch_size": 500,
    "allowed_models": ["res.partner", "sale.order"],
    "blocked_models": ["ir.config_parameter"]
  }
}
```

---

## Performance Settings

### Caching Configuration

**Configuration File**:
```json
{
  "performance": {
    "enable_caching": true,
    "cache_backend": "redis",
    "cache_config": {
      "url": "redis://localhost:6379/0",
      "ttl": 300,
      "max_size": 1000
    }
  }
}
```

**Environment Variables**:
```bash
export MCP_ENABLE_CACHING=true
export MCP_CACHE_BACKEND=redis
export REDIS_URL=redis://localhost:6379/0
export MCP_CACHE_TTL=300
export MCP_CACHE_MAX_SIZE=1000
```

### Caching Backends

| Backend | Use Case | Configuration |
|---------|----------|---------------|
| `memory` | Development, single instance | In-memory cache (default) |
| `redis` | Production, multiple instances | Requires Redis server |
| `memcached` | Production, distributed | Requires Memcached |

### Performance Tuning

**Configuration File**:
```json
{
  "performance": {
    "max_concurrent_requests": 50,
    "request_timeout": 30,
    "enable_compression": true,
    "compression_threshold": 1024
  }
}
```

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `max_concurrent_requests` | integer | `50` | Max parallel requests |
| `request_timeout` | integer | `30` | Request timeout (seconds) |
| `enable_compression` | boolean | `true` | Enable response compression |
| `compression_threshold` | integer | `1024` | Min bytes to compress |

---

## Feature Configuration

Control which MCP capabilities are enabled:

**Configuration File**:
```json
{
  "features": {
    "enable_tools": true,
    "enable_resources": true,
    "enable_prompts": true,
    "enable_transactions": true,
    "enable_batch_operations": true,
    "enable_analytics": true
  }
}
```

**Environment Variables**:
```bash
export MCP_ENABLE_TOOLS=true
export MCP_ENABLE_RESOURCES=true
export MCP_ENABLE_PROMPTS=true
export MCP_ENABLE_TRANSACTIONS=true
export MCP_ENABLE_BATCH_OPERATIONS=true
export MCP_ENABLE_ANALYTICS=true
```

### Feature Flags Reference

| Feature | Default | Description |
|---------|---------|-------------|
| `enable_tools` | `true` | Enable MCP tools (search, create, etc.) |
| `enable_resources` | `true` | Enable MCP resources |
| `enable_prompts` | `true` | Enable MCP prompts |
| `enable_transactions` | `true` | Enable transaction support |
| `enable_batch_operations` | `true` | Enable batch operations |
| `enable_analytics` | `true` | Enable analytics queries |

---

## Transport Configuration

### stdio Transport (Default)

Best for Claude Desktop and local tools.

**Configuration File**:
```json
{
  "transport_type": "stdio"
}
```

**Environment Variables**:
```bash
export MCP_TRANSPORT_TYPE=stdio
```

**Usage**: No additional configuration needed. Server communicates via stdin/stdout.

### HTTP Transport

For web applications and remote access.

**Configuration File**:
```json
{
  "transport_type": "http",
  "host": "0.0.0.0",
  "port": 8000,
  "http_config": {
    "cors_enabled": true,
    "cors_origins": ["http://localhost:3000"],
    "ssl_enabled": false,
    "ssl_cert": "/path/to/cert.pem",
    "ssl_key": "/path/to/key.pem"
  }
}
```

**Environment Variables**:
```bash
export MCP_TRANSPORT_TYPE=http
export MCP_HOST=0.0.0.0
export MCP_PORT=8000
export MCP_CORS_ENABLED=true
export MCP_CORS_ORIGINS=http://localhost:3000,http://app.example.com
export MCP_SSL_ENABLED=false
```

### SSE Transport (Server-Sent Events)

For streaming and real-time updates.

**Configuration File**:
```json
{
  "transport_type": "sse",
  "host": "0.0.0.0",
  "port": 8000,
  "sse_config": {
    "keepalive_interval": 30,
    "retry_timeout": 3000
  }
}
```

---

## Logging Configuration

### Log Levels

**Configuration File**:
```json
{
  "log_level": "INFO",
  "log_format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s",
  "log_file": "/var/log/mcp-server.log",
  "log_rotation": {
    "enabled": true,
    "max_bytes": 10485760,
    "backup_count": 5
  }
}
```

**Environment Variables**:
```bash
export MCP_LOG_LEVEL=INFO
export MCP_LOG_FILE=/var/log/mcp-server.log
export MCP_LOG_ROTATION_ENABLED=true
export MCP_LOG_MAX_BYTES=10485760
export MCP_LOG_BACKUP_COUNT=5
```

### Log Levels

| Level | Use Case | What Gets Logged |
|-------|----------|------------------|
| `DEBUG` | Development | Everything including debug info |
| `INFO` | Production (default) | Normal operations and events |
| `WARNING` | Production (quiet) | Warnings and errors only |
| `ERROR` | Production (minimal) | Errors only |

### Structured Logging

For JSON-formatted logs:

```json
{
  "log_format": "json",
  "log_fields": [
    "timestamp",
    "level",
    "message",
    "user",
    "tool",
    "duration"
  ]
}
```

---

## Complete Configuration Example

### Production Configuration

**File**: `production-config.json`

```json
{
  "name": "production-odoo-mcp",
  "version": "1.0",
  "description": "Production MCP Server for Odoo",

  "transport_type": "http",
  "host": "0.0.0.0",
  "port": 8000,

  "odoo_url": "https://odoo.example.com",
  "odoo_database": "production",
  "odoo_username": "mcp_service",
  "odoo_password": "${ODOO_PASSWORD}",

  "security": {
    "auth_method": "api_key",
    "api_keys": ["${API_KEY_1}", "${API_KEY_2}"],
    "session_timeout": 7200,
    "require_https": true,
    "rate_limit_enabled": true,
    "rate_limit_requests": 1000,
    "rate_limit_window": 60,
    "validate_input": true,
    "max_record_limit": 1000,
    "max_batch_size": 500
  },

  "performance": {
    "enable_caching": true,
    "cache_backend": "redis",
    "cache_config": {
      "url": "redis://redis:6379/0",
      "ttl": 600,
      "max_size": 10000
    },
    "max_concurrent_requests": 100,
    "request_timeout": 60,
    "enable_compression": true
  },

  "features": {
    "enable_tools": true,
    "enable_resources": true,
    "enable_prompts": true,
    "enable_transactions": true,
    "enable_batch_operations": true,
    "enable_analytics": true
  },

  "http_config": {
    "cors_enabled": true,
    "cors_origins": ["https://app.example.com"],
    "ssl_enabled": true,
    "ssl_cert": "/etc/ssl/certs/server.crt",
    "ssl_key": "/etc/ssl/private/server.key"
  },

  "log_level": "INFO",
  "log_format": "json",
  "log_file": "/var/log/mcp-server/server.log",
  "log_rotation": {
    "enabled": true,
    "max_bytes": 104857600,
    "backup_count": 10
  }
}
```

### Development Configuration

**File**: `development-config.json`

```json
{
  "name": "dev-odoo-mcp",
  "transport_type": "stdio",

  "odoo_url": "http://localhost:8069",
  "odoo_database": "dev",
  "odoo_username": "admin",
  "odoo_password": "admin",

  "security": {
    "auth_method": "api_key",
    "rate_limit_enabled": false,
    "validate_input": false
  },

  "performance": {
    "enable_caching": false,
    "max_concurrent_requests": 10
  },

  "features": {
    "enable_tools": true,
    "enable_resources": true,
    "enable_prompts": true,
    "enable_transactions": true,
    "enable_batch_operations": true,
    "enable_analytics": true
  },

  "log_level": "DEBUG",
  "log_format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
}
```

---

## Environment-Specific Configurations

### Docker Configuration

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  mcp-server:
    image: zenoo-mcp-server:latest
    environment:
      - ODOO_URL=http://odoo:8069
      - ODOO_DATABASE=${DB_NAME}
      - ODOO_USERNAME=${DB_USER}
      - ODOO_PASSWORD=${DB_PASSWORD}
      - MCP_TRANSPORT_TYPE=http
      - MCP_HOST=0.0.0.0
      - MCP_PORT=8000
      - REDIS_URL=redis://redis:6379/0
      - MCP_ENABLE_CACHING=true
      - MCP_LOG_LEVEL=INFO
    ports:
      - "8000:8000"
    depends_on:
      - redis
      - odoo
```

### Kubernetes Configuration

**ConfigMap**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mcp-server-config
data:
  config.json: |
    {
      "transport_type": "http",
      "host": "0.0.0.0",
      "port": 8000,
      "performance": {
        "enable_caching": true,
        "cache_backend": "redis",
        "cache_config": {
          "url": "redis://redis-service:6379/0"
        }
      }
    }
```

---

## Configuration Validation

Validate your configuration before starting:

```bash
# Validate configuration file
python -m zenoo_rpc.mcp_server.cli \
  --config production-config.json \
  --validate-config

# Output if valid:
# ✅ Configuration is valid
# {
#   "name": "production-odoo-mcp",
#   "transport_type": "http",
#   ...
# }
```

---

## Configuration Best Practices

### 1. Security

✅ **DO**:
- Use environment variables for secrets
- Rotate API keys regularly
- Enable rate limiting in production
- Use HTTPS for HTTP transport
- Set appropriate session timeouts

❌ **DON'T**:
- Commit passwords to version control
- Use default passwords in production
- Disable input validation
- Allow unlimited requests

### 2. Performance

✅ **DO**:
- Enable caching for production
- Use Redis for multi-instance deployments
- Set appropriate connection pools
- Configure reasonable timeouts

❌ **DON'T**:
- Set unlimited concurrent requests
- Use tiny cache sizes
- Disable compression

### 3. Logging

✅ **DO**:
- Use INFO level for production
- Enable log rotation
- Use structured logging (JSON)
- Monitor log files

❌ **DON'T**:
- Use DEBUG in production
- Log passwords or API keys
- Ignore disk space for logs

---

## Troubleshooting Configuration

### Issue: Configuration Not Loading

**Check Priority**:
```bash
# See which config is actually used
python -m zenoo_rpc.mcp_server.cli \
  --config myconfig.json \
  --validate-config
```

### Issue: Invalid JSON

**Validate JSON**:
```bash
python -m json.tool myconfig.json
```

### Issue: Environment Variables Not Working

**Check Variables**:
```bash
# List all MCP-related env vars
env | grep MCP

# Test with explicit vars
MCP_LOG_LEVEL=DEBUG python -m zenoo_rpc.mcp_server.cli
```

---

## Related Documentation

- **[Installation & Setup](./02-installation-setup.md)** - Install the server
- **[Security & Authentication](./08-security-authentication.md)** - Security details
- **[Deployment Guide](./09-deployment.md)** - Production deployment
- **[Troubleshooting](./10-troubleshooting.md)** - Common issues

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team
