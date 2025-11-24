# Platform8 IAM Ansible Automation Migration Plan

## Overview
Convert the existing shell script-based Platform8 IAM deployment (`install_platform8.sh`, `ds8.sh`, `am8.sh`, `idm8.sh`, `ig8.sh`, etc.) into a production-ready Ansible automation project with HashiCorp Vault integration, environment-specific inventories, and modular roles. Designed for on-premise deployments with manual playbook execution. Tomcat and supporting dependencies are assumed pre-installed.

## Current State Analysis

### Existing Components
- **Shell Scripts**: `install_platform8.sh` (orchestrator), `ds8.sh`, `am8.sh`, `am8fbc.sh`, `idm8.sh`, `ig8.sh`, `platformui.sh`
- **Configuration**: `platformconfig.env` (centralized config)
- **Config Files**: 
  - AM: `am/root/` (auth trees/nodes), `am/setenv.sh`, `am/configure_am.sh`
  - DS: `ds/configure_disk_thresholds.sh`
  - IDM: `idm/*.json` (repo.ds.json, access.json, managed.json, etc.)
  - IG: `ig/routes/*.json`, `ig/config.json`, `ig/admin.json`
- **Software**: Binaries stored in `software/` directories

### Key Functionality to Preserve
1. DS deployment (Config Store, CTS, IDRepo instances)
2. AM deployment (DS-based and FBC modes)
3. IDM deployment with external DS repository
4. IG deployment with routes
5. Platform UI deployment
6. AM configuration via REST API
7. Certificate management and truststore configuration
8. Service lifecycle management

## Target Ansible Structure

```
ansible-platform8/
├── README.md
├── ansible.cfg
├── requirements.yml
├── vault_pass.txt.example
├── .gitignore
├── inventories/
│   ├── dev/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   ├── ds.yml
│   │   │   ├── am.yml
│   │   │   ├── idm.yml
│   │   │   ├── ig.yml
│   │   │   └── ui.yml
│   │   └── host_vars/
│   ├── qa/
│   │   └── [same structure as dev]
│   └── prod/
│       └── [same structure as dev]
├── playbooks/
│   ├── site.yml                    # Master playbook
│   ├── deploy-ds.yml               # DS deployment
│   ├── deploy-am.yml               # AM deployment (DS-based)
│   ├── deploy-am-fbc.yml           # AM FBC deployment
│   ├── deploy-idm.yml              # IDM deployment
│   ├── deploy-ig.yml               # IG deployment
│   ├── deploy-ui.yml               # UI deployment
│   ├── configure-am-rest.yml      # AM REST configuration
│   └── bootstrap-platform8.yml    # Full bootstrap
├── roles/
│   ├── common/
│   │   ├── tasks/main.yml          # Prerequisites check, user creation, directories
│   │   ├── handlers/main.yml
│   │   └── vars/main.yml
│   ├── directory-services/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml         # Install Config, CTS, IDRepo instances
│   │   │   ├── configure.yml       # DS configuration
│   │   │   └── certificates.yml    # Certificate export/import
│   │   ├── handlers/main.yml
│   │   ├── templates/
│   │   │   └── [DS config templates if needed]
│   │   └── vars/main.yml
│   ├── access-management/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml         # WAR deployment, Amster setup
│   │   │   ├── configure.yml       # AM configuration via Amster
│   │   │   ├── amster.yml          # Amster operations
│   │   │   └── journeys.yml        # Import authentication trees
│   │   ├── handlers/main.yml
│   │   ├── templates/
│   │   │   ├── setenv.sh.j2        # Tomcat setenv.sh for AM
│   │   │   └── setenv-fbc.sh.j2    # FBC mode setenv.sh
│   │   └── vars/main.yml
│   ├── identity-management/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml         # IDM ZIP extraction
│   │   │   └── configure.yml       # IDM configuration deployment
│   │   ├── handlers/main.yml
│   │   ├── templates/
│   │   │   ├── repo.ds.json.j2
│   │   │   ├── access.json.j2
│   │   │   ├── managed.json.j2
│   │   │   ├── authentication.json.j2
│   │   │   ├── metrics.json.j2
│   │   │   ├── ui-configuration.json.j2
│   │   │   ├── ui-themerealm.json.j2
│   │   │   ├── servletfilter-cors.json.j2
│   │   │   └── resolver/boot.properties.j2
│   │   └── vars/main.yml
│   ├── identity-gateway/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml         # IG ZIP extraction
│   │   │   ├── configure.yml       # IG configuration
│   │   │   ├── routes.yml          # Route deployment
│   │   │   └── certificates.yml    # TLS keystore generation
│   │   ├── handlers/main.yml
│   │   ├── templates/
│   │   │   ├── config.json.j2
│   │   │   ├── admin.json.j2
│   │   │   └── routes/
│   │   │       ├── 01-am.json.j2
│   │   │       ├── 02-idm.json.j2
│   │   │       ├── 03-platform.json.j2
│   │   │       ├── 04-enduser.json.j2
│   │   │       └── 05-login.json.j2
│   │   └── vars/main.yml
│   ├── platform-ui/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   └── install.yml         # UI ZIP extraction and deployment
│   │   ├── handlers/main.yml
│   │   └── vars/main.yml
│   ├── vault-integration/
│   │   ├── tasks/main.yml          # Vault lookup utilities
│   │   └── vars/main.yml
│   └── tomcat/
│       ├── tasks/
│       │   ├── main.yml
│       │   └── service.yml         # Service management (start/stop/restart)
│       ├── handlers/main.yml
│       └── vars/main.yml
├── library/
│   └── [custom Ansible modules if needed]
├── filter_plugins/
│   └── [custom filters]
├── group_vars/
│   └── all.yml                     # Global defaults
├── templates/
│   └── [shared templates]
├── files/
│   └── [static files]
└── scripts/
    └── [helper scripts for manual execution]
```

## Implementation Phases

### Phase 1: Project Structure & Foundation
1. Create Ansible project directory structure
2. Set up `ansible.cfg` with best practices
3. Create `requirements.yml` for Ansible collections (community.general, hashi_vault)
4. Set up `.gitignore` for secrets and sensitive files
5. Create base inventory structure for dev/qa/prod

### Phase 2: Vault Integration
1. Create `roles/vault-integration/` role for HashiCorp Vault secret retrieval
2. Implement Vault lookup plugins for dynamic secret retrieval
3. Create Vault variable mapping (secrets → Ansible variables)
4. Document Vault path structure and required secrets
5. Create example Vault secret structure documentation

### Phase 3: Common Role & Prerequisites
1. Create `roles/common/` role for:
   - User creation (INSTALL_USER)
   - Directory structure setup
   - Java installation verification (assume Java 21+ pre-installed)
   - System prerequisites validation
2. Create `roles/tomcat/` role for Tomcat service management only (assume Tomcat 10.x pre-installed):
   - Service start/stop/restart via systemd
   - setenv.sh deployment
   - WAR file deployment to webapps directory
   - No Tomcat installation tasks

### Phase 4: Directory Services Role
1. Convert `ds8.sh` logic to Ansible tasks
2. Create `roles/directory-services/` with tasks for:
   - Process cleanup (kill_ds_processes)
   - Truststore preparation (prepare_am_truststore)
   - Config Store installation (install_config)
   - CTS Store installation (install_cts)
   - IDRepo Store installation (install_idrepo)
   - Certificate export and import (prepare_ds_certs)
   - LDAP connectivity testing (test_ldap_secure_bind)
3. Convert DS setup commands to Ansible modules/commands
4. Make tasks idempotent (check for existing installations)
5. Create Jinja2 templates for any DS configuration files

### Phase 5: Access Management Role
1. Convert `am8.sh` and `am8fbc.sh` logic to Ansible tasks
2. Create `roles/access-management/` with tasks for:
   - Tomcat service management
   - AM WAR deployment (clear_am)
   - Amster setup and execution (setup_amster, run_amster)
   - CTS configuration (configure_cts)
   - Alpha realm creation (create_alpha_realm)
   - Journey import (import_journeys)
   - setenv.sh template deployment (deploy_am_envfile)
3. Support both DS-based and FBC deployment modes
4. Convert Amster commands to Ansible tasks
5. Create Jinja2 templates for `setenv.sh`
6. Handle AM authentication trees/nodes from `am/root/` directory

### Phase 6: Identity Management Role
1. Convert `idm8.sh` logic to Ansible tasks
2. Create `roles/identity-management/` with tasks for:
   - IDM process management (stop_idm_if_running)
   - IDM extraction (extract_idm)
   - Truststore certificate import (configure_idm)
   - Configuration file deployment with variable substitution (copy_idm_config_files)
   - IDM service startup (start_idm)
3. Convert all IDM JSON config files to Jinja2 templates:
   - `repo.ds.json.j2`
   - `access.json.j2`
   - `managed.json.j2`
   - `authentication.json.j2`
   - `metrics.json.j2`
   - `ui-configuration.json.j2`
   - `ui-themerealm.json.j2`
   - `servletfilter-cors.json.j2`
   - `resolver/boot.properties.j2`
4. Implement variable substitution logic

### Phase 7: Identity Gateway Role
1. Convert `ig8.sh` logic to Ansible tasks
2. Create `roles/identity-gateway/` with tasks for:
   - IG cleanup (clear_ig)
   - TLS keystore generation (prepare_keys)
   - IG installation (install_ig)
   - Configuration file deployment with placeholder replacement (copy_ig_files, replace_placeholders)
   - IG service startup and health checks (start_ig)
3. Convert IG route JSON files to Jinja2 templates:
   - `routes/01-am.json.j2`
   - `routes/02-idm.json.j2`
   - `routes/03-platform.json.j2`
   - `routes/04-enduser.json.j2`
   - `routes/05-login.json.j2`
4. Convert `config.json` and `admin.json` to templates

### Phase 8: Platform UI Role
1. Convert `platformui.sh` logic to Ansible tasks
2. Create `roles/platform-ui/` with tasks for UI deployment
3. Handle UI ZIP extraction and deployment to Tomcat

### Phase 9: Playbooks
1. Create `playbooks/deploy-ds.yml` - DS deployment playbook
2. Create `playbooks/deploy-am.yml` - AM DS-based deployment
3. Create `playbooks/deploy-am-fbc.yml` - AM FBC deployment
4. Create `playbooks/deploy-idm.yml` - IDM deployment
5. Create `playbooks/deploy-ig.yml` - IG deployment
6. Create `playbooks/deploy-ui.yml` - UI deployment
7. Create `playbooks/configure-am-rest.yml` - AM REST configuration (convert `configure_am.sh`)
8. Create `playbooks/bootstrap-platform8.yml` - Full platform bootstrap (replaces `install_platform8.sh`)
9. Create `playbooks/site.yml` - Master playbook with all components

### Phase 10: Inventory & Variables
1. Create environment-specific inventories (dev/qa/prod)
2. Convert `platformconfig.env` variables to Ansible group_vars
3. Organize variables by component (ds.yml, am.yml, idm.yml, ig.yml, ui.yml)
4. Implement Vault variable overrides
5. Create variable precedence documentation

### Phase 11: Idempotency & Non-Destructive Redeploys
1. Add conditional checks for existing installations
2. Implement state checking (e.g., check if DS instance exists before setup)
3. Preserve existing data during redeploys
4. Add backup tasks before destructive operations
5. Implement rollback mechanisms where applicable

### Phase 12: Manual Execution Documentation
1. Create execution guide for manual playbook runs with command examples
2. Document Vault authentication setup for manual execution
3. Create helper scripts (optional) for common operations
4. Document best practices for on-premise manual deployments
5. Add troubleshooting guide for manual execution scenarios

### Phase 13: Documentation
1. Create comprehensive README.md with:
   - Architecture overview
   - Prerequisites
   - Installation instructions
   - Vault setup guide
   - Environment-specific deployment guides
2. Create architecture diagram (Mermaid/ASCII) showing:
   - VM layout
   - DS replication topology
   - Component dependencies
   - Network flow
3. Document idempotency rules
4. Document secret management best practices
5. Create DR (Disaster Recovery) architecture documentation
6. Add troubleshooting guide
7. Create example deployment scenarios

### Phase 14: Testing & Validation
1. Test each role independently
2. Test full bootstrap deployment
3. Test incremental redeployments
4. Test idempotency (run playbooks multiple times)
5. Validate Vault integration
6. Test environment-specific deployments

## Key Technical Considerations

### Component Dependencies & Deployment Order
Understanding the Platform8 IAM stack dependencies:
1. **DS (Directory Services)** - Foundation layer, must be deployed first
   - Config Store: AM configuration data
   - CTS Store: AM session/token storage
   - IDRepo Store: User identity data
2. **AM (Access Management)** - Depends on DS
   - Requires all 3 DS instances running
   - Uses Config Store for AM configuration
   - Uses IDRepo for user store
   - Uses CTS for token storage
3. **IDM (Identity Management)** - Depends on DS and AM
   - Requires IDRepo DS instance
   - Integrates with AM for authentication
4. **IG (Identity Gateway)** - Depends on AM and IDM
   - Routes traffic to AM and IDM
   - Requires AM and IDM to be operational
5. **UI (Platform UI)** - Depends on AM, IDM, and IG
   - Deployed into Tomcat (same instance as AM)
   - Uses IG URLs for routing

### Idempotency Requirements
- Check for existing DS instances before setup (verify installation directories)
- Verify AM deployment status before redeploy (check WAR file, AM directory)
- Check IDM process status before operations (pgrep validation)
- Validate IG configuration before changes (check config directory)
- Preserve existing data, realms, policies, users, routes during redeploys
- Implement backup tasks before destructive operations

### Vault Secret Structure
```
platform8/
├── dev/
│   ├── ds/
│   │   ├── admin_password          # DS root password
│   │   ├── deployment_id           # DS deployment ID
│   │   └── deployment_id_password  # DS deployment ID password
│   ├── am/
│   │   ├── admin_password          # AM admin password
│   │   └── truststore_password     # Java truststore password
│   ├── idm/
│   │   └── [IDM-specific secrets if needed]
│   └── ig/
│       └── [IG-specific secrets if needed]
├── qa/ [same structure]
└── prod/ [same structure]
```

### Variable Mapping
- Map all `platformconfig.env` variables to Ansible variables
- Use Vault lookups for sensitive values (passwords, deployment IDs)
- Maintain backward compatibility with existing config structure
- Organize variables by component in group_vars

### Template Conversion
- Convert all JSON config files with `{{variable}}` placeholders to Jinja2 templates
- Preserve original file structure and logic
- Replace shell `sed` variable substitution with Jinja2 templating
- Handle both DS-based and FBC mode configurations

### Service Management
- **Tomcat**: Use Ansible systemd module (assume pre-installed, systemd managed)
- **DS**: Use DS native start/stop scripts (start-ds, stop-ds)
- **IDM**: Use IDM startup.sh/shutdown.sh scripts
- **IG**: Use IG start.sh/stop.sh scripts
- Implement proper service dependencies and health checks
- Add wait conditions for service readiness

### On-Premise Deployment Considerations
- Assume all infrastructure pre-provisioned (VMs, network, DNS)
- Assume Tomcat 10.x pre-installed and configured
- Assume Java 21+ pre-installed
- Software binaries stored locally in `software/` directories
- Manual execution from Ansible control node
- SSH access configured for target hosts

## Migration Strategy

1. **Preserve Existing Structure**: Keep original shell scripts during migration for reference and validation
2. **Incremental Migration**: Migrate one component at a time following dependency order (DS → AM → IDM → IG → UI)
3. **Parallel Testing**: Test Ansible playbooks alongside existing scripts to ensure functional parity
4. **Documentation First**: Document each component's requirements, dependencies, and assumptions before implementation
5. **Manual Execution Focus**: Design for manual `ansible-playbook` execution with clear command examples
6. **Version Control**: Use Bitbucket for code versioning only (no CI/CD automation)

## Deliverables

1. Complete Ansible project structure with all directories and files
2. All roles implemented with idempotent tasks
3. Environment-specific inventories (dev/qa/prod) with proper variable organization
4. Comprehensive documentation including:
   - README with architecture overview
   - Manual execution guide with command examples
   - Vault setup and secret management guide
   - Environment-specific deployment procedures
   - Troubleshooting guide
5. Architecture diagrams (Mermaid/ASCII) showing:
   - VM layout and component distribution
   - DS replication topology (if applicable)
   - Component dependencies and deployment order
   - Network flow and port mappings
6. Deployment runbooks for:
   - Full bootstrap deployment
   - Incremental component deployment
   - Non-destructive redeployment
   - Component-specific updates
7. DR (Disaster Recovery) procedures and architecture documentation
8. Example execution scripts (optional helper scripts for common operations)

## Manual Execution Examples

### Basic Commands
```bash
# Full bootstrap deployment
ansible-playbook -i inventories/dev/hosts.yml playbooks/bootstrap-platform8.yml

# Deploy individual components
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ds.yml
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-am.yml
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-idm.yml
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ig.yml
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ui.yml

# AM FBC mode deployment
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-am-fbc.yml

# Configure AM via REST
ansible-playbook -i inventories/dev/hosts.yml playbooks/configure-am-rest.yml

# Environment-specific deployment
ansible-playbook -i inventories/prod/hosts.yml playbooks/bootstrap-platform8.yml
```

### With Vault
```bash
# Using vault password file
ansible-playbook -i inventories/dev/hosts.yml playbooks/bootstrap-platform8.yml --vault-password-file vault_pass.txt

# Using vault password prompt
ansible-playbook -i inventories/dev/hosts.yml playbooks/bootstrap-platform8.yml --ask-vault-pass
```

## Next Steps

Once you approve this plan, implementation will proceed phase by phase, starting with Phase 1 (Project Structure & Foundation) and moving sequentially through all 14 phases.

