# Platform8 IAM Deployment Guide

## Table of Contents
1. [Required Components and Tools](#required-components-and-tools)
2. [Service Accounts](#service-accounts)
3. [Project Structure](#project-structure)
4. [Step-by-Step Deployment Process](#step-by-step-deployment-process)
5. [Non-Destructive Redeployment](#non-destructive-redeployment)

---

## Required Components and Tools

### Software Components

#### 1. **PingDirectory (DS)**
- **Version:** 8.0.0
- **Package:** DS-8.0.0.zip
- **Description:** LDAP directory server providing three instances:
  - **Config Store:** Stores AM configuration data (realms, policies, authentication trees)
  - **CTS Store:** Stores OAuth2/OIDC tokens, SAML assertions, and session data
  - **IDRepo Store:** Stores user identities, groups, and attributes
- **Location:** `software/ds/DS-8.0.0.zip`

#### 2. **PingAccess Management (AM)**
- **Version:** 8.0.1
- **Package:** AM-8.0.1.war
- **Description:** Authentication and authorization server providing:
  - OAuth2/OIDC services
  - SAML services
  - Authentication trees and journeys
  - Session management
  - Policy enforcement
- **Location:** `software/am/AM-8.0.1.war`
- **Deployment:** Deployed in Apache Tomcat 10.x

#### 3. **Amster**
- **Version:** 8.0.1
- **Package:** Amster-8.0.1.zip
- **Description:** Command-line configuration tool for AM:
  - Bootstrap AM installation
  - Import/export AM configuration
  - Manage realms, authentication trees, and policies
  - Automates AM setup via scripts
- **Location:** `software/am/Amster-8.0.1.zip`
- **Usage:** Extracted and used during AM deployment

#### 4. **PingIdentity Management (IDM)**
- **Version:** 8.0.0
- **Package:** IDM-8.0.0.zip
- **Description:** Identity lifecycle management providing:
  - User provisioning and deprovisioning
  - Self-service capabilities
  - Workflow automation
  - Compliance management
  - Reconciliation engine
- **Location:** `software/idm/IDM-8.0.0.zip`
- **Deployment:** Standalone Java application

#### 5. **PingIdentity Gateway (IG)**
- **Version:** 2025.3.0
- **Package:** PingGateway-2025.3.0.zip
- **Description:** Reverse proxy and API gateway providing:
  - Request routing to AM and IDM
  - Authentication enforcement
  - Request/response transformation
  - TLS termination
  - Load balancing capabilities
- **Location:** `software/ig/PingGateway-2025.3.0.zip`
- **Deployment:** Standalone Java application

#### 6. **Platform UI**
- **Version:** 8.0.1.0523
- **Package:** PlatformUI-8.0.1.0523.zip
- **Description:** Web-based user interfaces:
  - **Platform UI:** Admin console for administrators
  - **End-user UI:** Self-service portal for end users
  - **Login UI:** Authentication and login pages
- **Location:** `software/ui/PlatformUI-8.0.1.0523.zip`
- **Deployment:** Deployed in Apache Tomcat (same instance as AM)

### Supporting Infrastructure

#### 7. **Apache Tomcat**
- **Version:** 10.x (pre-installed)
- **Description:** Java servlet container hosting:
  - AM WAR file
  - Platform UI components
- **Management:** Systemd service
- **Ports:** HTTP 8081 (configurable)

#### 8. **Java Runtime**
- **Version:** OpenJDK 21+ (pre-installed)
- **Description:** Required for all Platform8 components
- **Verification:** `java -version` should show Java 21 or higher

#### 9. **HashiCorp Vault**
- **Description:** Secret management system for:
  - Storing passwords
  - Storing deployment IDs
  - Managing certificates
  - Environment-specific secrets
- **Integration:** Ansible Vault lookup plugins

---

## Service Accounts

### 1. **Installation User (INSTALL_USER)**
- **Default:** `fradmin`
- **Purpose:** User account that owns all Platform8 installations
- **Permissions:**
  - Read/write access to installation directories
  - Execute permissions for startup scripts
  - Access to configuration directories
- **Home Directory:** `~/.openamcfg`, `~/.openig`, `~/openam`
- **Ansible Variable:** `install_user: "fradmin"`

### 2. **DS Admin Account**
- **DN:** `cn=Directory Manager`
- **Purpose:** Root administrator for all DS instances
- **Usage:**
  - Initial DS setup
  - Configuration changes
  - Certificate export
- **Storage:** HashiCorp Vault path: `platform8/{env}/ds/admin_password`
- **Ansible Variable:** `ds_admin_password` (from Vault)

### 3. **DS Deployment ID Account**
- **Deployment ID:** `AX8fkJybs4nP3qfAXN3CMw4BCYspGQ5CBVN1bkVDAOgwVKjG2Wo2ZTs`
- **Purpose:** Shared deployment identifier for all DS instances
- **Usage:**
  - Certificate generation
  - Replication setup
  - Key management
- **Storage:** HashiCorp Vault path: `platform8/{env}/ds/deployment_id` and `deployment_id_password`
- **Ansible Variable:** `ds_deployment_id` (from Vault)

### 4. **AM Config Store Admin**
- **DN:** `uid=am-config,ou=admins,ou=am-config`
- **Purpose:** Administrator for AM Config Store DS instance
- **Usage:** AM configuration read/write operations
- **Storage:** HashiCorp Vault path: `platform8/{env}/am/config_store_password`
- **Ansible Variable:** `am_config_store_password` (from Vault)

### 5. **AM Identity Store Bind Account**
- **DN:** `uid=am-identity-bind-account,ou=admins,ou=identities`
- **Purpose:** Bind account for AM to read user data from IDRepo
- **Usage:** User authentication and lookup operations
- **Storage:** HashiCorp Vault path: `platform8/{env}/am/identity_store_password`
- **Ansible Variable:** `am_identity_store_password` (from Vault)

### 6. **AM CTS Admin Account**
- **DN:** `uid=openam_cts,ou=admins,ou=famrecords,ou=openam-session,ou=tokens`
- **Purpose:** Administrator for AM CTS Store DS instance
- **Usage:** Token and session storage operations
- **Storage:** HashiCorp Vault path: `platform8/{env}/am/cts_store_password`
- **Ansible Variable:** `am_cts_store_password` (from Vault)

### 7. **AM Admin Account**
- **Username:** `amadmin`
- **Purpose:** Primary administrator for Access Management
- **Usage:**
  - AM console access
  - REST API authentication
  - Configuration management
- **Storage:** HashiCorp Vault path: `platform8/{env}/am/admin_password`
- **Ansible Variable:** `am_admin_password` (from Vault)

### 8. **Tomcat Service Account**
- **User:** `tomcat` (systemd service)
- **Purpose:** Runs Tomcat process
- **Permissions:** Read/write access to Tomcat directories
- **Management:** Systemd service management

### 9. **Ansible Control Node User**
- **Purpose:** User executing Ansible playbooks
- **Requirements:**
  - SSH access to target hosts
  - Sudo privileges (if needed)
  - Vault authentication credentials
- **Vault Access:** Requires Vault token or authentication method

---

## Project Structure

```
ansible-platform8/
├── README.md                          # Project documentation
├── ansible.cfg                        # Ansible configuration
├── requirements.yml                   # Ansible collections
├── vault_pass.txt.example            # Example Vault password file
├── .gitignore                        # Git ignore rules
│
├── inventories/                       # Environment inventories
│   ├── dev/
│   │   ├── hosts.yml                 # Dev host definitions
│   │   ├── group_vars/
│   │   │   ├── all.yml               # Global variables
│   │   │   ├── ds.yml                # DS-specific variables
│   │   │   ├── am.yml                # AM-specific variables
│   │   │   ├── idm.yml               # IDM-specific variables
│   │   │   ├── ig.yml                # IG-specific variables
│   │   │   └── ui.yml                # UI-specific variables
│   │   └── host_vars/                # Host-specific overrides
│   ├── qa/                           # Same structure as dev
│   └── prod/                         # Same structure as dev
│
├── playbooks/                        # Ansible playbooks
│   ├── site.yml                      # Master playbook (all components)
│   ├── bootstrap-platform8.yml       # Full platform bootstrap
│   ├── deploy-ds.yml                 # DS deployment only
│   ├── deploy-am.yml                 # AM deployment (DS-based)
│   ├── deploy-am-fbc.yml            # AM deployment (FBC mode)
│   ├── deploy-idm.yml                # IDM deployment
│   ├── deploy-ig.yml                 # IG deployment
│   ├── deploy-ui.yml                 # Platform UI deployment
│   └── configure-am-rest.yml        # AM REST API configuration
│
├── roles/                            # Ansible roles
│   ├── common/                       # Common tasks
│   │   ├── tasks/main.yml           # User creation, directories
│   │   ├── handlers/main.yml
│   │   └── vars/main.yml
│   ├── directory-services/           # DS role
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml          # DS instance installation
│   │   │   ├── configure.yml        # DS configuration
│   │   │   └── certificates.yml     # Certificate management
│   │   ├── handlers/main.yml
│   │   ├── templates/
│   │   └── vars/main.yml
│   ├── access-management/            # AM role
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml          # WAR deployment
│   │   │   ├── configure.yml        # AM configuration
│   │   │   ├── amster.yml           # Amster operations
│   │   │   └── journeys.yml         # Auth tree import
│   │   ├── handlers/main.yml
│   │   ├── templates/
│   │   │   ├── setenv.sh.j2
│   │   │   └── setenv-fbc.sh.j2
│   │   └── vars/main.yml
│   ├── identity-management/          # IDM role
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml          # IDM extraction
│   │   │   └── configure.yml        # Config file deployment
│   │   ├── handlers/main.yml
│   │   ├── templates/                # All IDM JSON configs
│   │   └── vars/main.yml
│   ├── identity-gateway/             # IG role
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml
│   │   │   ├── configure.yml
│   │   │   ├── routes.yml
│   │   │   └── certificates.yml
│   │   ├── handlers/main.yml
│   │   ├── templates/                # IG route configs
│   │   └── vars/main.yml
│   ├── platform-ui/                  # UI role
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   └── install.yml
│   │   ├── handlers/main.yml
│   │   └── vars/main.yml
│   ├── vault-integration/            # Vault role
│   │   ├── tasks/main.yml
│   │   └── vars/main.yml
│   └── tomcat/                       # Tomcat role
│       ├── tasks/
│       │   ├── main.yml
│       │   └── service.yml          # Service management
│       ├── handlers/main.yml
│       └── vars/main.yml
│
├── group_vars/
│   └── all.yml                       # Global default variables
│
├── templates/                        # Shared templates
├── files/                            # Static files
└── scripts/                          # Helper scripts
```

---

## Step-by-Step Deployment Process

### Prerequisites

#### Before Starting Deployment

1. **Infrastructure Setup**
   ```bash
   # Verify VMs are provisioned
   # Verify network connectivity
   # Verify DNS/hostname resolution
   # Verify SSH access from Ansible control node
   ```

2. **Software Binaries**
   ```bash
   # Place all software binaries in software/ directories:
   software/
   ├── ds/DS-8.0.0.zip
   ├── am/AM-8.0.1.war
   ├── am/Amster-8.0.1.zip
   ├── idm/IDM-8.0.0.zip
   ├── ig/PingGateway-2025.3.0.zip
   └── ui/PlatformUI-8.0.1.0523.zip
   ```

3. **HashiCorp Vault Setup**
   ```bash
   # Configure Vault with required secrets:
   # - DS admin passwords
   # - AM admin passwords
   # - Deployment IDs
   # - Truststore passwords
   ```

4. **Ansible Setup**
   ```bash
   # Install Ansible
   pip install ansible
   
   # Install required collections
   ansible-galaxy collection install -r requirements.yml
   
   # Configure Vault authentication
   export VAULT_ADDR=https://vault.example.com
   export VAULT_TOKEN=<your-token>
   ```

### Deployment Steps

#### Step 1: Deploy Directory Services (DS)
```bash
# Deploy all DS instances (Config, CTS, IDRepo)
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ds.yml --vault-password-file vault_pass.txt

# What it does:
# - Stops existing DS processes
# - Installs Config Store instance
# - Installs CTS Store instance
# - Installs IDRepo Store instance
# - Exports CA certificates
# - Imports certificates into AM truststore
# - Tests LDAP connectivity
```

**Verification:**
```bash
# Check DS instances are running
ldapsearch -h amconfig1.example.com -p 3636 -D "uid=am-config,ou=admins,ou=am-config" -w <password> -b "ou=am-config" -Z

# Check ports are listening
netstat -tuln | grep -E "3389|1389|2389|3636|1636|2636"
```

#### Step 2: Deploy Access Management (AM)
```bash
# Deploy AM with DS-based configuration
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-am.yml --vault-password-file vault_pass.txt

# OR deploy AM with FBC mode
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-am-fbc.yml --vault-password-file vault_pass.txt

# What it does:
# - Stops Tomcat
# - Deploys AM WAR file
# - Configures setenv.sh
# - Starts Tomcat
# - Sets up Amster
# - Runs AM bootstrap (install-openam)
# - Configures CTS data store
# - Creates Alpha realm
# - Imports authentication trees
```

**Verification:**
```bash
# Check AM is accessible
curl http://am.example.com:8081/am/

# Check Tomcat is running
systemctl status tomcat

# Verify AM console access
# Open browser: http://am.example.com:8081/am/
```

#### Step 3: Configure AM via REST API
```bash
# Apply additional AM configuration via REST
ansible-playbook -i inventories/dev/hosts.yml playbooks/configure-am-rest.yml --vault-password-file vault_pass.txt

# What it does:
# - Logs in as amadmin
# - Creates realms
# - Configures OAuth2 clients
# - Sets up policies
# - Configures other AM settings
```

#### Step 4: Deploy Identity Management (IDM)
```bash
# Deploy IDM
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-idm.yml --vault-password-file vault_pass.txt

# What it does:
# - Stops existing IDM process
# - Extracts IDM ZIP
# - Imports DS certificate into IDM truststore
# - Deploys configuration files (JSON templates)
# - Updates boot.properties
# - Starts IDM service
```

**Verification:**
```bash
# Check IDM is running
ps aux | grep openidm

# Check IDM REST API
curl http://openidm.example.com:8080/openidm/info/ping

# Check IDM logs
tail -f /opt/ping/openidm/logs/openidm0.log
```

#### Step 5: Deploy Identity Gateway (IG)
```bash
# Deploy IG
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ig.yml --vault-password-file vault_pass.txt

# What it does:
# - Stops existing IG process
# - Generates TLS keystore
# - Extracts IG ZIP
# - Deploys configuration files
# - Deploys route definitions
# - Starts IG service
# - Performs health checks
```

**Verification:**
```bash
# Check IG health
curl http://platform.example.com:7080/openig/ping
curl -k https://platform.example.com:9443/openig/ping

# Check IG is running
ps aux | grep openig
```

#### Step 6: Deploy Platform UI
```bash
# Deploy Platform UI
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ui.yml --vault-password-file vault_pass.txt

# What it does:
# - Extracts Platform UI ZIP
# - Updates JavaScript URLs
# - Deploys UI components to Tomcat
# - Restarts Tomcat
```

**Verification:**
```bash
# Check UI is accessible through IG
curl -k https://platform.example.com:9443/platform-ui/
curl -k https://platform.example.com:9443/enduser-ui/
```

#### Step 7: Full Bootstrap (Alternative)
```bash
# Deploy entire platform in one command
ansible-playbook -i inventories/dev/hosts.yml playbooks/bootstrap-platform8.yml --vault-password-file vault_pass.txt

# This executes all steps in order:
# 1. Deploy DS
# 2. Deploy AM
# 3. Deploy IDM
# 4. Deploy IG
# 5. Configure AM REST
# 6. Deploy UI
```

### After Deployment

1. **Verify All Services**
   ```bash
   # Check all services are running
   ansible-playbook -i inventories/dev/hosts.yml playbooks/verify-services.yml
   ```

2. **Test Authentication Flow**
   ```bash
   # Test login through IG
   curl -k https://platform.example.com:9443/am/XUI/
   ```

3. **Backup Configuration**
   ```bash
   # Create initial backup
   ansible-playbook -i inventories/dev/hosts.yml playbooks/backup-config.yml
   ```

---

## Non-Destructive Redeployment

### Idempotency Strategy

All playbooks are designed to be **idempotent** - they can be run multiple times without destroying existing data.

### Component-Specific Redeployment

#### DS Redeployment

**Scenario:** DS instance already exists, need to update configuration

**Strategy:**
```yaml
# In roles/directory-services/tasks/install.yml
- name: Check if DS instance exists
  stat:
    path: "{{ ds_instance_dir }}/opendj"
  register: ds_instance_exists

- name: Install DS instance
  command: ./setup ...
  args:
    chdir: "{{ ds_instance_dir }}/opendj"
  when: not ds_instance_exists.stat.exists

- name: Update DS configuration (if needed)
  # Only update specific config, don't reinstall
  when: ds_instance_exists.stat.exists
```

**What is Preserved:**
- Existing DS data
- User accounts
- Configuration data
- Replication setup

**What Can Be Updated:**
- Configuration settings (via dsconfig)
- Certificate renewals
- Log levels
- Performance tuning

#### AM Redeployment

**Scenario:** AM already deployed, need to update WAR or configuration

**Strategy:**
```yaml
# In roles/access-management/tasks/install.yml
- name: Check if AM is deployed
  stat:
    path: "{{ am_dir }}"
  register: am_deployed

- name: Backup existing AM configuration
  command: amster export ...
  when: am_deployed.stat.exists

- name: Deploy AM WAR
  copy:
    src: "{{ am_war }}"
    dest: "{{ tomcat_webapps_dir }}/{{ am_context }}.war"
  notify: restart tomcat

- name: Import configuration (only if new)
  command: amster import-config ...
  when: not am_deployed.stat.exists or am_force_reimport | default(false)
```

**What is Preserved:**
- Realms and realm configuration
- Authentication trees
- OAuth2 clients
- Policies
- User sessions (if CTS preserved)
- User data (in IDRepo)

**What Can Be Updated:**
- AM WAR version (upgrade)
- Authentication trees (additive import)
- New realms (additive)
- Configuration settings

**Critical:** Use `amster export` before redeployment to backup configuration.

#### IDM Redeployment

**Scenario:** IDM already running, need to update configuration

**Strategy:**
```yaml
# In roles/identity-management/tasks/install.yml
- name: Check if IDM is installed
  stat:
    path: "{{ idm_extract_dir }}"
  register: idm_installed

- name: Backup IDM configuration
  copy:
    src: "{{ idm_config_dir }}/*.json"
    dest: "{{ backup_dir }}/idm-config-{{ ansible_date_time.epoch }}/"
  when: idm_installed.stat.exists

- name: Extract IDM (only if not exists)
  unarchive:
    src: "{{ idm_zip }}"
    dest: "{{ idm_dir }}"
  when: not idm_installed.stat.exists

- name: Deploy configuration files
  template:
    src: "{{ item }}.j2"
    dest: "{{ idm_config_dir }}/{{ item }}"
  loop: "{{ idm_config_files }}"
  # Only update if file changed (Ansible handles this automatically)
```

**What is Preserved:**
- Managed objects (users, roles, etc.)
- Reconciliation data
- Workflow instances
- Audit logs
- IDM repository data in DS

**What Can Be Updated:**
- Configuration files (JSON)
- boot.properties
- Truststore certificates

#### IG Redeployment

**Scenario:** IG already running, need to update routes or configuration

**Strategy:**
```yaml
# In roles/identity-gateway/tasks/configure.yml
- name: Check if IG is configured
  stat:
    path: "{{ ig_config_dir }}"
  register: ig_configured

- name: Backup existing routes
  copy:
    src: "{{ ig_routes_dir }}/*.json"
    dest: "{{ backup_dir }}/ig-routes-{{ ansible_date_time.epoch }}/"
  when: ig_configured.stat.exists

- name: Deploy route files
  template:
    src: "routes/{{ item }}.j2"
    dest: "{{ ig_routes_dir }}/{{ item | replace('.j2', '') }}"
  loop: "{{ ig_route_files }}"
  notify: restart ig
  # Ansible only updates if content changed
```

**What is Preserved:**
- Existing routes (additive updates)
- TLS keystore
- Configuration settings

**What Can Be Updated:**
- Route definitions
- Configuration files
- Logging configuration

#### Platform UI Redeployment

**Scenario:** UI already deployed, need to update

**Strategy:**
```yaml
# In roles/platform-ui/tasks/install.yml
- name: Check if UI is deployed
  stat:
    path: "{{ webapps_dir }}/platform"
  register: ui_deployed

- name: Deploy UI components
  copy:
    src: "{{ item }}"
    dest: "{{ webapps_dir }}/{{ item | basename }}"
  loop: "{{ ui_components }}"
  notify: restart tomcat
  # Ansible only updates if files changed
```

**What is Preserved:**
- User preferences
- Custom themes (if stored separately)

**What Can Be Updated:**
- UI files (static content)
- JavaScript files
- CSS files

### Redeployment Best Practices

#### 1. **Always Backup Before Redeployment**
```yaml
# Create backup playbook
- name: Create backup
  include_role:
    name: backup
  vars:
    backup_components:
      - ds
      - am
      - idm
      - ig
      - ui
```

#### 2. **Use Check Mode First**
```bash
# Dry-run to see what would change
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-am.yml --check --diff
```

#### 3. **Incremental Updates**
```bash
# Update only specific component
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-am.yml --tags "amster,journeys"
```

#### 4. **State Validation**
```yaml
# In each role, validate current state
- name: Validate AM is operational
  uri:
    url: "{{ am_url }}/json/serverinfo/*"
    method: GET
  register: am_status
  failed_when: am_status.status != 200
```

#### 5. **Rollback Capability**
```yaml
# Store previous configuration versions
- name: Version configuration files
  copy:
    src: "{{ item }}"
    dest: "{{ config_backup_dir }}/{{ ansible_date_time.epoch }}/{{ item | basename }}"
  loop: "{{ config_files }}"
```

### Redeployment Scenarios

#### Scenario 1: AM WAR Upgrade
```bash
# 1. Backup current configuration
ansible-playbook -i inventories/dev/hosts.yml playbooks/backup-am.yml

# 2. Export AM configuration
amster export --path /backup/am-config-$(date +%Y%m%d)

# 3. Deploy new WAR
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-am.yml --tags "war_deploy"

# 4. Verify AM is operational
# 5. Re-import configuration if needed
```

#### Scenario 2: Add New Authentication Tree
```bash
# 1. Add new tree JSON to am/root/AuthTree/
# 2. Run AM deployment (only journeys import)
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-am.yml --tags "journeys"

# Existing trees are preserved, new tree is added
```

#### Scenario 3: Update IDM Configuration
```bash
# 1. Update template file
# 2. Deploy only IDM configuration
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-idm.yml --tags "configure"

# Existing managed objects preserved
```

#### Scenario 4: Update IG Route
```bash
# 1. Update route template
# 2. Deploy only routes
ansible-playbook -i inventories/dev/hosts.yml playbooks/deploy-ig.yml --tags "routes"

# Other routes preserved, only updated route changes
```

### Data Preservation Guarantees

| Component | Data Preserved | Update Method |
|-----------|---------------|---------------|
| **DS Config** | All AM configuration | Only updates if template changed |
| **DS CTS** | All tokens/sessions | Preserved during redeployment |
| **DS IDRepo** | All user data | Preserved during redeployment |
| **AM** | Realms, trees, policies | Additive updates via Amster |
| **IDM** | Managed objects | Preserved in DS, config files updated |
| **IG** | Routes, config | Additive route updates |
| **UI** | Static files | File-level updates only |

### Safety Mechanisms

1. **Idempotency Checks:** All tasks check for existing installations
2. **Backup Tasks:** Automatic backups before destructive operations
3. **State Validation:** Verify services before and after deployment
4. **Rollback Support:** Version-controlled configuration backups
5. **Dry-Run Mode:** Test changes before applying (`--check` flag)

---

## Summary

This deployment guide provides:
- Complete list of required components and their purposes
- All service accounts needed with descriptions
- Detailed project structure
- Step-by-step deployment instructions
- Comprehensive non-destructive redeployment strategies

All playbooks are designed to be **safe for redeployment** - they preserve existing data and only update what's necessary.

