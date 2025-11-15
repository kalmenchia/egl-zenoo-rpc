---
title: "MCP Server Installation & Setup"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "Administrators"
---

# MCP Server Installation & Setup

## Overview

This guide walks you through installing and setting up the Zenoo RPC MCP Server from scratch. By the end, you'll have a running MCP server ready to connect with Claude Desktop or other MCP clients.

**Estimated Time**: 10-15 minutes
**Difficulty**: Beginner to Intermediate

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation Methods](#installation-methods)
3. [Quick Start](#quick-start)
4. [Detailed Setup](#detailed-setup)
5. [Verification](#verification)
6. [Next Steps](#next-steps)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### System Requirements

**Minimum**:
- Python 3.8 or higher
- pip (Python package manager)
- Access to an Odoo server (local or remote)
- 512MB RAM
- 100MB disk space

**Recommended**:
- Python 3.10 or higher
- virtualenv or conda for environment isolation
- Redis server (for caching)
- 1GB+ RAM
- 500MB disk space

### Required Access

- ✅ Odoo server URL (e.g., `http://localhost:8069`)
- ✅ Odoo database name
- ✅ Odoo username and password with appropriate permissions
- ✅ Network connectivity to Odoo server

### Supported Platforms

| Platform | Supported | Notes |
|----------|-----------|-------|
| **Ubuntu/Debian** | ✅ Yes | Recommended for production |
| **macOS** | ✅ Yes | Great for development |
| **Windows** | ✅ Yes | Use WSL2 for best experience |
| **Docker** | ✅ Yes | Cross-platform compatibility |

---

## Installation Methods

Choose the installation method that best fits your needs:

| Method | Use Case | Difficulty |
|--------|----------|------------|
| [PyPI (pip)](#method-1-pypi-recommended) | Production, most users | ⭐ Easy |
| [From Source](#method-2-from-source) | Development, customization | ⭐⭐ Moderate |
| [Docker](#method-3-docker) | Containerized deployment | ⭐⭐ Moderate |

---

## Method 1: PyPI (Recommended)

### Step 1: Create Virtual Environment

```bash
# Create a new virtual environment
python3 -m venv zenoo-mcp-env

# Activate the environment
# On Linux/macOS:
source zenoo-mcp-env/bin/activate

# On Windows:
zenoo-mcp-env\Scripts\activate
```

**Why virtual environment?**
- Isolates dependencies
- Prevents conflicts with system packages
- Easy to recreate and manage

### Step 2: Install Zenoo RPC with MCP Support

```bash
# Install with MCP support (includes FastMCP dependencies)
pip install zenoo-rpc[mcp]

# Or install all features (MCP + AI + Redis caching)
pip install zenoo-rpc[mcp,ai,redis]
```

**Installation Options**:

| Package | What It Includes |
|---------|------------------|
| `zenoo-rpc` | Core library only |
| `zenoo-rpc[mcp]` | Core + MCP server |
| `zenoo-rpc[ai]` | Core + AI features (Gemini, OpenAI) |
| `zenoo-rpc[redis]` | Core + Redis caching |
| `zenoo-rpc[mcp,ai,redis]` | All features |

### Step 3: Verify Installation

```bash
# Verify zenoo-rpc is installed
python -c "import zenoo_rpc; print(zenoo_rpc.__version__)"

# Check MCP server is available
python -m zenoo_rpc.mcp_server.cli --version
```

**Expected Output**:
```
Zenoo RPC MCP Server 1.0.0
```

### Step 4: Configure Environment Variables

Create a `.env` file or set environment variables:

```bash
# Create .env file
cat > .env << 'EOF'
# Odoo Connection
ODOO_URL=http://localhost:8069
ODOO_DATABASE=demo
ODOO_USERNAME=admin
ODOO_PASSWORD=admin

# MCP Server Configuration
MCP_SERVER_NAME=zenoo-mcp-server
MCP_TRANSPORT_TYPE=stdio
MCP_LOG_LEVEL=INFO

# Security (optional)
# MCP_API_KEYS=your-secret-key-1,your-secret-key-2
EOF

# Load environment variables
source .env  # or use 'export $(cat .env | xargs)' on some systems
```

### Step 5: Start the MCP Server

```bash
# Start with default settings (stdio transport)
python -m zenoo_rpc.mcp_server.cli

# Or with explicit configuration
python -m zenoo_rpc.mcp_server.cli \
  --odoo-url http://localhost:8069 \
  --odoo-database demo \
  --odoo-username admin \
  --odoo-password admin
```

**You should see**:
```
🚀 Zenoo RPC MCP Server
==================================================
Name: zenoo-mcp-server
Version: 1.0.0
Transport: stdio
Odoo URL: http://localhost:8069
Odoo Database: demo
Auth Method: api_key
Log Level: INFO
==================================================
✅ Server initialized successfully
🔌 Starting server...
```

---

## Method 2: From Source

### Step 1: Clone Repository

```bash
# Clone the repository
git clone https://github.com/tuanle96/zenoo-rpc.git
cd zenoo-rpc
```

### Step 2: Create Virtual Environment

```bash
# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### Step 3: Install in Development Mode

```bash
# Install with all development dependencies
pip install -e ".[dev,mcp,ai,redis]"

# Install pre-commit hooks (optional, for contributors)
pre-commit install
```

### Step 4: Verify Installation

```bash
# Run tests to verify installation
pytest tests/ -v

# Check MCP server
python -m zenoo_rpc.mcp_server.cli --version
```

### Step 5: Configure and Run

```bash
# Copy example configuration
cp .env.example .env

# Edit configuration
nano .env  # or use your preferred editor

# Start server
python -m zenoo_rpc.mcp_server.cli
```

---

## Method 3: Docker

### Option A: Using Pre-built Image (Coming Soon)

```bash
# Pull the image
docker pull tuanle96/zenoo-mcp-server:latest

# Run the container
docker run -d \
  --name zenoo-mcp-server \
  -e ODOO_URL=http://odoo-server:8069 \
  -e ODOO_DATABASE=demo \
  -e ODOO_USERNAME=admin \
  -e ODOO_PASSWORD=admin \
  -e MCP_TRANSPORT_TYPE=http \
  -p 8000:8000 \
  tuanle96/zenoo-mcp-server:latest
```

### Option B: Build from Source

Create a `Dockerfile`:

```dockerfile
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Install zenoo-rpc
RUN pip install -e ".[mcp]"

# Expose HTTP port (if using HTTP transport)
EXPOSE 8000

# Set environment variables
ENV PYTHONUNBUFFERED=1

# Start MCP server
CMD ["python", "-m", "zenoo_rpc.mcp_server.cli"]
```

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  mcp-server:
    build: .
    container_name: zenoo-mcp-server
    environment:
      - ODOO_URL=http://odoo:8069
      - ODOO_DATABASE=demo
      - ODOO_USERNAME=admin
      - ODOO_PASSWORD=admin
      - MCP_TRANSPORT_TYPE=http
      - MCP_HOST=0.0.0.0
      - MCP_PORT=8000
    ports:
      - "8000:8000"
    depends_on:
      - redis
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    container_name: zenoo-redis
    restart: unless-stopped
```

Build and run:

```bash
# Build image
docker-compose build

# Start services
docker-compose up -d

# View logs
docker-compose logs -f mcp-server

# Stop services
docker-compose down
```

---

## Quick Start

### Fastest Way to Get Running

```bash
# 1. Install (one command)
pip install zenoo-rpc[mcp]

# 2. Start server (one command)
python -m zenoo_rpc.mcp_server.cli \
  --odoo-url http://localhost:8069 \
  --odoo-database demo \
  --odoo-username admin \
  --odoo-password admin

# 3. Connect Claude Desktop (see Claude Desktop Integration guide)
```

---

## Detailed Setup

### Configuration File Method

Instead of environment variables or CLI arguments, you can use a JSON configuration file:

**Create `mcp-server-config.json`**:

```json
{
  "name": "my-odoo-mcp-server",
  "version": "1.0",
  "transport_type": "stdio",
  "host": "localhost",
  "port": 8000,

  "odoo_url": "http://localhost:8069",
  "odoo_database": "production",
  "odoo_username": "admin",
  "odoo_password": "admin",

  "security": {
    "auth_method": "api_key",
    "api_keys": ["your-secret-key-here"],
    "rate_limit_requests": 100,
    "rate_limit_window": 60,
    "session_timeout": 3600
  },

  "features": {
    "enable_tools": true,
    "enable_resources": true,
    "enable_prompts": true,
    "enable_caching": true
  },

  "log_level": "INFO",
  "log_format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
}
```

**Start server with config file**:

```bash
python -m zenoo_rpc.mcp_server.cli --config mcp-server-config.json
```

### Transport Types

#### stdio (Default)
```bash
# Best for Claude Desktop and local tools
python -m zenoo_rpc.mcp_server.cli --transport stdio
```

**Use Cases**:
- Claude Desktop integration
- Local development
- Single user scenarios

**Pros**:
- Simple setup
- No network configuration
- Fast local communication

**Cons**:
- Single user only
- No remote access

#### HTTP
```bash
# For web applications and remote access
python -m zenoo_rpc.mcp_server.cli \
  --transport http \
  --host 0.0.0.0 \
  --port 8000
```

**Use Cases**:
- Production deployments
- Multi-user access
- Web application integration

**Pros**:
- Multiple concurrent users
- Remote access
- Scalable

**Cons**:
- Requires security configuration
- More complex setup

#### Server-Sent Events (SSE)
```bash
# For streaming and real-time updates
python -m zenoo_rpc.mcp_server.cli --transport sse
```

**Use Cases**:
- Real-time data streaming
- Live updates
- Event-driven applications

---

## Verification

### Test Basic Connectivity

**Method 1: Using Python**

```python
import asyncio
from zenoo_rpc import ZenooClient

async def test_connection():
    async with ZenooClient("http://localhost:8069") as client:
        await client.login("demo", "admin", "admin")

        # Test basic search
        partners = await client.search_read(
            "res.partner",
            domain=[],
            limit=1
        )

        print(f"✅ Connection successful! Found {len(partners)} record(s)")

asyncio.run(test_connection())
```

**Expected Output**:
```
✅ Connection successful! Found 1 record(s)
```

**Method 2: Validate Configuration**

```bash
# Validate configuration and exit
python -m zenoo_rpc.mcp_server.cli --validate-config

# Should output:
# ✅ Configuration is valid
# {
#   "name": "zenoo-mcp-server",
#   "transport_type": "stdio",
#   ...
# }
```

### Check Server Health

**For HTTP transport**:

```bash
# If running on HTTP, check health endpoint
curl http://localhost:8000/health

# Expected response:
# {"status": "healthy", "odoo_connected": true}
```

### Test MCP Tools

Once connected via Claude Desktop or custom client, test tools:

```
# In Claude Desktop:
User: "Can you search for Odoo partners named 'Admin'?"

Claude: [Uses search_records tool]
        "I found 1 partner named 'Admin'..."
```

---

## Next Steps

### For Claude Desktop Users

1. **Configure Claude Desktop**: Follow [Claude Desktop Integration Guide](./04-claude-desktop-integration.md)
2. **Test Natural Language Queries**: Try example queries from [Tools Reference](./05-tools-reference.md)
3. **Explore Use Cases**: Check out [Use Case Guides](./use-cases/)

### For Administrators

1. **Configure Security**: Set up [Security & Authentication](./08-security-authentication.md)
2. **Review Configuration**: Optimize [Configuration Settings](./03-configuration.md)
3. **Plan Deployment**: Read [Deployment Guide](./09-deployment.md)

### For Developers

1. **Explore API**: Review [Tools Reference](./05-tools-reference.md)
2. **Build Custom Clients**: Learn about [Custom MCP Clients](./advanced/custom-clients.md)
3. **Contribute**: See [Contributing Guide](../../contributing/development.md)

---

## Troubleshooting

### Installation Issues

#### Problem: `pip install` fails

**Error**:
```
ERROR: Could not find a version that satisfies the requirement zenoo-rpc
```

**Solutions**:
```bash
# Update pip
pip install --upgrade pip

# Install from source
git clone https://github.com/tuanle96/zenoo-rpc.git
cd zenoo-rpc
pip install -e ".[mcp]"
```

#### Problem: Python version incompatible

**Error**:
```
ERROR: Package requires Python >=3.8
```

**Solutions**:
```bash
# Check Python version
python3 --version

# Install Python 3.10+ (Ubuntu/Debian)
sudo apt update
sudo apt install python3.10 python3.10-venv

# Use Python 3.10
python3.10 -m venv venv
source venv/bin/activate
pip install zenoo-rpc[mcp]
```

### Connection Issues

#### Problem: Cannot connect to Odoo

**Error**:
```
❌ Server error: Odoo connection failed: [Errno 111] Connection refused
```

**Solutions**:

1. **Check Odoo is running**:
   ```bash
   # Test Odoo accessibility
   curl http://localhost:8069/web/database/selector
   ```

2. **Verify credentials**:
   ```bash
   # Test login via Python
   python3 << EOF
   import xmlrpc.client

   url = "http://localhost:8069"
   db = "demo"
   username = "admin"
   password = "admin"

   common = xmlrpc.client.ServerProxy(f'{url}/xmlrpc/2/common')
   uid = common.authenticate(db, username, password, {})
   print(f"✅ Login successful! UID: {uid}")
   EOF
   ```

3. **Check firewall**:
   ```bash
   # On Linux, check if port is accessible
   telnet localhost 8069
   ```

#### Problem: Permission denied

**Error**:
```
❌ Access denied
```

**Solutions**:

1. Ensure Odoo user has necessary permissions
2. Check Odoo access rights in Settings → Users
3. Verify database name is correct

### Runtime Issues

#### Problem: Server starts but doesn't respond

**Diagnostic Steps**:

```bash
# 1. Check server logs (increase verbosity)
python -m zenoo_rpc.mcp_server.cli --log-level DEBUG

# 2. Test in development mode
python -m zenoo_rpc.mcp_server.cli --dev

# 3. Validate configuration
python -m zenoo_rpc.mcp_server.cli --validate-config
```

#### Problem: High memory usage

**Solutions**:

1. **Disable caching** (if enabled):
   ```bash
   # Set in config or environment
   export MCP_ENABLE_CACHING=false
   ```

2. **Limit concurrent connections**:
   ```json
   {
     "performance": {
       "max_concurrent_requests": 10
     }
   }
   ```

### Getting More Help

If issues persist:

1. **Enable Debug Logging**:
   ```bash
   python -m zenoo_rpc.mcp_server.cli --log-level DEBUG --dev
   ```

2. **Check Logs**: Review output for detailed error messages

3. **Consult Documentation**:
   - [Troubleshooting Guide](./10-troubleshooting.md)
   - [FAQ](../troubleshooting/faq.md)

4. **Get Community Help**:
   - [GitHub Issues](https://github.com/tuanle96/zenoo-rpc/issues)
   - [GitHub Discussions](https://github.com/tuanle96/zenoo-rpc/discussions)

---

## Related Documentation

- **[Overview & Architecture](./01-overview-architecture.md)** - Understanding the system
- **[Configuration Guide](./03-configuration.md)** - Detailed configuration options
- **[Claude Desktop Integration](./04-claude-desktop-integration.md)** - Connect with Claude
- **[Security Guide](./08-security-authentication.md)** - Secure your installation
- **[Deployment Guide](./09-deployment.md)** - Production deployment

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team
