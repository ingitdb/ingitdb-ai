# Prompt 8: Access Management Policy

> **Agent**: Integration Engineer
> **Phase**: 2 — Server Implementation (extended)

Implement a comprehensive access management system for inGitDB with these requirements:

Overview:
- Role-based access control (RBAC)
- Fine-grained permissions
- Policy-based authorization
- Integration with identity providers

Permission Levels:
1. Database Level:
   - read: Read database metadata and tables
   - write: Create/update/delete tables
   - admin: Full control including triggers and policies

2. Table Level:
   - read: Read table schema and records
   - insert: Insert new records
   - update: Update existing records
   - delete: Delete records
   - schema_update: Modify table schema

3. Record Level:
   - read: Read specific records
   - update: Update specific records
   - delete: Delete specific records

4. Branch Level:
   - read: Read branch
   - write: Push to branch
   - merge: Merge branches
   - create: Create new branches

Role Definitions (.ingitdb-policy.yaml):
```yaml
roles:
  # Predefined roles
  admin:
    description: Full administrative access
    permissions:
      - database:*:admin
      - table:*:*
      - branch:*:*

  developer:
    description: Read and write data, create branches
    permissions:
      - database:*:read
      - database:*:write
      - table:*:read
      - table:*:insert
      - table:*:update
      - branch:*:read
      - branch:*:write
      - branch:*:create

  viewer:
    description: Read-only access
    permissions:
      - database:*:read
      - table:*:read
      - branch:main:read

  # Custom roles
  data_analyst:
    description: Read data, cannot modify schemas
    permissions:
      - database:analytics:read
      - table:analytics/*:read
      - table:analytics/*:insert
      - table:analytics/*:update
    restrictions:
      - "!table:analytics/*:schema_update"
      - "!table:analytics/*:delete"

# User-role assignments
users:
  alice@example.com:
    roles: [admin]

  bob@example.com:
    roles: [developer]
    databases: [myapp, staging]

  charlie@example.com:
    roles: [data_analyst]

  team-api@example.com:
    roles: [viewer]
    api_key: true

# Policies
policies:
  # Deny delete on production branch
  - name: protect-production
    effect: deny
    actions: [delete]
    resources:
      - branch:production
      - table:*/production/*

  # Allow only specific users to merge to main
  - name: main-branch-protection
    effect: allow
    actions: [merge]
    resources:
      - branch:main
    principals:
      - alice@example.com
      - bob@example.com

  # Record-level policy
  - name: user-own-records
    effect: allow
    actions: [update, delete]
    resources:
      - table:users/records/*
    conditions:
      - "record.owner == user.email"
```

Authentication Methods:
1. API Keys:
   - Generate and manage API keys
   - Key rotation policies
   - Scope-limited keys

2. JWT Tokens:
   - Support standard JWT claims
   - Custom claims for permissions
   - Token validation and refresh

3. OAuth 2.0 / OIDC:
   - Integration with identity providers
   - Support for GitHub, Google, Azure AD
   - SSO capabilities

4. Basic Auth:
   - Username/password (for CLI)
   - Secure password storage

CLI Commands:
```bash
# Role management
ingitdb-go policy roles list
ingitdb-go policy roles create --name analyst --from-file role.yaml
ingitdb-go policy roles assign --user alice@example.com --role admin

# User management
ingitdb-go policy users list
ingitdb-go policy users create --email bob@example.com --role developer
ingitdb-go policy users revoke --email charlie@example.com --role admin

# API key management
ingitdb-go policy apikeys create --user bob@example.com --scope "database:myapp:read"
ingitdb-go policy apikeys list --user bob@example.com
ingitdb-go policy apikeys revoke --key-id abc123

# Policy testing
ingitdb-go policy check \
  --user bob@example.com \
  --action update \
  --resource table:myapp/users/record/123
```

Server API Endpoints:
- POST /auth/login - Authenticate user
- POST /auth/token/refresh - Refresh token
- GET /auth/user - Get current user info
- GET /policies/roles - List roles
- POST /policies/roles - Create role
- GET /policies/users - List users
- POST /policies/users - Create user
- PUT /policies/users/{id}/roles - Assign roles
- GET /policies/check - Check permissions

Implementation Requirements:
- Policy evaluation engine
- Permission caching for performance
- Audit logging of access attempts
- Integration with identity providers
- Secure token storage
- Password hashing (bcrypt/argon2)
- Rate limiting per user/key
- Session management

Security Features:
- Password complexity requirements
- Account lockout after failed attempts
- Two-factor authentication (optional)
- IP whitelisting (optional)
- Audit trail for all access
- Secure credential storage

Testing:
- Unit tests for policy evaluation
- Integration tests with auth providers
- Test permission inheritance
- Test policy conflicts
- Security testing (bypass attempts)
- Performance tests (policy evaluation)
