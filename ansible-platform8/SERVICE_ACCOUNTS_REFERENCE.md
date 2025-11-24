# Platform8 IAM Service Accounts Reference

This document lists all service accounts, bind accounts, and admin accounts required for each Platform8 IAM product.

---

## Overview

| Product | Service Accounts Required | Purpose |
|---------|--------------------------|---------|
| **DS** | 4 accounts | Root admin + 3 store-specific admins |
| **AM** | 1 account | AM console/admin access |
| **IDM** | 3 internal users | IDM internal operations |
| **IG** | 0 accounts | Uses system account |
| **UI** | 0 accounts | Uses AM/IDM accounts |
| **System** | 1 account | Installation user |

---

## Directory Services (DS) Service Accounts

### 1. DS Root Administrator
| Property | Value |
|----------|-------|
| **DN** | `cn=Directory Manager` |
| **Variable** | `ds_admin_dn` |
| **Password Variable** | `ds_admin_password` |
| **Vault Path** | `platform8/{env}/ds/admin_password` |
| **Purpose** | Root administrator for all DS instances (Config, CTS, IDRepo) |
| **Usage** | - Initial DS setup<br>- Configuration changes<br>- Certificate export/import<br>- Replication setup |
| **Created By** | DS setup script during installation |
| **Location** | All DS instances (Config Store, CTS Store, IDRepo Store) |

### 2. DS Config Store Admin
| Property | Value |
|----------|-------|
| **DN** | `uid=am-config,ou=admins,ou=am-config` |
| **Variable** | `ds_config_admin_dn` |
| **Password Variable** | `ds_config_admin_password` |
| **Vault Path** | `platform8/{env}/ds/config_admin_password` |
| **Purpose** | Administrator for AM Config Store DS instance |
| **Usage** | - AM configuration read/write operations<br>- Config Store management |
| **Created By** | DS setup script (am-config profile) |
| **Location** | Config Store DS instance only |
| **Base DN** | `ou=am-config` |

### 3. DS IDRepo Bind Account
| Property | Value |
|----------|-------|
| **DN** | `uid=am-identity-bind-account,ou=admins,ou=identities` |
| **Variable** | `ds_idrepo_admin_dn` |
| **Password Variable** | `ds_idrepo_admin_password` |
| **Vault Path** | `platform8/{env}/ds/idrepo_admin_password` |
| **Purpose** | Bind account for AM and IDM to read/write user data from IDRepo |
| **Usage** | - User authentication<br>- User lookup operations<br>- User provisioning<br>- Identity data access |
| **Created By** | DS setup script (am-identity-store profile) |
| **Location** | IDRepo Store DS instance only |
| **Base DN** | `ou=identities` |

### 4. DS CTS Admin Account
| Property | Value |
|----------|-------|
| **DN** | `uid=openam_cts,ou=admins,ou=famrecords,ou=openam-session,ou=tokens` |
| **Variable** | `ds_cts_admin_dn` |
| **Password Variable** | `ds_cts_admin_password` |
| **Vault Path** | `platform8/{env}/ds/cts_admin_password` |
| **Purpose** | Administrator for AM CTS (Common Token Store) DS instance |
| **Usage** | - Token storage operations<br>- Session management<br>- OAuth2 token persistence |
| **Created By** | DS setup script (am-cts profile) |
| **Location** | CTS Store DS instance only |
| **Base DN** | `ou=famrecords,ou=openam-session,ou=tokens` |

### 5. DS Deployment ID Account
| Property | Value |
|----------|-------|
| **Deployment ID** | `AX8fkJybs4nP3qfAXN3CMw4BCYspGQ5CBVN1bkVDAOgwVKjG2Wo2ZTs` |
| **Variable** | `ds_deployment_id` |
| **Password Variable** | `ds_deployment_id_password` |
| **Vault Path** | `platform8/{env}/ds/deployment_id` and `deployment_id_password` |
| **Purpose** | Shared deployment identifier for all DS instances |
| **Usage** | - Certificate generation<br>- Replication setup<br>- Key management<br>- TLS key pair generation |
| **Created By** | Provided during DS setup |
| **Location** | All DS instances (shared) |

---

## Access Management (AM) Service Accounts

### 1. AM Admin Account
| Property | Value |
|----------|-------|
| **Username** | `amadmin` |
| **Variable** | `am_admin_username` |
| **Password Variable** | `am_admin_password` |
| **Vault Path** | `platform8/{env}/am/admin_password` |
| **Purpose** | Primary administrator for Access Management |
| **Usage** | - AM console access<br>- REST API authentication<br>- Configuration management<br>- Realm administration<br>- Policy management |
| **Created By** | AM bootstrap process (install-openam) |
| **Location** | AM application (stored in Config Store) |
| **Default Realm** | `/` (root realm) |

### 2. AM Store Passwords
These are passwords for AM to connect to DS stores (not separate accounts):

| Store | Password Variable | Vault Path | Purpose |
|-------|------------------|------------|---------|
| **Config Store** | `am_config_store_password` | `platform8/{env}/am/config_store_password` | AM to Config Store connection |
| **Identity Store** | `am_identity_store_password` | `platform8/{env}/am/identity_store_password` | AM to IDRepo connection |
| **CTS Store** | `am_cts_store_password` | `platform8/{env}/am/cts_store_password` | AM to CTS Store connection |

**Note**: These passwords correspond to the DS admin accounts:
- Config Store password = `ds_config_admin_password`
- Identity Store password = `ds_idrepo_admin_password`
- CTS Store password = `ds_cts_admin_password`

---

## Identity Management (IDM) Service Accounts

### 1. IDM Admin User (Internal)
| Property | Value |
|----------|-------|
| **Internal User** | `internal/user/openidm-admin` |
| **Subject** | `(usr!amAdmin)` |
| **Roles** | `internal/role/openidm-authorized`, `internal/role/openidm-admin` |
| **Purpose** | IDM internal administrator account |
| **Usage** | - IDM administrative operations<br>- System configuration<br>- Privileged operations |
| **Created By** | IDM internal (pre-configured) |
| **Authentication** | Via AM (subject mapping) |

### 2. IDM Provisioning User (Internal)
| Property | Value |
|----------|-------|
| **Internal User** | `internal/user/idm-provisioning` |
| **Subject** | `(age!idm-provisioning)` |
| **Roles** | `internal/role/platform-provisioning` |
| **Purpose** | Service account for AM-to-IDM provisioning operations |
| **Usage** | - User provisioning from AM<br>- Managed object operations<br>- Policy operations |
| **Created By** | IDM internal (pre-configured) |
| **Authentication** | Via OAuth2 client `idm-provisioning` |

### 3. IDM Anonymous User (Internal)
| Property | Value |
|----------|-------|
| **Internal User** | `internal/user/anonymous` |
| **Roles** | `internal/role/openidm-reg` |
| **Purpose** | Anonymous/unauthenticated access |
| **Usage** | - Public endpoints<br>- Registration flows<br>- Unauthenticated operations |
| **Created By** | IDM internal (pre-configured) |

### 4. IDM OAuth2 Client
| Property | Value |
|----------|-------|
| **Client ID** | `idm-provisioning` |
| **Variable** | Referenced in AM configuration |
| **Purpose** | OAuth2 client for AM-to-IDM communication |
| **Usage** | - AM provisioning to IDM<br>- Cross-product authentication |
| **Created By** | AM configuration (via REST API or Amster) |
| **Location** | AM Alpha realm |

---

## Identity Gateway (IG) Service Accounts

| Property | Value |
|----------|-------|
| **Service Accounts** | None required |
| **Purpose** | IG runs as system user |
| **Usage** | - Reverse proxy operations<br>- Route handling<br>- TLS termination |
| **Authentication** | Uses AM/IDM for backend authentication |

**Note**: IG does not require separate service accounts. It acts as a reverse proxy and forwards authentication to AM/IDM.

---

## Platform UI Service Accounts

| Property | Value |
|----------|-------|
| **Service Accounts** | None required |
| **Purpose** | UI uses AM/IDM accounts |
| **Usage** | - User interface access<br>- Admin console access |
| **OAuth Clients** | - `end-user-ui` (End-user UI)<br>- `idm-admin-ui` (Admin UI) |

**Note**: Platform UI does not require separate service accounts. It uses OAuth2 clients that authenticate against AM.

---

## System/Infrastructure Accounts

### 1. Installation User
| Property | Value |
|----------|-------|
| **Username** | `fradmin` |
| **Variable** | `install_user` |
| **Home Directory** | `/home/fradmin` |
| **Purpose** | System user for running installations and services |
| **Usage** | - File ownership<br>- Service execution<br>- Configuration file storage |
| **Permissions** | - Sudo access (for service management)<br>- Ownership of installation directories |
| **Created By** | Ansible `common` role |

### 2. Tomcat Service Account (Optional)
| Property | Value |
|----------|-------|
| **Username** | `tomcat` (if using systemd) |
| **Purpose** | Runs Tomcat process |
| **Usage** | - Tomcat service execution<br>- Process management |
| **Created By** | System/Tomcat installation |
| **Note** | May use `install_user` if not using systemd |

---

## Service Account Summary Table

| Account Name | Product | Type | DN/Username | Vault Path | Required |
|--------------|---------|------|-------------|------------|----------|
| Directory Manager | DS | Admin | `cn=Directory Manager` | `platform8/{env}/ds/admin_password` | ✅ Yes |
| am-config | DS | Admin | `uid=am-config,ou=admins,ou=am-config` | `platform8/{env}/ds/config_admin_password` | ✅ Yes |
| am-identity-bind-account | DS | Bind | `uid=am-identity-bind-account,ou=admins,ou=identities` | `platform8/{env}/ds/idrepo_admin_password` | ✅ Yes |
| openam_cts | DS | Admin | `uid=openam_cts,ou=admins,ou=famrecords,ou=openam-session,ou=tokens` | `platform8/{env}/ds/cts_admin_password` | ✅ Yes |
| amadmin | AM | Admin | `amadmin` | `platform8/{env}/am/admin_password` | ✅ Yes |
| openidm-admin | IDM | Internal | `internal/user/openidm-admin` | N/A (internal) | ✅ Yes |
| idm-provisioning | IDM | Internal | `internal/user/idm-provisioning` | N/A (internal) | ✅ Yes |
| idm-provisioning | AM | OAuth Client | `idm-provisioning` | Created in AM | ✅ Yes |
| fradmin | System | User | `fradmin` | N/A | ✅ Yes |

---

## Vault Secret Structure

All passwords should be stored in HashiCorp Vault using the following structure:

```
platform8/
├── dev/
│   ├── ds/
│   │   ├── admin_password
│   │   ├── deployment_id
│   │   ├── deployment_id_password
│   │   ├── config_admin_password
│   │   ├── idrepo_admin_password
│   │   └── cts_admin_password
│   ├── am/
│   │   ├── admin_password
│   │   ├── config_store_password
│   │   ├── identity_store_password
│   │   ├── cts_store_password
│   │   └── truststore_password
│   └── ig/
│       └── keystore_password
├── qa/
│   └── [same structure]
└── prod/
    └── [same structure]
```

---

## Account Creation Order

1. **DS Root Admin** - Created during DS installation
2. **DS Store Admins** - Created during DS profile setup (am-config, am-cts, am-identity-store)
3. **AM Admin** - Created during AM bootstrap (install-openam)
4. **IDM Internal Users** - Pre-configured in IDM
5. **OAuth Clients** - Created via AM REST API or Amster

---

## Security Best Practices

1. **Never store passwords in code or configuration files**
   - All passwords should be in HashiCorp Vault
   - Use Vault lookup plugins in Ansible

2. **Use strong, unique passwords**
   - Each account should have a unique password
   - Follow password policy requirements

3. **Rotate passwords regularly**
   - Implement password rotation policy
   - Update Vault secrets accordingly

4. **Limit account permissions**
   - Use least privilege principle
   - Separate admin accounts for different stores

5. **Monitor account usage**
   - Log all administrative operations
   - Audit account access regularly

---

## Troubleshooting

### Common Issues

1. **DS Bind Failures**
   - Verify DN format is correct
   - Check password in Vault
   - Ensure account exists in DS

2. **AM Connection Issues**
   - Verify store passwords match DS admin passwords
   - Check truststore contains DS CA certificates
   - Verify network connectivity

3. **IDM Provisioning Failures**
   - Verify OAuth2 client exists in AM
   - Check subject mapping configuration
   - Verify roles are assigned correctly

---

## Related Documentation

- **Variables Reference**: See `VARIABLES_REFERENCE.md`
- **Vault Setup**: See deployment guide
- **Account Configuration**: See component-specific playbooks

