# Prompt 7: inGitDB Triggers System

> **Agent**: Integration Engineer
> **Phase**: 2 — Server Implementation (extended)

Implement a triggers system for inGitDB with these requirements:

Overview:
- Execute actions on create, update, delete operations
- Support CLI commands and webhook calls
- Configure triggers declaratively
- Provide pre- and post-operation hooks

Trigger Types:
1. CLI Command Triggers:
   - Execute shell commands
   - Pass operation context as arguments
   - Capture output and exit codes
   - Support async execution

2. Webhook Triggers:
   - HTTP POST to configured URLs
   - Send operation payload as JSON
   - Handle authentication (API keys, JWT)
   - Retry with exponential backoff

3. Script Triggers:
   - Execute custom scripts (bash, python, node)
   - Provide operation context
   - Sandbox execution environment
   - Resource limits

Trigger Configuration (.ingitdb-triggers.yaml):
```yaml
triggers:
  # Before record insert
  - name: validate-user-email
    event: before_insert
    table: users
    type: cli
    command: ./scripts/validate-email.sh
    args:
      - "${record.email}"
    failOnError: true

  # After record update
  - name: notify-user-update
    event: after_update
    table: users
    type: webhook
    url: https://api.example.com/webhooks/user-updated
    method: POST
    headers:
      Authorization: "Bearer ${env.API_TOKEN}"
    payload:
      userId: "${record.id}"
      changes: "${changes}"
      timestamp: "${timestamp}"
    retries: 3

  # Before delete
  - name: archive-before-delete
    event: before_delete
    table: orders
    type: script
    script: ./scripts/archive-order.py
    interpreter: python3
    timeout: 30s

  # After commit
  - name: sync-to-cache
    event: after_commit
    database: mydb
    type: webhook
    url: https://cache.example.com/invalidate
    async: true
```

CLI Integration:
```bash
# List configured triggers
ingitdb-go triggers list

# Test a trigger
ingitdb-go triggers test validate-user-email \
  --data '{"email":"test@example.com"}'

# Enable/disable triggers
ingitdb-go triggers enable validate-user-email
ingitdb-go triggers disable notify-user-update

# View trigger execution history
ingitdb-go triggers history --table users --limit 50
```

Server API Endpoints:
- GET /databases/{db}/triggers - List triggers
- POST /databases/{db}/triggers - Create trigger
- PUT /databases/{db}/triggers/{id} - Update trigger
- DELETE /databases/{db}/triggers/{id} - Delete trigger
- POST /databases/{db}/triggers/{id}/test - Test trigger
- GET /databases/{db}/triggers/{id}/history - Execution history

Events:
- before_insert, after_insert
- before_update, after_update
- before_delete, after_delete
- before_commit, after_commit
- before_merge, after_merge
- before_branch, after_branch

Operation Context:
- record: Current record data
- changes: Modified fields (for updates)
- oldRecord: Previous record data (for updates/deletes)
- operation: Operation type
- table: Table name
- database: Database name
- branch: Current branch
- user: User performing operation
- timestamp: Operation timestamp

Implementation Requirements:
- Trigger execution engine
- Context variable substitution
- Error handling (fail-fast vs. continue)
- Execution logs and audit trail
- Performance monitoring
- Rate limiting for webhooks
- Security sandboxing for scripts
- Circular dependency detection

Testing:
- Unit tests for each trigger type
- Integration tests with real webhooks
- Test error scenarios
- Test retry logic
- Performance tests
- Security tests (script isolation)
