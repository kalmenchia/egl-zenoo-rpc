---
title: "MCP Server Roadmap"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
---

# MCP Server Roadmap

## Overview

This document outlines the planned development roadmap for the Zenoo RPC MCP Server, including upcoming features, improvements, and version timelines.

---

## Version Timeline

### v1.0.0 (Current - Released Nov 2025)

✅ **Core Features**:
- 8 MCP tools (CRUD + advanced)
- 3 MCP resources
- 3 MCP prompts
- stdio, HTTP, SSE transports
- API key authentication
- Basic rate limiting
- Pydantic validation
- Transaction support
- Batch operations
- Analytics queries

---

### v1.1.0 (Planned - Q1 2026)

🔄 **Performance & Scalability**:
- [ ] Distributed rate limiting with Redis
- [ ] Streaming response support
- [ ] Improved connection pooling
- [ ] Query result caching enhancements
- [ ] Horizontal scaling support

🔄 **New Features**:
- [ ] OAuth 2.0 authentication
- [ ] Webhook support for event notifications
- [ ] Custom field mapping
- [ ] GraphQL-style query syntax
- [ ] Bulk import/export tools

🔄 **Developer Experience**:
- [ ] TypeScript SDK
- [ ] Python SDK improvements
- [ ] Auto-generated API documentation
- [ ] Interactive API explorer
- [ ] CLI tool for testing

**Target Release**: February 2026

---

### v1.2.0 (Planned - Q2 2026)

🔄 **AI Enhancements**:
- [ ] Enhanced natural language domain parsing
- [ ] AI-powered query optimization suggestions
- [ ] Automatic relationship inference
- [ ] Predictive analytics tools
- [ ] Smart data summarization

🔄 **Integration Features**:
- [ ] Slack bot integration
- [ ] Microsoft Teams integration
- [ ] Discord bot support
- [ ] Zapier integration
- [ ] n8n workflow support

🔄 **Advanced Tools**:
- [ ] Data import/export wizard
- [ ] Report builder tool
- [ ] Dashboard creation tool
- [ ] Workflow automation tool
- [ ] Data migration tool

**Target Release**: May 2026

---

### v1.3.0 (Planned - Q3 2026)

🔄 **Real-Time Features**:
- [ ] WebSocket transport
- [ ] Real-time data synchronization
- [ ] Live query subscriptions
- [ ] Change notifications
- [ ] Collaborative editing support

🔄 **Enterprise Features**:
- [ ] Multi-tenancy support
- [ ] Advanced audit logging
- [ ] Compliance reporting
- [ ] Data retention policies
- [ ] Role-based access control (RBAC)

🔄 **Monitoring & Observability**:
- [ ] Prometheus metrics export
- [ ] OpenTelemetry support
- [ ] Performance dashboards
- [ ] Health check endpoints
- [ ] SLA monitoring

**Target Release**: August 2026

---

### v2.0.0 (Planned - Q4 2026)

🔄 **Major Features**:
- [ ] Plugin marketplace
- [ ] Visual workflow builder
- [ ] No-code tool creator
- [ ] Multi-Odoo instance support
- [ ] Cross-instance queries

🔄 **Breaking Changes**:
- [ ] New authentication system
- [ ] Redesigned configuration format
- [ ] Updated MCP protocol support
- [ ] Improved error handling

**Target Release**: November 2026

---

## Feature Requests

### Most Requested Features

| Feature | Votes | Priority | Target Version |
|---------|-------|----------|---------------|
| WebSocket support | 45 | High | v1.3.0 |
| TypeScript SDK | 38 | High | v1.1.0 |
| Slack integration | 32 | Medium | v1.2.0 |
| Multi-instance support | 28 | Medium | v2.0.0 |
| GraphQL queries | 24 | Medium | v1.1.0 |
| Visual workflow builder | 22 | Low | v2.0.0 |

**Vote for features**: [GitHub Discussions](https://github.com/tuanle96/zenoo-rpc/discussions)

---

## Odoo Version Support

### Current Support

| Odoo Version | Status | Notes |
|--------------|--------|-------|
| 18.0 | ✅ Fully Supported | Tested and verified |
| 17.0 | ⚠️ Not Tested | May work, community feedback welcome |
| 16.0 | ⚠️ Not Tested | May work, community feedback welcome |
| 15.0 | ❌ Not Supported | Use at own risk |

### Planned Support

- **v1.1.0**: Add official support for Odoo 17.0
- **v1.2.0**: Add official support for Odoo 16.0
- **v2.0.0**: Drop support for Odoo <16.0

---

## Deprecation Timeline

### Planned Deprecations

| Feature | Deprecated | Removed | Replacement |
|---------|-----------|---------|-------------|
| Legacy auth format | v1.2.0 | v2.0.0 | New auth system |
| Old config format | v1.3.0 | v2.0.0 | JSON schema config |
| Python 3.8 support | v1.3.0 | v2.0.0 | Python 3.10+ |

---

## Migration Guides

### Upcoming Migrations

- **v1.x to v2.0**: Comprehensive migration guide (available Q3 2026)
- **Authentication Migration**: Guide for new auth system (v1.2.0)
- **Configuration Migration**: Automated migration tool (v1.3.0)

---

## Community Contributions

### How to Influence the Roadmap

1. **Vote on Features**: Use GitHub Discussions reactions
2. **Submit Feature Requests**: Open GitHub Discussion
3. **Contribute Code**: Submit Pull Requests
4. **Sponsor Development**: GitHub Sponsors

### Community Priorities

We prioritize features based on:
- **User votes** (40% weight)
- **Business value** (30% weight)
- **Technical feasibility** (20% weight)
- **Strategic alignment** (10% weight)

---

## Research & Exploration

### Under Investigation

These features are being researched but not yet committed:

- AI model fine-tuning for Odoo-specific queries
- Blockchain-based audit trails
- Edge computing support
- Mobile SDK (iOS/Android)
- Voice interface integration
- AR/VR data visualization

---

## Version Support Policy

### Support Lifecycle

| Version | Support Type | Duration |
|---------|-------------|----------|
| Current | Full support | Until next major |
| Previous major | Security fixes | 12 months |
| Older | Community only | No official support |

### Current Support Status

- **v1.0.x**: Full support until v2.0.0 release
- **v0.x**: Community support only

---

## Beta & Early Access

### Beta Program

Join our beta program to test upcoming features:

1. **Sign up**: [Beta Program Form](#)
2. **Get early access**: Test pre-release versions
3. **Provide feedback**: Help shape features
4. **Influence priority**: Beta feedback affects roadmap

**Beta Releases**:
- v1.1.0-beta: December 2025
- v1.2.0-beta: March 2026
- v1.3.0-beta: June 2026

---

## Related Documentation

- **[Known Issues](../12-known-issues/README.md)** - Current limitations
- **[Changelog](./changelog.md)** - Release history
- **[Contributing](../../contributing/)** - How to contribute

---

## Feedback

Have suggestions for the roadmap?

- **GitHub Discussions**: [Feature Requests](https://github.com/tuanle96/zenoo-rpc/discussions)
- **GitHub Issues**: [Report Bugs](https://github.com/tuanle96/zenoo-rpc/issues)
- **Email**: zenoo-rpc@example.com

---

**Last Updated**: 2025-11-15
**Next Review**: 2025-12-15
**Maintained By**: Zenoo RPC Development Team

---

*This roadmap is subject to change based on community feedback, technical constraints, and business priorities.*
