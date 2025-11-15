---
title: "Known Issues & Limitations"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
---

# Known Issues & Limitations

## Overview

This section documents known issues, limitations, and workarounds for the Zenoo RPC MCP Server.

---

## Current Status

**Last Updated**: 2025-11-15
**Server Version**: 1.0.0
**Python Support**: 3.8, 3.9, 3.10, 3.11, 3.12
**Odoo Compatibility**: 18.0 (tested), others not verified

---

## Known Issues

### High Priority

#### ISSUE-001: Performance with Large Result Sets

**Severity**: Medium
**Status**: Open
**Affected Versions**: All

**Description**:
Queries returning >1000 records may experience slow performance.

**Impact**:
- Response times >5 seconds for large datasets
- Potential timeout errors
- High memory usage

**Workaround**:
```python
# Use pagination instead of large limits
# Bad:
result = await search_records(model="res.partner", limit=5000)

# Good:
for offset in range(0, 5000, 100):
    batch = await search_records(
        model="res.partner",
        limit=100,
        offset=offset
    )
```

**Planned Fix**: v1.1.0 - Implement streaming responses

---

#### ISSUE-002: Complex Domain Translation

**Severity**: Low
**Status**: Open
**Affected Versions**: All

**Description**:
Some complex Odoo domains with nested OR/AND logic may not translate correctly from natural language.

**Impact**:
- AI may generate incorrect domain filters
- Manual domain specification required

**Example**:
```
User: "Find customers in CA or TX with orders over $1000 and no invoices"

Expected domain:
[
    '|', ['state', '=', 'CA'], ['state', '=', 'TX'],
    ['order_ids.amount_total', '>', 1000],
    ['invoice_ids', '=', False]
]

AI may generate simpler domain missing nested logic
```

**Workaround**:
Use the `search_records` tool directly with explicit domain.

**Planned Fix**: v1.2.0 - Enhanced domain parser

---

### Medium Priority

#### ISSUE-003: Rate Limiting Accuracy

**Severity**: Low
**Status**: Open
**Affected Versions**: All

**Description**:
Rate limiting is per-process, not distributed across multiple server instances.

**Impact**:
- In multi-instance deployments, effective rate limit = limit × instance_count
- May allow more requests than intended

**Workaround**:
Use Redis-based distributed rate limiting (planned for v1.1.0) or implement at load balancer level.

---

#### ISSUE-004: WebSocket Transport Not Implemented

**Severity**: Low
**Status**: Planned
**Affected Versions**: All

**Description**:
WebSocket transport is not yet implemented.

**Impact**:
- No bi-directional streaming
- Cannot push updates from server to client

**Workaround**:
Use SSE (Server-Sent Events) transport for one-way streaming, or poll with HTTP.

**Planned Fix**: v1.3.0

---

## Limitations

### By Design

#### L-001: Read-Only Resources

**Description**: MCP Resources are read-only by design per MCP specification.

**Implication**: Use Tools for write operations.

#### L-002: No Direct SQL Access

**Description**: No direct SQL query execution for security.

**Implication**: All data access through Odoo ORM only.

#### L-003: Odoo Version Compatibility

**Description**: Currently tested only with Odoo 18.0.

**Implication**: Other Odoo versions may work but are not officially supported.

**Planned**: Testing with Odoo 16.0, 17.0 in future releases.

---

### Performance Limits

| Operation | Current Limit | Planned Improvement |
|-----------|--------------|---------------------|
| Records per query | 1000 (recommended) | 5000 (v1.1.0) |
| Concurrent requests | 50 | 200 (v1.1.0) |
| Request timeout | 30s | Configurable (v1.1.0) |
| Cache size | 10,000 items | Unlimited with Redis (v1.1.0) |

---

## Reporting Issues

### Before Reporting

1. **Check this document** for known issues
2. **Search GitHub Issues**: https://github.com/tuanle96/zenoo-rpc/issues
3. **Try latest version**: Upgrade to latest release
4. **Check logs**: Review error logs for details

### Report New Issue

**Include**:
- MCP server version
- Python version
- Odoo version
- Operating system
- Error message (full stack trace)
- Steps to reproduce
- Expected vs actual behavior

**GitHub Issue Template**:
```markdown
## Description
[Clear description of the issue]

## Environment
- MCP Server Version:
- Python Version:
- Odoo Version:
- OS:

## Steps to Reproduce
1.
2.
3.

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Error Logs
```
[Paste error logs here]
```

## Additional Context
[Any other relevant information]
```

---

## Issue Status Definitions

| Status | Meaning |
|--------|---------|
| **Open** | Issue confirmed, not yet fixed |
| **In Progress** | Actively being worked on |
| **Planned** | Scheduled for future release |
| **Fixed** | Resolved in latest version |
| **Wont Fix** | Will not be addressed |
| **Duplicate** | Already reported elsewhere |

---

## Related Documentation

- **[Troubleshooting](../10-troubleshooting.md)** - Common problems and solutions
- **[Roadmap](../13-roadmap/README.md)** - Planned improvements
- **[GitHub Issues](https://github.com/tuanle96/zenoo-rpc/issues)** - Active issue tracker

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-01
**Maintained By**: Zenoo RPC Development Team
