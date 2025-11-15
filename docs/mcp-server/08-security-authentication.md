---
title: "Security & Authentication Guide"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "Administrators"
---

# Security & Authentication Guide

## Overview

This guide covers security best practices, authentication methods, and hardening strategies for the Zenoo RPC MCP Server.

---

## Table of Contents

1. [Security Model](#security-model)
2. [Authentication Methods](#authentication-methods)
3. [Authorization](#authorization)
4. [Rate Limiting](#rate-limiting)
5. [Input Validation](#input-validation)
6. [Secure Configuration](#secure-configuration)
7. [Audit Logging](#audit-logging)
8. [Security Best Practices](#security-best-practices)

---

## Security Model

### Multi-Layer Security

```
┌──────────────────────────────────────┐
│  1. Transport Security (TLS/HTTPS)   │
├──────────────────────────────────────┤
│  2. Authentication (API Keys/OAuth)  │
├──────────────────────────────────────┤
│  3. Rate Limiting (DoS Prevention)   │
├──────────────────────────────────────┤
│  4. Input Validation (Pydantic)      │
├──────────────────────────────────────┤
│  5. Authorization (Odoo ACL)         │
├──────────────────────────────────────┤
│  6. Audit Logging (Compliance)       │
└──────────────────────────────────────┘
```

---

## Authentication Methods

### Method 1: API Key Authentication (Default)

**Configuration**:
```json
{
  "security": {
    "auth_method": "api_key",
    "api_keys": [
      "sk_prod_abc123xyz789",
      "sk_prod_def456uvw012"
    ]
  }
}
```

**Environment Variables**:
```bash
export MCP_AUTH_METHOD=api_key
export MCP_API_KEYS=sk_prod_abc123,sk_prod_def456
```

**Usage**:
```http
POST /mcp HTTP/1.1
Host: mcp-server.example.com
Authorization: Bearer sk_prod_abc123xyz789
Content-Type: application/json
```

**Pros**:
- Simple to implement
- Stateless
- Easy to rotate

**Cons**:
- Need secure storage
- Can be leaked if not careful

**Best Practices**:
- Use strong, random keys (min 32 characters)
- Prefix keys (e.g., `sk_prod_`, `sk_dev_`)
- Rotate keys regularly (quarterly)
- Store in environment variables, not code
- Use different keys per environment

### Method 2: OAuth 2.0

**Configuration**:
```json
{
  "security": {
    "auth_method": "oauth",
    "oauth_provider": "auth0",
    "oauth_config": {
      "client_id": "your_client_id",
      "client_secret": "${OAUTH_CLIENT_SECRET}",
      "token_url": "https://auth.example.com/oauth/token",
      "authorize_url": "https://auth.example.com/oauth/authorize"
    }
  }
}
```

**Pros**:
- Industry standard
- Fine-grained permissions
- Token expiration

**Cons**:
- More complex setup
- Requires OAuth provider

### Method 3: Session-Based

**Configuration**:
```json
{
  "security": {
    "auth_method": "session",
    "session_timeout": 3600,
    "session_storage": "redis"
  }
}
```

---

## Authorization

### Odoo-Level Authorization

MCP server uses Odoo's built-in ACL:

```
MCP Request
    ↓
Authenticate User
    ↓
Execute as Odoo User
    ↓
Odoo Checks:
  - Access Rights (model-level)
  - Record Rules (record-level)
  - Field Access
    ↓
Return Result (if authorized)
```

### Model Access Control

Create dedicated Odoo user for MCP with specific permissions:

**Recommended Groups**:
```xml
<record id="user_mcp_readonly" model="res.users">
    <field name="name">MCP Server (Read-Only)</field>
    <field name="login">mcp_readonly</field>
    <field name="groups_id" eval="[(6,0,[
        ref('base.group_user'),
        ref('sales_team.group_sale_salesman_all_leads')
    ])]"/>
</record>
```

---

## Rate Limiting

### Configuration

**Per API Key**:
```json
{
  "security": {
    "rate_limit_enabled": true,
    "rate_limit_requests": 1000,
    "rate_limit_window": 60,
    "rate_limit_by": "api_key"
  }
}
```

**Per IP Address**:
```json
{
  "security": {
    "rate_limit_by": "ip"
  }
}
```

### Rate Limit Tiers

| Tier | Requests/Minute | Use Case |
|------|-----------------|----------|
| Development | Unlimited | Testing |
| Basic | 100 | Single user |
| Standard | 1000 | Small team |
| Enterprise | 10000 | Large deployment |

---

## Input Validation

### Pydantic Validation

All inputs validated automatically:

```python
# Automatically validated by Pydantic
{
  "model": "res.partner",  # Must be string
  "limit": 100,            # Must be integer
  "domain": [[...]]        # Must be valid domain array
}
```

### Domain Validation

Prevents injection attacks:
- SQL injection: Domains validated before query
- Code injection: No arbitrary code execution
- Path traversal: Model names validated against whitelist

### Field Validation

**Allowed Models** (optional whitelist):
```json
{
  "security": {
    "allowed_models": [
      "res.partner",
      "sale.order",
      "product.product"
    ]
  }
}
```

**Blocked Models** (blacklist):
```json
{
  "security": {
    "blocked_models": [
      "ir.config_parameter",
      "res.users"
    ]
  }
}
```

---

## Secure Configuration

### Production Checklist

✅ **HTTPS/TLS**:
```json
{
  "http_config": {
    "ssl_enabled": true,
    "ssl_cert": "/etc/ssl/certs/server.crt",
    "ssl_key": "/etc/ssl/private/server.key"
  }
}
```

✅ **Strong Authentication**:
```bash
# Generate strong API key
openssl rand -hex 32
```

✅ **Rate Limiting**:
```json
{
  "security": {
    "rate_limit_enabled": true
  }
}
```

✅ **Input Validation**:
```json
{
  "security": {
    "validate_input": true,
    "max_record_limit": 1000
  }
}
```

✅ **Audit Logging**:
```json
{
  "security": {
    "audit_log_enabled": true,
    "audit_log_file": "/var/log/mcp-audit.log"
  }
}
```

---

## Audit Logging

### Log Format

```json
{
  "timestamp": "2025-11-15T10:30:00Z",
  "level": "INFO",
  "user": "api_key_abc123",
  "tool": "search_records",
  "model": "res.partner",
  "arguments": {"limit": 10},
  "status": "success",
  "duration_ms": 234,
  "ip_address": "192.168.1.100"
}
```

### Enable Audit Logging

```json
{
  "security": {
    "audit_log_enabled": true,
    "audit_log_file": "/var/log/mcp-audit.log",
    "audit_log_rotation": {
      "enabled": true,
      "max_bytes": 104857600,
      "backup_count": 10
    }
  }
}
```

---

## Security Best Practices

### 1. Principle of Least Privilege

✅ **DO**:
- Create dedicated Odoo user for MCP
- Grant only necessary permissions
- Use read-only user when possible

❌ **DON'T**:
- Use admin credentials
- Grant full access

### 2. Secrets Management

✅ **DO**:
```bash
# Store in environment
export ODOO_PASSWORD=$(vault read secret/odoo/password)

# Or use secrets manager
export ODOO_PASSWORD=$(aws secretsmanager get-secret-value ...)
```

❌ **DON'T**:
```json
{
  "odoo_password": "admin123"  // Don't hardcode!
}
```

### 3. Network Security

✅ **DO**:
- Use firewall rules
- Restrict to known IPs
- Use VPN for remote access

```bash
# Firewall example (ufw)
sudo ufw allow from 10.0.0.0/8 to any port 8000
```

### 4. Regular Updates

- Update Zenoo RPC regularly
- Monitor security advisories
- Apply Odoo security patches

### 5. Monitoring

**Monitor for**:
- Failed authentication attempts
- Rate limit violations
- Unusual access patterns
- Large data exports

---

## Security Incident Response

### Detection

Monitor logs for:
```bash
# Failed auth attempts
grep "authentication failed" /var/log/mcp-server.log

# Rate limit violations
grep "rate limit exceeded" /var/log/mcp-server.log

# Suspicious queries
grep "DELETE\|DROP" /var/log/mcp-audit.log
```

### Response Steps

1. **Isolate**: Disable compromised API keys
2. **Investigate**: Review audit logs
3. **Contain**: Block suspicious IPs
4. **Recover**: Rotate credentials
5. **Review**: Update security policies

---

## Compliance

### GDPR Compliance

- Audit logs track data access
- Support for data deletion (delete_record tool)
- User consent management through Odoo

### SOC 2 Compliance

- Access controls documented
- Audit trail maintained
- Encryption in transit (TLS)

---

## Related Documentation

- **[Configuration Guide](./03-configuration.md)** - Security configuration options
- **[Deployment Guide](./09-deployment.md)** - Secure deployment
- **[Troubleshooting](./10-troubleshooting.md)** - Security issues

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team
