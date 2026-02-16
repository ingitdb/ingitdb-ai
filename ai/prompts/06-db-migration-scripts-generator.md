# Prompt 6: DB Migration Scripts Generator

> **Agent**: Integration Engineer
> **Phase**: 2 — Server Implementation (extended)

Implement a DB migration scripts generator in ingitdb-go CLI with these requirements:

Purpose:
- Compare inGitDB data (source) against target database systems
- Generate migration scripts to synchronize data
- Support multiple target database types
- Handle schema and data differences

Supported Target Databases:
1. SQL Databases:
   - PostgreSQL
   - MySQL
   - SQLite
   - SQL Server

2. Key-Value Stores:
   - Redis
   - etcd
   - Consul KV

Core Features:
1. Schema Analysis:
   - Read inGitDB table schemas
   - Compare with target database schema
   - Detect additions, modifications, deletions
   - Generate CREATE TABLE, ALTER TABLE statements

2. Data Synchronization:
   - Compare record sets between inGitDB and target
   - Generate INSERT, UPDATE, DELETE statements
   - Handle bulk operations efficiently
   - Support incremental sync

3. Migration Script Generation:
   - Generate idempotent migration scripts
   - Include rollback scripts
   - Add transaction boundaries
   - Include verification queries

4. Configuration:
   - Target database connection settings
   - Mapping rules (table names, column types)
   - Transformation functions
   - Migration strategy (full, incremental)

CLI Commands:
```bash
# Analyze differences
ingitdb-go migrate analyze \
  --source ./data/mydb \
  --target postgres://localhost/mydb \
  --output diff-report.json

# Generate migration script
ingitdb-go migrate generate \
  --source ./data/mydb \
  --target postgres://localhost/mydb \
  --output migrate-up.sql \
  --rollback migrate-down.sql

# Apply migration
ingitdb-go migrate apply \
  --source ./data/mydb \
  --target postgres://localhost/mydb \
  --script migrate-up.sql \
  --verify

# Sync data (incremental)
ingitdb-go migrate sync \
  --source ./data/mydb \
  --target redis://localhost:6379 \
  --mode incremental \
  --batch-size 1000
```

Implementation Requirements:
- Support for custom type mappings
- Conflict resolution strategies
- Progress reporting for large datasets
- Dry-run mode
- Detailed logging
- Error handling and recovery
- Connection pooling for target databases

Testing:
- Unit tests for each database type
- Integration tests with real databases
- Test schema evolution scenarios
- Test large dataset migrations
- Test error scenarios and rollback
