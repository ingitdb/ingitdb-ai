# Prompt 2: Server Implementation

> **Agent**: Server Backend Developer
> **Phase**: 2 — Server Implementation

Implement the inGitDB server based on the OpenAPI specification with these requirements:

Technology Stack:
- Use Node.js with TypeScript (or Go if preferred)
- Express or Fastify framework
- libgit2 bindings (nodegit or go-git) for Git operations
- JSON Schema validation
- JWT for authentication

Core Features:
1. Database Management:
   - Create/delete database (as Git repository or subdirectory)
   - Initialize with .gitignore and README
   - Track metadata in index files

2. Table Management:
   - Store table data in JSON/YAML files
   - Maintain schema definitions
   - Validate data against schemas
   - Support multiple storage formats

3. Record Operations:
   - CRUD operations on records
   - Atomic commits for each operation
   - Query language support (simple filtering)
   - Pagination and sorting

4. Version Control:
   - Git commit on each data change
   - Branch creation and switching
   - Merge with conflict detection
   - History and diff operations

5. Validation:
   - Schema validation before writes
   - Reference integrity checks
   - Custom validation rules

Implementation Requirements:
- Follow OpenAPI specification exactly
- Comprehensive error handling
- Logging with structured logs
- Configuration via environment variables
- Docker support
- Health check endpoints
- Metrics endpoint (Prometheus format)

Testing:
- Unit tests with 80%+ coverage
- Integration tests for all endpoints
- Git operation tests
- Concurrency tests
