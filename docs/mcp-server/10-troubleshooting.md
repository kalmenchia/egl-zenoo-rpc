---
title: "Troubleshooting Guide"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "All"
---

# Troubleshooting Guide

## Overview

This guide helps you diagnose and resolve common issues with the Zenoo RPC MCP Server.

---

## Quick Diagnostics

### Check Server Status

```bash
# Systemd service
sudo systemctl status mcp-server

# Docker
docker ps | grep mcp-server

# Process
ps aux | grep "zenoo_rpc.mcp_server"
```

### Check Connectivity

```bash
# Test Odoo connection
curl http://localhost:8069/web/database/selector

# Test MCP server (if HTTP)
curl http://localhost:8000/health
```

### View Logs

```bash
# Systemd
sudo journalctl -u mcp-server -f

# Docker
docker logs -f mcp-server

# Application log
tail -f /var/log/mcp-server/server.log
```

---

## Common Issues

### Issue 1: Server Won't Start

**Symptoms**:
```
Service fails to start
No response from server
```

**Diagnostic**:
```bash
# Check logs
sudo journalctl -u mcp-server -n 50

# Validate config
python -m zenoo_rpc.mcp_server.cli --validate-config
```

**Common Causes**:

1. **Port Already in Use**
   ```bash
   # Find process using port
   sudo lsof -i :8000

   # Solution: Kill process or use different port
   export MCP_PORT=8001
   ```

2. **Invalid Configuration**
   ```bash
   # Validate JSON
   python -m json.tool config.json

   # Check required fields
   cat config.json | jq '.odoo_url, .odoo_database'
   ```

3. **Missing Dependencies**
   ```bash
   # Reinstall
   pip install --force-reinstall zenoo-rpc[mcp]
   ```

---

### Issue 2: Cannot Connect to Odoo

**Symptoms**:
```
Error: Odoo connection failed
Error: Connection refused
```

**Diagnostic**:
```bash
# Test Odoo accessibility
curl -I http://your-odoo-server:8069

# Test authentication
python << EOF
from zenoo_rpc import ZenooClient
import asyncio

async def test():
    try:
        async with ZenooClient("http://localhost:8069") as client:
            await client.login("demo", "admin", "admin")
            print("✅ Connection successful!")
    except Exception as e:
        print(f"❌ Error: {e}")

asyncio.run(test())
EOF
```

**Solutions**:

1. **Odoo Not Running**
   ```bash
   # Check Odoo status
   sudo systemctl status odoo
   ```

2. **Wrong Credentials**
   - Verify username/password
   - Check database name exists
   - Confirm user has access

3. **Network Issues**
   ```bash
   # Test network
   ping your-odoo-server
   telnet your-odoo-server 8069
   ```

4. **Firewall Blocking**
   ```bash
   # Allow Odoo port
   sudo ufw allow from your-ip to any port 8069
   ```

---

### Issue 3: Claude Desktop Not Connecting

**Symptoms**:
```
🔌 icon not appearing in Claude Desktop
Server not listed in MCP servers
```

**Diagnostic**:

1. **Check Config File Location**
   ```bash
   # macOS
   cat ~/.config/claude/claude_desktop_config.json

   # Windows
   type %APPDATA%\Claude\claude_desktop_config.json
   ```

2. **Validate JSON**
   ```bash
   python -m json.tool ~/.config/claude/claude_desktop_config.json
   ```

3. **Test Python Command**
   ```bash
   python -m zenoo_rpc.mcp_server.cli --version
   ```

**Solutions**:

1. **Fix JSON Syntax**
   - Remove trailing commas
   - Check bracket matching
   - Use JSON validator

2. **Fix Python Path**
   ```json
   {
     "command": "/full/path/to/python",
     "args": ["-m", "zenoo_rpc.mcp_server.cli"]
   }
   ```

3. **Restart Claude Desktop**
   - Completely quit (Cmd/Ctrl+Q)
   - Restart application

---

### Issue 4: Tools Not Working

**Symptoms**:
```
Tool execution failed
Permission denied
Record not found
```

**Diagnostic**:
```bash
# Enable debug logging
export MCP_LOG_LEVEL=DEBUG
python -m zenoo_rpc.mcp_server.cli
```

**Solutions**:

1. **Permission Issues**
   - Check Odoo user has access rights
   - Verify record rules allow access
   - Review security groups

2. **Invalid Model Names**
   ```
   Correct: res.partner
   Wrong: Partner, res_partner
   ```

3. **Domain Syntax Errors**
   ```python
   # Correct
   domain=[['name', '=', 'John']]

   # Wrong
   domain=['name=John']
   ```

---

### Issue 5: Slow Performance

**Symptoms**:
```
Queries take >5 seconds
Timeouts
High CPU usage
```

**Diagnostic**:
```bash
# Check system resources
htop

# Monitor Redis (if using)
redis-cli monitor

# Check Odoo performance
# Look at Odoo logs
```

**Solutions**:

1. **Enable Caching**
   ```json
   {
     "performance": {
       "enable_caching": true,
       "cache_backend": "redis"
     }
   }
   ```

2. **Optimize Queries**
   - Limit result sets
   - Select only needed fields
   - Use appropriate indexes in Odoo

3. **Scale Resources**
   - Increase server RAM
   - Add more CPU cores
   - Use Redis for caching

---

### Issue 6: Rate Limit Errors

**Symptoms**:
```
Error: Rate limit exceeded
429 Too Many Requests
```

**Solutions**:

1. **Increase Limits**
   ```json
   {
     "security": {
       "rate_limit_requests": 10000,
       "rate_limit_window": 60
     }
   }
   ```

2. **Use Multiple API Keys**
   - Distribute load across keys
   - Implement client-side rate limiting

---

## Error Messages

### Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| `Connection refused` | Server not running | Start server |
| `Authentication failed` | Wrong credentials | Check username/password |
| `Model not found` | Invalid model name | Check model exists |
| `Permission denied` | Insufficient rights | Check Odoo permissions |
| `Rate limit exceeded` | Too many requests | Increase limit or slow down |
| `Timeout` | Query too slow | Optimize query |

---

## Debug Mode

### Enable Debug Logging

```bash
# Environment variable
export MCP_LOG_LEVEL=DEBUG

# Config file
{
  "log_level": "DEBUG"
}

# CLI argument
python -m zenoo_rpc.mcp_server.cli --log-level DEBUG
```

### Useful Debug Information

```bash
# Server version
python -m zenoo_rpc.mcp_server.cli --version

# Configuration dump
python -m zenoo_rpc.mcp_server.cli --validate-config

# Environment variables
env | grep -E "(ODOO|MCP)"
```

---

## Getting Help

### Before Asking for Help

Gather this information:

1. **Error Message**: Full error text
2. **Logs**: Last 50 lines of logs
3. **Configuration**: Sanitized config (remove passwords)
4. **Environment**: OS, Python version, Zenoo RPC version
5. **Steps to Reproduce**: What triggers the error

### Where to Get Help

1. **Documentation**: Search this guide and other docs
2. **GitHub Issues**: https://github.com/tuanle96/zenoo-rpc/issues
3. **GitHub Discussions**: https://github.com/tuanle96/zenoo-rpc/discussions
4. **Stack Overflow**: Tag `zenoo-rpc`

---

## Related Documentation

- **[Installation & Setup](./02-installation-setup.md)** - Setup instructions
- **[Configuration Guide](./03-configuration.md)** - Configuration options
- **[Security Guide](./08-security-authentication.md)** - Security issues
- **[Deployment Guide](./09-deployment.md)** - Deployment problems

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team
