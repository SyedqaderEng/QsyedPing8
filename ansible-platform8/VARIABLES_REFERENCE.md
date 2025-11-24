# Platform8 IAM Ansible Variables Reference

This document provides a comprehensive reference of all variables used in the Platform8 IAM Ansible automation project, including their descriptions, example values, and where they are defined.

## Variable Location Legend

- **Global**: `group_vars/all.yml` - Applies to all environments
- **Dev**: `inventories/dev/group_vars/*.yml` - Development environment specific
- **QA**: `inventories/qa/group_vars/*.yml` - QA environment specific (to be created)
- **Prod**: `inventories/prod/group_vars/*.yml` - Production environment specific (to be created)
- **Vault**: Retrieved from HashiCorp Vault at runtime

---

## Global Variables (All Environments)

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `install_user` | System user for installation | `fradmin` | `group_vars/all.yml` |
| `install_user_home` | Home directory for install user | `/home/fradmin` | `group_vars/all.yml` |
| `base_install_dir` | Base directory for all installations | `/opt/ping` | `group_vars/all.yml` |
| `tomcat_dir` | Tomcat installation directory | `/opt/tomcat` | `group_vars/all.yml` |
| `software_dir` | Directory containing software binaries | `{{ playbook_dir }}/../software` | `group_vars/all.yml` |
| `am_context` | AM web application context path | `am` | `group_vars/all.yml` |
| `default_password` | Default password (override with Vault) | `password` | `group_vars/all.yml` |
| `ds_deployment_id` | DS deployment identifier | `AX8fkJybs4nP3qfAXN3CMw4BCYspGQ5CBVN1bkVDAOgwVKjG2Wo2ZTs` | `group_vars/all.yml` |
| `truststore_password` | Truststore password (override with Vault) | `changeit` | `group_vars/all.yml` |
| `ds_repo_suffix` | DS repository domain suffix | `forgerock.com` | `group_vars/all.yml` |
| `cookie_domain` | Cookie domain for sessions | `example.com` | `group_vars/all.yml` |
| `vault_addr` | HashiCorp Vault address | `https://vault.example.com` | `group_vars/all.yml` |
| `vault_auth_method` | Vault authentication method | `token` | `group_vars/all.yml` |
| `backup_base_dir` | Base directory for backups | `/opt/platform8/backups` | `group_vars/all.yml` |

---

## Environment Variables

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `environment` | Environment name (dev/qa/prod) | `dev` | `inventories/dev/group_vars/all.yml` |
| `vault_environment` | Vault environment path | `dev` | `inventories/dev/group_vars/all.yml` |

---

## Hostname Variables

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `am_hostname` | Access Management hostname | `am.example.com` | `inventories/dev/group_vars/all.yml` |
| `idm_hostname` | Identity Management hostname | `openidm.example.com` | `inventories/dev/group_vars/all.yml` |
| `ig_hostname` | Identity Gateway hostname | `platform.example.com` | `inventories/dev/group_vars/all.yml` |
| `platform_hostname` | Platform UI hostname | `platform.example.com` | `inventories/dev/group_vars/all.yml` |
| `login_hostname` | Login UI hostname | `login.example.com` | `inventories/dev/group_vars/all.yml` |
| `admin_hostname` | Admin UI hostname | `admin.example.com` | `inventories/dev/group_vars/all.yml` |
| `enduser_hostname` | End-user UI hostname | `enduser.example.com` | `inventories/dev/group_vars/all.yml` |
| `cts_hostname` | CTS store hostname | `cts1.example.com` | `inventories/dev/group_vars/all.yml` |
| `amconfig_hostname` | AM Config store hostname | `amconfig1.example.com` | `inventories/dev/group_vars/all.yml` |
| `amidmrepo_hostname` | AM IDM Repository hostname | `idrepo1.example.com` | `inventories/dev/group_vars/all.yml` |

---

## Software Version Variables

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `ds_version` | PingDirectory version | `8.0.0` | `inventories/dev/group_vars/all.yml` |
| `am_version` | PingAccess Management version | `8.0.1` | `inventories/dev/group_vars/all.yml` |
| `amster_version` | Amster version | `8.0.1` | `inventories/dev/group_vars/all.yml` |
| `idm_version` | PingIdentity Management version | `8.0.0` | `inventories/dev/group_vars/all.yml` |
| `ig_version` | Identity Gateway version | `2025.3.0` | `inventories/dev/group_vars/all.yml` |
| `ui_version` | Platform UI version | `8.0.1.0523` | `inventories/dev/group_vars/all.yml` |
| `tomcat_version` | Apache Tomcat version | `10.1.46` | `inventories/dev/group_vars/all.yml` |

---

## Software File Path Variables

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `ds_zip_file` | DS ZIP file path | `{{ software_dir }}/ds/DS-8.0.0.zip` | `inventories/dev/group_vars/all.yml` |
| `am_war` | AM WAR file path | `{{ software_dir }}/am/AM-8.0.1.war` | `inventories/dev/group_vars/all.yml` |
| `amster_zip` | Amster ZIP file path | `{{ software_dir }}/am/Amster-8.0.1.zip` | `inventories/dev/group_vars/all.yml` |
| `idm_zip` | IDM ZIP file path | `{{ software_dir }}/idm/IDM-8.0.0.zip` | `inventories/dev/group_vars/all.yml` |
| `ig_zip` | IG ZIP file path | `{{ software_dir }}/ig/PingGateway-2025.3.0.zip` | `inventories/dev/group_vars/all.yml` |
| `ui_zip` | UI ZIP file path | `{{ software_dir }}/ui/PlatformUI-8.0.1.0523.zip` | `inventories/dev/group_vars/all.yml` |

---

## Directory Services (DS) Variables

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `ds_dir` | DS base installation directory | `/opt/ping/ds` | `inventories/dev/group_vars/all.yml` |
| `ds_config_dir` | DS Config Store directory | `/opt/ping/ds/config` | `inventories/dev/group_vars/all.yml` |
| `ds_cts_dir` | DS CTS Store directory | `/opt/ping/ds/cts` | `inventories/dev/group_vars/all.yml` |
| `ds_idrepo_dir` | DS IDRepo Store directory | `/opt/ping/ds/idrepo` | `inventories/dev/group_vars/all.yml` |
| `ds_admin_dn` | DS admin distinguished name | `cn=Directory Manager` | `inventories/dev/group_vars/ds.yml` |
| `ds_admin_password` | DS admin password | `password` | **Vault**: `platform8/{env}/ds/admin_password` |
| `ds_deployment_id_password` | DS deployment ID password | `password` | **Vault**: `platform8/{env}/ds/deployment_id_password` |
| `ds_idrepo_server` | IDRepo server hostname | `idrepo1.example.com` | `inventories/dev/group_vars/ds.yml` |
| `ds_idrepo_server_ldap_port` | IDRepo LDAP port | `2389` | `inventories/dev/group_vars/ds.yml` |
| `ds_idrepo_server_ldaps_port` | IDRepo LDAPS port | `2636` | `inventories/dev/group_vars/ds.yml` |
| `ds_idrepo_server_http_port` | IDRepo HTTP port | `28081` | `inventories/dev/group_vars/ds.yml` |
| `ds_idrepo_server_https_port` | IDRepo HTTPS port | `28443` | `inventories/dev/group_vars/ds.yml` |
| `ds_idrepo_server_admin_connector_port` | IDRepo admin connector port | `24444` | `inventories/dev/group_vars/ds.yml` |
| `ds_idrepo_dn` | IDRepo base DN | `ou=identities` | `inventories/dev/group_vars/ds.yml` |
| `ds_amconfig_server` | Config Store server hostname | `amconfig1.example.com` | `inventories/dev/group_vars/ds.yml` |
| `ds_amconfig_server_ldap_port` | Config Store LDAP port | `3389` | `inventories/dev/group_vars/ds.yml` |
| `ds_amconfig_server_ldaps_port` | Config Store LDAPS port | `3636` | `inventories/dev/group_vars/ds.yml` |
| `ds_amconfig_server_http_port` | Config Store HTTP port | `38081` | `inventories/dev/group_vars/ds.yml` |
| `ds_amconfig_server_https_port` | Config Store HTTPS port | `38443` | `inventories/dev/group_vars/ds.yml` |
| `ds_amconfig_server_admin_connector_port` | Config Store admin connector port | `34444` | `inventories/dev/group_vars/ds.yml` |
| `ds_cts_server` | CTS server hostname | `cts1.example.com` | `inventories/dev/group_vars/ds.yml` |
| `ds_cts_server_ldap_port` | CTS LDAP port | `1389` | `inventories/dev/group_vars/ds.yml` |
| `ds_cts_server_ldaps_port` | CTS LDAPS port | `1636` | `inventories/dev/group_vars/ds.yml` |
| `ds_cts_server_http_port` | CTS HTTP port | `18081` | `inventories/dev/group_vars/ds.yml` |
| `ds_cts_server_https_port` | CTS HTTPS port | `18443` | `inventories/dev/group_vars/ds.yml` |
| `ds_cts_server_admin_connector_port` | CTS admin connector port | `14444` | `inventories/dev/group_vars/ds.yml` |
| `ds_config_admin_dn` | Config Store admin DN | `uid=am-config,ou=admins,ou=am-config` | `inventories/dev/group_vars/ds.yml` |
| `ds_config_admin_password` | Config Store admin password | `password` | **Vault**: `platform8/{env}/ds/config_admin_password` |
| `ds_idrepo_admin_dn` | IDRepo admin DN | `uid=am-identity-bind-account,ou=admins,ou=identities` | `inventories/dev/group_vars/ds.yml` |
| `ds_idrepo_admin_password` | IDRepo admin password | `password` | **Vault**: `platform8/{env}/ds/idrepo_admin_password` |
| `ds_cts_admin_dn` | CTS admin DN | `uid=openam_cts,ou=admins,ou=famrecords,ou=openam-session,ou=tokens` | `inventories/dev/group_vars/ds.yml` |
| `ds_cts_admin_password` | CTS admin password | `password` | **Vault**: `platform8/{env}/ds/cts_admin_password` |
| `ldap_bind_password` | LDAP bind password | `password` | `inventories/dev/group_vars/ds.yml` |
| `dskeymgr` | DS key manager executable path | `/opt/ping/ds/idrepo/opendj/bin/dskeymgr` | `inventories/dev/group_vars/all.yml` |

---

## Access Management (AM) Variables

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `am_url` | AM base URL | `http://am.example.com:8081/am` | `inventories/dev/group_vars/all.yml` |
| `am_admin_username` | AM admin username | `amadmin` | `inventories/dev/group_vars/am.yml` |
| `am_admin_password` | AM admin password | `password` | **Vault**: `platform8/{env}/am/admin_password` |
| `am_config_store_password` | AM Config Store password | `password` | **Vault**: `platform8/{env}/am/config_store_password` |
| `am_identity_store_password` | AM Identity Store password | `password` | **Vault**: `platform8/{env}/am/identity_store_password` |
| `am_cts_store_password` | AM CTS Store password | `password` | **Vault**: `platform8/{env}/am/cts_store_password` |
| `am_deployment_mode` | AM deployment mode | `ds` or `fbc` | `inventories/dev/group_vars/am.yml` |
| `am_dir` | AM directory (DS mode) | `/home/fradmin/openam` | `inventories/dev/group_vars/all.yml` |
| `am_cfg_dir` | AM config directory | `/home/fradmin/.openamcfg` | `inventories/dev/group_vars/all.yml` |
| `am_fbc_dir` | AM FBC directory | `/home/fradmin/am` | `inventories/dev/group_vars/all.yml` |
| `am_truststore_folder` | AM truststore directory | `/opt/ping/am/security/keystores` | `inventories/dev/group_vars/all.yml` |
| `am_truststore` | AM truststore file path | `/opt/ping/am/security/keystores/truststore` | `inventories/dev/group_vars/all.yml` |
| `am_journeys_dir` | AM authentication journeys directory | `{{ playbook_dir }}/../am/root` | `inventories/dev/group_vars/all.yml` |
| `amster_dir` | Amster installation directory | `{{ software_dir }}/am/amster` | `inventories/dev/group_vars/am.yml` |
| `am_realms` | AM realms to create | `[{name: "alpha", parent_path: "/", active: true}]` | `inventories/dev/group_vars/am.yml` |
| `am_import_journeys` | Import authentication journeys | `true` | `inventories/dev/group_vars/am.yml` |
| `tomcat_http_port` | Tomcat HTTP port | `8081` | `inventories/dev/group_vars/am.yml` |
| `tomcat_webapps_dir` | Tomcat webapps directory | `/opt/tomcat/webapps` | `inventories/dev/group_vars/all.yml` |
| `tomcat_bin_dir` | Tomcat bin directory | `/opt/tomcat/bin` | `inventories/dev/group_vars/all.yml` |
| `tomcat_am_war` | AM WAR file in Tomcat | `/opt/tomcat/webapps/am.war` | `inventories/dev/group_vars/all.yml` |
| `am_setenv` | AM setenv.sh file path | `/opt/tomcat/bin/setenv.sh` | `inventories/dev/group_vars/all.yml` |
| `secure_xui_dir` | Secure XUI directory | `/opt/tomcat/webapps/am/XUI` | `inventories/dev/group_vars/all.yml` |

---

## Identity Management (IDM) Variables

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `idm_dir` | IDM base installation directory | `/opt/ping` | `inventories/dev/group_vars/all.yml` |
| `idm_extract_dir` | IDM extracted directory | `/opt/ping/openidm` | `inventories/dev/group_vars/all.yml` |
| `idm_config_dir` | IDM configuration directory | `/opt/ping/openidm/conf` | `inventories/dev/group_vars/all.yml` |
| `idm_logs_dir` | IDM logs directory | `/opt/ping/openidm/logs` | `inventories/dev/group_vars/all.yml` |
| `boot_port_http` | IDM HTTP port | `8080` | `inventories/dev/group_vars/idm.yml` |
| `boot_port_https` | IDM HTTPS port | `8553` | `inventories/dev/group_vars/idm.yml` |
| `boot_port_mutualauth` | IDM mutual auth port | `9444` | `inventories/dev/group_vars/idm.yml` |
| `boot_host` | IDM hostname | `openidm.example.com` | `inventories/dev/group_vars/idm.yml` |
| `boot_file` | IDM boot.properties file | `/opt/ping/openidm/resolver/boot.properties` | `inventories/dev/group_vars/all.yml` |
| `idm_cert_file` | IDM certificate file | `/opt/ping/am/security/keystores/ds-repo-ca-cert.pem` | `inventories/dev/group_vars/all.yml` |
| `idm_truststore` | IDM truststore file | `/opt/ping/openidm/security/truststore` | `inventories/dev/group_vars/all.yml` |
| `idm_storepass_file` | IDM storepass file | `/opt/ping/openidm/security/storepass` | `inventories/dev/group_vars/all.yml` |
| `idm_config_files` | List of IDM config files to deploy | `["repo.ds.json", "access.json", ...]` | `inventories/dev/group_vars/idm.yml` |
| `idm_repo_config_file` | IDM repository config file name | `repo.ds.json` | `inventories/dev/group_vars/idm.yml` |

---

## Identity Gateway (IG) Variables

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `ig_dir` | IG installation directory | `/opt/ping/identity-gateway-2025.3.0` | `inventories/dev/group_vars/all.yml` |
| `ig_cfg_dir` | IG config directory | `/home/fradmin/.openig` | `inventories/dev/group_vars/all.yml` |
| `ig_config` | IG config path | `/home/fradmin/.openig/config` | `inventories/dev/group_vars/all.yml` |
| `ig_routes` | IG routes directory | `/home/fradmin/.openig/config/routes` | `inventories/dev/group_vars/all.yml` |
| `ig_tmp_dir` | IG temporary directory | `/home/fradmin/.openig/tmp` | `inventories/dev/group_vars/all.yml` |
| `ig_key_dir` | IG security keys directory | `/opt/ping/ig_security` | `inventories/dev/group_vars/all.yml` |
| `ig_keystore_pin_file` | IG keystore PIN file | `/opt/ping/ig_security/keystore.pin` | `inventories/dev/group_vars/all.yml` |
| `ig_keystore_file` | IG keystore file | `/opt/ping/ig_security/keystore` | `inventories/dev/group_vars/all.yml` |
| `ig_ca_cert_file` | IG CA certificate file | `/opt/ping/ig_security/ca-cert.pem` | `inventories/dev/group_vars/all.yml` |
| `ig_http_port` | IG HTTP port | `7080` | `inventories/dev/group_vars/ig.yml` |
| `ig_https_port` | IG HTTPS port | `9443` | `inventories/dev/group_vars/ig.yml` |
| `ig_route_files` | List of IG route files | `["01-am.json", "02-idm.json", ...]` | `inventories/dev/group_vars/ig.yml` |
| `ig_config_files` | List of IG config files | `["config.json", "admin.json", "logback.xml"]` | `inventories/dev/group_vars/ig.yml` |
| `ig_keystore_password` | IG keystore password | `password` | **Vault**: `platform8/{env}/ig/keystore_password` |

---

## Platform UI Variables

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `tmp_root` | Temporary root directory | `{{ software_dir }}/tmp` | `inventories/dev/group_vars/all.yml` |
| `tmp_ui` | Temporary UI directory | `{{ software_dir }}/tmp/ui-zip` | `inventories/dev/group_vars/all.yml` |
| `webapps_dir` | Tomcat webapps directory | `/opt/tomcat/webapps` | `inventories/dev/group_vars/all.yml` |
| `login_port` | Login UI port (reference) | `8083` | `inventories/dev/group_vars/ui.yml` |
| `admin_port` | Admin UI port (reference) | `8082` | `inventories/dev/group_vars/ui.yml` |
| `enduser_port` | End-user UI port (reference) | `8888` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars` | UI environment variables for URL replacement | See below | `inventories/dev/group_vars/ui.yml` |

### UI Environment Variables (ui_env_vars)

| Variable Name | Description | Example Value | Location |
|--------------|-------------|---------------|----------|
| `ui_env_vars.AM_URL` | AM URL for UI | `https://platform.example.com:9443/am` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.AM_ADMIN_URL` | AM Admin URL | `https://platform.example.com:9443/am/ui-admin` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.IDM_REST_URL` | IDM REST API URL | `https://platform.example.com:9443/openidm` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.IDM_ADMIN_URL` | IDM Admin URL | `https://platform.example.com:9443/admin` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.IDM_UPLOAD_URL` | IDM Upload URL | `https://platform.example.com:9443/upload` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.IDM_EXPORT_URL` | IDM Export URL | `https://platform.example.com:9443/export` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.ENDUSER_UI_URL` | End-user UI URL | `https://platform.example.com:9443/enduser-ui/` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.PLATFORM_ADMIN_URL` | Platform Admin UI URL | `https://platform.example.com:9443/platform-ui/` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.ENDUSER_CLIENT_ID` | End-user OAuth client ID | `end-user-ui` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.ADMIN_CLIENT_ID` | Admin OAuth client ID | `idm-admin-ui` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.THEME` | UI theme | `default` | `inventories/dev/group_vars/ui.yml` |
| `ui_env_vars.PLATFORM_UI_LOCALE` | UI locale | `en` | `inventories/dev/group_vars/ui.yml` |

---

## HashiCorp Vault Secret Paths

All sensitive passwords and secrets should be stored in HashiCorp Vault. The following table shows the Vault path structure:

| Secret Name | Vault Path | Description | Example Value |
|------------|-----------|-------------|---------------|
| DS Admin Password | `platform8/{env}/ds/admin_password` | DS root admin password | `password` |
| DS Deployment ID Password | `platform8/{env}/ds/deployment_id_password` | DS deployment ID password | `password` |
| DS Config Admin Password | `platform8/{env}/ds/config_admin_password` | Config Store admin password | `password` |
| DS IDRepo Admin Password | `platform8/{env}/ds/idrepo_admin_password` | IDRepo admin password | `password` |
| DS CTS Admin Password | `platform8/{env}/ds/cts_admin_password` | CTS admin password | `password` |
| AM Admin Password | `platform8/{env}/am/admin_password` | AM admin password | `password` |
| AM Config Store Password | `platform8/{env}/am/config_store_password` | AM Config Store password | `password` |
| AM Identity Store Password | `platform8/{env}/am/identity_store_password` | AM Identity Store password | `password` |
| AM CTS Store Password | `platform8/{env}/am/cts_store_password` | AM CTS Store password | `password` |
| AM Truststore Password | `platform8/{env}/am/truststore_password` | AM truststore password | `changeit` |
| IG Keystore Password | `platform8/{env}/ig/keystore_password` | IG keystore password | `password` |

**Note**: Replace `{env}` with `dev`, `qa`, or `prod` based on the environment.

---

## Variable Precedence

Variables are loaded in the following order (later sources override earlier ones):

1. `group_vars/all.yml` (global defaults)
2. `inventories/{env}/group_vars/all.yml` (environment-specific)
3. `inventories/{env}/group_vars/{component}.yml` (component-specific)
4. `inventories/{env}/host_vars/{hostname}.yml` (host-specific)
5. Vault secrets (runtime lookup)

---

## Usage Examples

### Accessing Variables in Templates

```jinja2
# In a Jinja2 template
{{ am_url }}
{{ ds_idrepo_server }}:{{ ds_idrepo_server_ldaps_port }}
```

### Accessing Variables in Playbooks

```yaml
# In an Ansible playbook
- name: Display AM URL
  debug:
    msg: "AM URL is {{ am_url }}"
```

### Accessing Vault Secrets

```yaml
# In group_vars or playbooks
ds_admin_password: "{{ lookup('hashi_vault', 'secret/platform8/' + vault_environment + '/ds/admin_password') }}"
```

---

## Notes

- All file paths use forward slashes (`/`) even on Windows, as Ansible normalizes paths
- Port numbers are integers (no quotes needed in YAML)
- Hostnames should be FQDNs (fully qualified domain names)
- Passwords should always be retrieved from Vault in production
- Environment-specific values should be set in `inventories/{env}/group_vars/` files

