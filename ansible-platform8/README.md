# Platform8 IAM Ansible Automation

Complete Ansible-based automation for deploying Ping Identity Platform8 IAM stack including PingDirectory (DS), PingAccess Management (AM), PingIdentity Management (IDM), Identity Gateway (IG), and Platform UI.

## Overview

This Ansible project automates the deployment and configuration of the complete Platform8 IAM stack with:
- HashiCorp Vault integration for secret management
- Environment-specific inventories (dev/qa/prod)
- Modular roles for each component
- Idempotent, non-destructive redeployment
- Manual execution support

## Architecture

See `platform8-iam-complete-architecture.html` for detailed architecture diagrams.

## Prerequisites

### Infrastructure
- Linux VMs for each component (DS, AM, IDM, IG, UI)
- Apache Tomcat 10.x pre-installed
- Java 21+ pre-installed
- SSH access from Ansible control node
- Network connectivity between components

### Software Binaries
Place the following in `../software/` directories:
```
software/
├── ds/DS-8.0.0.zip
├── am/AM-8.0.1.war
├── am/Amster-8.0.1.zip
├── idm/IDM-8.0.0.zip
├── ig/PingGateway-2025.3.0.zip
└── ui/PlatformUI-8.0.1.0523.zip
```

### HashiCorp Vault
Configure Vault with secrets at path: `platform8/{environment}/`

## Quick Start

### 1. Install Ansible Collections
```bash
ansible-galaxy collection install -r requirements.yml
```

### 2. Configure Vault
```bash
export VAULT_ADDR=https://vault.example.com
export VAULT_TOKEN=<your-token>
```

### 3. Deploy Full Platform
```bash
ansible-playbook -i inventories/dev/hosts.yml playbooks/bootstrap-platform8.yml --vault-password-file vault_pass.txt
```

### 4. Deploy Individual Components
```bash
# Deploy DS
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ds.yml

# Deploy AM
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-am.yml

# Deploy IDM
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-idm.yml

# Deploy IG
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ig.yml

# Deploy UI
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ui.yml
```

## Project Structure

```
ansible-platform8/
├── inventories/          # Environment inventories
├── playbooks/            # Ansible playbooks
├── roles/                # Ansible roles
├── group_vars/           # Global variables
└── ansible.cfg           # Ansible configuration
```

## Documentation

- **Deployment Guide:** See `../DEPLOYMENT_GUIDE.md`
- **Migration Plan:** See `../ANSIBLE_MIGRATION_PLAN.md`
- **Architecture:** See `../platform8-iam-complete-architecture.html`

## Environment-Specific Deployment

```bash
# Development
ansible-playbook -i inventories/dev/hosts.yml playbooks/bootstrap-platform8.yml

# QA
ansible-playbook -i inventories/qa/hosts.yml playbooks/bootstrap-platform8.yml

# Production
ansible-playbook -i inventories/prod/hosts.yml playbooks/bootstrap-platform8.yml
```

## Non-Destructive Redeployment

All playbooks are idempotent and preserve existing data:
- Existing DS data is preserved
- AM realms and trees are preserved (additive updates)
- IDM managed objects are preserved
- IG routes are preserved (additive updates)

See `../DEPLOYMENT_GUIDE.md` for detailed redeployment strategies.

## Support

For issues or questions, refer to the deployment guide or architecture documentation.

