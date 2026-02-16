# Prompt 1: OpenAPI Specification Design

> **Agent**: API Architect
> **Phase**: 1 — Foundation

Create a comprehensive OpenAPI 3.x specification for inGitDB server API with the following requirements:

Core Concepts:
- Database: A named collection of tables (stored as a directory)
- Table: A collection of records (stored as JSON/YAML/columnar files)
- Record: Individual data entries with schema validation
- Branch: Git branch for versioning data
- Commit: Git commit representing data state

Required Endpoints:

Database Operations:
- POST /databases - Create a new database
- GET /databases - List all databases
- GET /databases/{dbName} - Get database details
- DELETE /databases/{dbName} - Delete a database

Table Operations:
- POST /databases/{dbName}/tables - Create a table
- GET /databases/{dbName}/tables - List tables
- GET /databases/{dbName}/tables/{tableName} - Get table schema
- PUT /databases/{dbName}/tables/{tableName} - Update table schema
- DELETE /databases/{dbName}/tables/{tableName} - Delete table

Record Operations:
- POST /databases/{dbName}/tables/{tableName}/records - Insert records
- GET /databases/{dbName}/tables/{tableName}/records - Query records
- GET /databases/{dbName}/tables/{tableName}/records/{recordId} - Get record
- PUT /databases/{dbName}/tables/{tableName}/records/{recordId} - Update record
- DELETE /databases/{dbName}/tables/{tableName}/records/{recordId} - Delete record

Version Control Operations:
- GET /databases/{dbName}/branches - List branches
- POST /databases/{dbName}/branches - Create branch
- GET /databases/{dbName}/commits - List commits
- GET /databases/{dbName}/commits/{sha} - Get commit details
- POST /databases/{dbName}/merge - Merge branches

Include:
- Comprehensive schemas for all request/response bodies
- Error response definitions
- Authentication via Bearer token
- Query parameters for filtering, pagination, sorting
- Support for JSON and YAML content types
