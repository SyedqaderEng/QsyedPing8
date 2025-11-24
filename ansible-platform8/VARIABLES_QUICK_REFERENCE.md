# Platform8 IAM Variables - Quick Reference

## Most Commonly Used Variables

| Variable | Example | Location |
|----------|---------|----------|
| `am_url` | `http://am.example.com:8081/am` | `inventories/dev/group_vars/all.yml` |
| `am_hostname` | `am.example.com` | `inventories/dev/group_vars/all.yml` |
| `idm_hostname` | `openidm.example.com` | `inventories/dev/group_vars/all.yml` |
| `ig_hostname` | `platform.example.com` | `inventories/dev/group_vars/all.yml` |
| `platform_hostname` | `platform.example.com` | `inventories/dev/group_vars/all.yml` |
| `tomcat_http_port` | `8081` | `inventories/dev/group_vars/am.yml` |
| `ig_https_port` | `9443` | `inventories/dev/group_vars/ig.yml` |
| `ds_admin_password` | `password` | **Vault**: `platform8/dev/ds/admin_password` |
| `am_admin_password` | `password` | **Vault**: `platform8/dev/am/admin_password` |
| `truststore_password` | `changeit` | `group_vars/all.yml` (override with Vault) |

## Variable Storage Locations

### Global (All Environments)
- **File**: `ansible-platform8/group_vars/all.yml`
- **Contains**: Base paths, default passwords, installation directories

### Environment-Specific (Dev)
- **Files**: `ansible-platform8/inventories/dev/group_vars/*.yml`
  - `all.yml` - Hostnames, versions, directories, URLs
  - `ds.yml` - Directory Services configuration
  - `am.yml` - Access Management configuration
  - `idm.yml` - Identity Management configuration
  - `ig.yml` - Identity Gateway configuration
  - `ui.yml` - Platform UI configuration

### Secrets (Vault)
- **Path Pattern**: `platform8/{environment}/{component}/{secret_name}`
- **Example**: `platform8/dev/ds/admin_password`

## Quick Lookup by Component

### AM (Access Management)
- URL: `am_url` → `http://am.example.com:8081/am`
- Hostname: `am_hostname` → `am.example.com`
- Port: `tomcat_http_port` → `8081`
- Admin: `am_admin_username` → `amadmin`
- Password: `am_admin_password` → **Vault**

### IDM (Identity Management)
- Hostname: `idm_hostname` → `openidm.example.com`
- HTTP Port: `boot_port_http` → `8080`
- HTTPS Port: `boot_port_https` → `8553`

### IG (Identity Gateway)
- Hostname: `ig_hostname` → `platform.example.com`
- HTTP Port: `ig_http_port` → `7080`
- HTTPS Port: `ig_https_port` → `9443`

### DS (Directory Services)
- IDRepo Server: `ds_idrepo_server` → `idrepo1.example.com`
- IDRepo LDAPS Port: `ds_idrepo_server_ldaps_port` → `2636`
- Config Server: `ds_amconfig_server` → `amconfig1.example.com`
- CTS Server: `ds_cts_server` → `cts1.example.com`
- Admin DN: `ds_admin_dn` → `cn=Directory Manager`
- Admin Password: `ds_admin_password` → **Vault**

For complete details, see `VARIABLES_REFERENCE.md`.

