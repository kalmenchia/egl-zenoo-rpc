---
title: "Deployment Guide"
version: "1.0"
last_updated: "2025-11-15"
maintained_by: "Zenoo RPC Development Team"
status: "Active"
audience: "Administrators, DevOps"
---

# Deployment Guide

## Overview

This guide covers deploying the Zenoo RPC MCP Server to production environments with best practices for reliability, scalability, and security.

---

## Table of Contents

1. [Deployment Options](#deployment-options)
2. [System Requirements](#system-requirements)
3. [Production Deployment](#production-deployment)
4. [Docker Deployment](#docker-deployment)
5. [Kubernetes Deployment](#kubernetes-deployment)
6. [Monitoring & Maintenance](#monitoring--maintenance)
7. [Scaling Strategies](#scaling-strategies)

---

## Deployment Options

| Method | Use Case | Complexity | Scalability |
|--------|----------|------------|-------------|
| **Systemd Service** | Single server | Low | Low |
| **Docker** | Containerized | Medium | Medium |
| **Docker Compose** | Multi-container | Medium | Medium |
| **Kubernetes** | Enterprise | High | High |
| **Cloud Services** | Managed | Medium | High |

---

## System Requirements

### Production Server

**Minimum**:
- CPU: 2 cores
- RAM: 2GB
- Disk: 10GB
- OS: Ubuntu 20.04+ / Debian 11+ / RHEL 8+

**Recommended**:
- CPU: 4+ cores
- RAM: 8GB
- Disk: 50GB SSD
- OS: Ubuntu 22.04 LTS

### Network

- Outbound: Access to Odoo server
- Inbound: Port 8000 (HTTP) or stdio only
- Firewall: Restrict access to known IPs

---

## Production Deployment

### Step 1: Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python 3.10+
sudo apt install python3.10 python3.10-venv python3-pip -y

# Install Redis (for caching)
sudo apt install redis-server -y

# Install Nginx (reverse proxy)
sudo apt install nginx -y
```

### Step 2: Create Service User

```bash
# Create dedicated user
sudo useradd -m -s /bin/bash mcp-server

# Create directories
sudo mkdir -p /opt/mcp-server /var/log/mcp-server
sudo chown mcp-server:mcp-server /opt/mcp-server /var/log/mcp-server
```

### Step 3: Install Application

```bash
# Switch to service user
sudo su - mcp-server

# Create virtual environment
cd /opt/mcp-server
python3 -m venv venv
source venv/bin/activate

# Install Zenoo RPC with MCP
pip install zenoo-rpc[mcp,redis]
```

### Step 4: Configuration

```bash
# Create config directory
mkdir -p /opt/mcp-server/config

# Create production config
cat > /opt/mcp-server/config/production.json << 'EOFCONFIG'
{
  "name": "production-mcp-server",
  "transport_type": "http",
  "host": "127.0.0.1",
  "port": 8000,
  
  "security": {
    "auth_method": "api_key",
    "rate_limit_enabled": true,
    "rate_limit_requests": 1000,
    "rate_limit_window": 60
  },
  
  "performance": {
    "enable_caching": true,
    "cache_backend": "redis",
    "cache_config": {
      "url": "redis://localhost:6379/0"
    }
  },
  
  "log_level": "INFO",
  "log_file": "/var/log/mcp-server/server.log"
}
EOFCONFIG
```

### Step 5: Environment Variables

```bash
# Create .env file (secure this!)
cat > /opt/mcp-server/.env << 'EOFENV'
ODOO_URL=https://your-odoo-server.com
ODOO_DATABASE=production
ODOO_USERNAME=mcp_service
ODOO_PASSWORD=your-secure-password
MCP_API_KEYS=sk_prod_your_key_here
EOFENV

chmod 600 /opt/mcp-server/.env
```

### Step 6: Systemd Service

```bash
# Exit mcp-server user
exit

# Create systemd service
sudo cat > /etc/systemd/system/mcp-server.service << 'EOFSVC'
[Unit]
Description=Zenoo RPC MCP Server
After=network.target redis.service

[Service]
Type=simple
User=mcp-server
Group=mcp-server
WorkingDirectory=/opt/mcp-server
Environment="PATH=/opt/mcp-server/venv/bin"
EnvironmentFile=/opt/mcp-server/.env
ExecStart=/opt/mcp-server/venv/bin/python -m zenoo_rpc.mcp_server.cli --config /opt/mcp-server/config/production.json
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOFSVC

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable mcp-server
sudo systemctl start mcp-server

# Check status
sudo systemctl status mcp-server
```

### Step 7: Nginx Reverse Proxy

```bash
# Create Nginx config
sudo cat > /etc/nginx/sites-available/mcp-server << 'EOFNGINX'
server {
    listen 80;
    server_name mcp.example.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
EOFNGINX

# Enable site
sudo ln -s /etc/nginx/sites-available/mcp-server /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### Step 8: SSL/TLS Setup

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Get certificate
sudo certbot --nginx -d mcp.example.com

# Auto-renewal is configured automatically
```

---

## Docker Deployment

### Dockerfile

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install dependencies
RUN pip install --no-cache-dir zenoo-rpc[mcp,redis]

# Create non-root user
RUN useradd -m -u 1000 mcp && chown -R mcp:mcp /app
USER mcp

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

# Start server
CMD ["python", "-m", "zenoo_rpc.mcp_server.cli"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  mcp-server:
    build: .
    container_name: mcp-server
    restart: unless-stopped
    environment:
      - ODOO_URL=${ODOO_URL}
      - ODOO_DATABASE=${ODOO_DATABASE}
      - ODOO_USERNAME=${ODOO_USERNAME}
      - ODOO_PASSWORD=${ODOO_PASSWORD}
      - MCP_TRANSPORT_TYPE=http
      - MCP_HOST=0.0.0.0
      - MCP_PORT=8000
      - REDIS_URL=redis://redis:6379/0
      - MCP_ENABLE_CACHING=true
    ports:
      - "8000:8000"
    depends_on:
      - redis
    networks:
      - mcp-network
    volumes:
      - ./logs:/var/log/mcp-server
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  redis:
    image: redis:7-alpine
    container_name: mcp-redis
    restart: unless-stopped
    networks:
      - mcp-network
    volumes:
      - redis-data:/data

networks:
  mcp-network:
    driver: bridge

volumes:
  redis-data:
```

**Deploy**:
```bash
# Create .env file
cat > .env << 'EOF'
ODOO_URL=https://odoo.example.com
ODOO_DATABASE=production
ODOO_USERNAME=mcp_service
ODOO_PASSWORD=secure-password
