# Prompt 10: Datatug Integration

> **Agent**: Integration Engineer
> **Phase**: 5 — Documentation & Polish (parallel)

Implement inGitDB metadata provider for Datatug with these requirements:

Overview:
- Provide inGitDB metadata to Datatug
- Enable viewing and exploring inGitDB data in Web UI
- Support CLI viewer integration
- Provide query builder capabilities

Repository: Can be part of ingitdb-go or separate plugin

Datatug Integration Points:
1. Metadata Provider:
   - Database structure (tables, columns)
   - Schema definitions
   - Relationships and constraints
   - Statistics and indexes

2. Data Access:
   - Query execution
   - Result set formatting
   - Pagination support
   - Export capabilities

3. Version Control Integration:
   - Show branch information
   - Display commit history
   - View historical data
   - Compare versions

Metadata Provider Implementation:
```go
package ingitdb_datatug

import (
    "context"
    "github.com/datatug/datatug/pkg/models"
)

// Provider implements datatug metadata provider interface
type Provider struct {
    ingitdbURL string
    database   string
}

// NewProvider creates an inGitDB provider for Datatug
func NewProvider(url, database string) *Provider {
    return &Provider{
        ingitdbURL: url,
        database:   database,
    }
}

// GetDatabaseInfo returns database metadata
func (p *Provider) GetDatabaseInfo(ctx context.Context) (*models.Database, error) {
    // Return database structure
}

// GetTables returns list of tables
func (p *Provider) GetTables(ctx context.Context) ([]*models.Table, error) {
    // Return table list with schemas
}

// GetTableInfo returns detailed table information
func (p *Provider) GetTableInfo(ctx context.Context, tableName string) (*models.Table, error) {
    // Return table schema, columns, constraints
}

// ExecuteQuery executes a query and returns results
func (p *Provider) ExecuteQuery(ctx context.Context, query string) (*models.QueryResult, error) {
    // Translate query and execute
}

// GetRecordCount returns count of records in table
func (p *Provider) GetRecordCount(ctx context.Context, tableName string) (int64, error) {
    // Return record count
}
```

Configuration (datatug.yaml):
```yaml
projects:
  - id: my-ingitdb
    title: My InGitDB Project

    environments:
      - id: production
        title: Production

        stores:
          - id: ingitdb-prod
            title: InGitDB Production
            driver: ingitdb
            parameters:
              url: http://ingitdb.example.com
              database: production
              branch: main
              api_key: ${INGITDB_API_KEY}

      - id: development
        title: Development

        stores:
          - id: ingitdb-dev
            title: InGitDB Development
            driver: ingitdb
            parameters:
              url: http://localhost:3000
              database: mydb
              branch: develop

# InGitDB-specific configuration
ingitdb:
  # Show version control information
  show_branches: true
  show_commits: true

  # Enable time-travel queries
  enable_historical_queries: true

  # Query builder settings
  query_builder:
    enable: true
    max_results: 1000
```

Datatug Web UI Features:
1. Database Explorer:
   - Tree view of databases and tables
   - Schema visualization
   - Relationship diagrams
   - Search capabilities

2. Data Browser:
   - Table data viewer with pagination
   - Column filtering and sorting
   - Record editing (if permissions allow)
   - Export to CSV/JSON/Excel

3. Query Builder:
   - Visual query builder
   - Support for filters and joins
   - Query history
   - Save and share queries

4. Version Control Integration:
   - Branch selector
   - Commit timeline
   - Diff viewer for data changes
   - Time-travel queries (view data at specific commit)

5. Schema Management:
   - View and edit schemas
   - Generate schema documentation
   - Track schema changes
   - Validate schema consistency

CLI Integration:
```bash
# Install datatug with inGitDB support
datatug install-driver ingitdb

# Configure inGitDB connection
datatug project add \
  --id my-project \
  --store ingitdb \
  --url http://localhost:3000 \
  --database mydb

# Explore database structure
datatug explore my-project --store ingitdb

# Browse table data
datatug browse my-project.users --limit 50

# Execute query
datatug query my-project "SELECT * FROM users WHERE active = true"

# View schema
datatug schema my-project.users

# Show version history
datatug history my-project.users --record-id user1

# Compare branches
datatug diff my-project --base main --compare feature
```

InGitDB-Specific Features in Datatug:
```typescript
// Custom UI components for inGitDB

// Branch selector
<BranchSelector
  branches={branches}
  current={currentBranch}
  onSwitch={handleBranchSwitch}
/>

// Commit timeline
<CommitTimeline
  commits={commits}
  onSelectCommit={handleSelectCommit}
/>

// Time-travel query
<TimeTravelQuery
  table="users"
  commit={selectedCommit}
  query={query}
/>

// Diff viewer
<DiffViewer
  baseBranch="main"
  compareBranch="feature"
  table="users"
  recordId="user1"
/>

// Merge conflict resolver
<MergeConflictResolver
  conflicts={conflicts}
  onResolve={handleResolve}
/>
```

API Endpoints (if extending Datatug API):
```
GET /api/ingitdb/databases - List databases
GET /api/ingitdb/databases/{db} - Get database info
GET /api/ingitdb/databases/{db}/branches - List branches
GET /api/ingitdb/databases/{db}/commits - List commits
GET /api/ingitdb/databases/{db}/tables - List tables
GET /api/ingitdb/databases/{db}/tables/{table} - Get table schema
GET /api/ingitdb/databases/{db}/tables/{table}/data - Browse data
POST /api/ingitdb/databases/{db}/query - Execute query
GET /api/ingitdb/databases/{db}/diff - Compare branches
```

Implementation Requirements:
- Implement Datatug provider interface
- Support for inGitDB-specific features
- Efficient data fetching
- Caching for metadata
- WebSocket support for real-time updates
- Export functionality
- Security (respect inGitDB permissions)

Testing:
- Unit tests for provider implementation
- Integration tests with Datatug
- UI tests for custom components
- Performance tests with large datasets
- Security tests (permission enforcement)

Documentation:
- Installation guide
- Configuration examples
- Usage tutorials
- Screenshots and videos
- Troubleshooting guide
