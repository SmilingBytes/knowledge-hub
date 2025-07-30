<p align="center">
  <img src="https://assets.website-files.com/605e01bc25f7e19a82e74788/624d9c4a375a55100be6b257_Airbyte_logo_color_dark.svg" alt="Airbyte">
</p>

![version 1.7.0](https://img.shields.io/badge/📌%20version-1.7-blue?style=flat-square)
![category](https://img.shields.io/badge/🏷️%20category-etl--tools-blue?style=flat-square)
[![docs](https://img.shields.io/badge/🔗%20docs-airbyte--docs-blue?style=flat-square)](https://docs.airbyte.com/platform/)

## 🌟 Overview

- **What it is:** Airbyte is an open-source ELT platform for syncing data from APIs, databases, and apps into warehouses.
- **Why it matters:** Offers fast setup with 300+ connectors, schema change handling, and easy integration with orchestration tools.
- **Ideal Use Cases:** Ingesting data into lakes/warehouses, building ELT pipelines, syncing SaaS and DB sources.
- **Main Alternatives:**
  - **Fivetran** – Managed, reliable, but costly
  - **Stitch** – Simple, but limited
  - **Meltano** – CLI-first, open-source, Singer-based

## Contents

## Table of contents

<!-- toc -->

- [🚀 Getting Started](#%F0%9F%9A%80-getting-started)
- [🛠️ Advanced Setup Examples](#%F0%9F%9B%A0%EF%B8%8F-advanced-setup-examples)
- [🔄 Management Commands](#%F0%9F%94%84-management-commands)
- [🌐 Network & Access Configuration](#%F0%9F%8C%90-network--access-configuration)
- [🐛 Troubleshooting](#%F0%9F%90%9B-troubleshooting)
- [🛡️ Security & Production Considerations](#%F0%9F%9B%A1%EF%B8%8F-security--production-considerations)
- [📚 Additional Resources](#%F0%9F%93%9A-additional-resources)

<!-- tocstop -->

## 🚀 Getting Started

### 🔥 Install abctl

PS. Must install Docker Desktop or Docker Engine

```bash
# Universal installer
curl -LsfS https://get.airbyte.com | bash -

# Mac installer
brew tap airbytehq/tap
brew install abctl

# go installer
go install github.com/airbytehq/abctl@latest

# Verify installation
abctl version
```

### 📌 Standard Installation

```bash
# Install Airbyte (takes 15-30 minutes)
abctl local install

# Check status
abctl local status

# Get credentials
abctl local credentials
```

### 🌐 Access Airbyte

- **Default URL**: <http://localhost:8000>
- **Credentials**: Run `abctl local credentials` to view

## ⚙️ Configuration Options

### 🔧 Installation Flags

| Flag | Default | Description | Example |
|------|---------|-------------|---------|
| `--host` | localhost | FQDN for external access | `airbyte.company.com` |
| `--port` | 8000 | Access port | `9000` |
| `--chart-version` | latest | Specific Airbyte version | `0.422.2` |
| `--low-resource-mode` | false | Reduced resource usage | `true` |
| `--values` | - | Custom Helm values file | `./values.yaml` |
| `--secret` | - | Kubernetes secrets | `./secret.yaml` |
| `--volume` | - | Mount host directories | `./data:/data` |
| `--no-browser` | false | Skip browser launch | `true` |
| `--insecure-cookies` | false | Disable secure cookies | `true` |

### 🌍 Remote/Production Setup

```bash
# External access configuration
abctl local install \
  --host airbyte.mycompany.com \
  --port 8080 \
  --insecure-cookies \
  --no-browser
```

### 💾 Low Resource Mode

```bash
# Minimal resource usage (disables Connector Builder)
abctl local install --low-resource-mode true
```

### 📁 Custom Values Configuration

```bash
# Use custom Helm values
abctl local install --values ./my-values.yaml
```

### 🔧 Basic Configuration Template

```yaml
# values.yaml
global:
  # Database configuration
  database:
    type: "external"  # or "internal"
    host: "my-database.cloud.com"
    port: 5432
    database: "airbyte"
    username: "airbyte_user"
    password: "secure_password"

  # Resource limits
  resources:
    limits:
      memory: "2Gi"
      cpu: "1000m"
    requests:
      memory: "1Gi"
      cpu: "500m"

# Web application settings
webapp:
  replicaCount: 1
  resources:
    limits:
      memory: "1Gi"
      cpu: "500m"

# Worker configuration
worker:
  replicaCount: 1
  resources:
    limits:
      memory: "2Gi"
      cpu: "1000m"

# Storage configuration
minio:
  enabled: false  # Use external storage

# Ingress configuration
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: airbyte.example.com
      paths:
        - path: /
          pathType: Prefix
```

### 🗄️ External Database Example

```yaml
# External PostgreSQL
global:
  database:
    type: "external"
    host: "postgres.amazonaws.com"
    port: 5432
    database: "airbyte_db"
    username: "airbyte"
    userSecretKey: "postgresql-password"

postgresql:
  enabled: false
```

### 🔐 Security Configuration

```yaml
# Security settings
global:
  auth:
    enabled: true

webapp:
  env_vars:
    AIRBYTE_SERVER_HOST: "https://airbyte.mycompany.com"
    CONNECTOR_BUILDER_SERVER_API_HOST: "https://airbyte.mycompany.com/connector-builder-api"
```

## 🛠️ Advanced Setup Examples

### 🏢 Production-Ready Setup

```bash
abctl local install \
  --host airbyte.production.com \
  --port 443 \
  --values production-values.yaml \
  --secret database-secret.yaml \
  --volume /data/airbyte:/airbyte-data \
  --no-browser
```

### 🔒 Docker Registry Authentication

```bash
# Private registry access
abctl local install \
  --docker-server docker.mycompany.com \
  --docker-username myuser \
  --docker-password mypassword \
  --docker-email [email protected]
```

### 📊 Multi-Volume Setup

```bash
# Mount multiple directories
abctl local install \
  --volume /host/data:/container/data \
  --volume /host/logs:/container/logs \
  --volume /host/config:/container/config
```

## 🔄 Management Commands

### 📊 Status & Information

```bash
# Check installation status
abctl local status

# View current credentials
abctl local credentials

# List Kubernetes deployments
abctl local deployments

# Get Docker images manifest
abctl images manifest
```

### 🔐 Credential Management

```bash
# Update login credentials
abctl local credentials \
  --email [email protected] \
  --password MyNewPassword
```

### 🔄 Restart & Updates

```bash
# Restart specific deployment
abctl local deployments --restart webapp

# Update to latest version
abctl local install  # Updates existing installation

# Update with new configuration
abctl local install --values new-values.yaml
```

### 🗑️ Cleanup Commands

```bash
# Stop containers, keep data
abctl local uninstall

# Remove everything including data
abctl local uninstall --persisted

# Clean abctl configuration
rm -rf ~/.airbyte/abctl
```

## 🌐 Network & Access Configuration

### 🔗 External Access Setup

```bash
# EC2/VM external access
abctl local install \
  --host $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) \
  --port 8000
```

### 🛡️ Security Best Practices

```bash
# HTTPS with custom certificates
abctl local install \
  --host secure.airbyte.com \
  --values secure-values.yaml \
  --secret tls-secret.yaml
```

## 🐛 Troubleshooting

### 🔍 Debug Mode

```bash
# Enable verbose logging
abctl --verbose local install
abctl --verbose local status
```

### 🚨 Common Issues

| Issue | Solution |
|-------|----------|
| ❌ **Port 8000 in use** | Use `--port 9000` |
| 🐳 **Docker not running** | Start Docker Desktop/Engine |
| 💾 **Low memory** | Use `--low-resource-mode` |
| 🌐 **Can't access externally** | Add `--host` flag with FQDN |
| 🔐 **Authentication failed** | Check credentials with `abctl local credentials` |

### 📋 System Information

```bash
# Check Docker status
docker info

# Verify Kubernetes cluster
kubectl --kubeconfig ~/.airbyte/abctl/abctl.kubeconfig get pods

# Check resource usage
docker stats
```

### 🔧 Reset Installation

```bash
# Complete reset
abctl local uninstall --persisted
rm -rf ~/.airbyte/abctl
docker system prune -a
abctl local install
```

## 🛡️ Security & Production Considerations

### 🔐 Environment Variables

```bash
# Disable telemetry
export DO_NOT_TRACK=1

# Docker registry credentials
export ABCTL_LOCAL_INSTALL_DOCKER_EMAIL="[email protected]"
export ABCTL_LOCAL_INSTALL_DOCKER_PASSWORD="password"
export ABCTL_LOCAL_INSTALL_DOCKER_SERVER="docker.io"
export ABCTL_LOCAL_INSTALL_DOCKER_USERNAME="username"
```

### 📁 Data Persistence

- **Config**: `~/.airbyte/abctl/`
- **Data**: Kubernetes persistent volumes
- **Logs**: Docker container logs

### 🔄 Backup Strategy

```bash
# Export configuration
kubectl --kubeconfig ~/.airbyte/abctl/abctl.kubeconfig get secret -o yaml > backup-secrets.yaml

# Backup data volumes
docker run --rm -v airbyte_data:/data -v $(pwd):/backup alpine tar czf /backup/airbyte-data-backup.tar.gz /data
```

## 📚 Additional Resources

- 📖 **Documentation**: [docs.airbyte.com](https://docs.airbyte.com/platform/next/deploying-airbyte/abctl)
- 🐙 **GitHub**: [github.com/airbytehq/abctl](https://github.com/airbytehq/abctl)
- 🔧 **Troubleshooting**: [docs.airbyte.com/deploying-airbyte/troubleshoot-deploy](https://docs.airbyte.com/deploying-airbyte/troubleshoot-deploy)
- ☁️ **Helm Charts**: [github.com/airbytehq/helm-charts](https://github.com/airbytehq/helm-charts)
