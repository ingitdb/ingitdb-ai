# Prompt 3: TypeScript Client Development

> **Agent**: TypeScript Client Developer
> **Phase**: 3 — TypeScript Client

Create a TypeScript client library (ingitdb-ts) for the inGitDB server with these requirements:

Core Features:
1. Auto-generate base client from OpenAPI specification
2. Create high-level, ergonomic wrapper API
3. Full TypeScript type safety
4. Connection pooling and management
5. Automatic retry with exponential backoff
6. Comprehensive error handling

API Design:
```typescript
// Example usage
import { InGitDB } from 'ingitdb-ts';

const db = new InGitDB({
  baseUrl: 'http://localhost:3000',
  apiKey: 'your-api-key',
  timeout: 5000
});

// Database operations
await db.databases.create('mydb');
const databases = await db.databases.list();

// Table operations
await db.database('mydb').tables.create('users', {
  schema: {
    type: 'object',
    properties: {
      id: { type: 'string' },
      name: { type: 'string' },
      email: { type: 'string', format: 'email' }
    },
    required: ['id', 'name']
  }
});

// Record operations
await db.database('mydb').table('users').records.insert({
  id: '1',
  name: 'John Doe',
  email: 'john@example.com'
});

const users = await db.database('mydb').table('users').records.query({
  filter: { name: { $contains: 'John' } },
  limit: 10,
  offset: 0
});

// Version control
await db.database('mydb').branches.create('feature-branch');
await db.database('mydb').branches.switch('feature-branch');
const commits = await db.database('mydb').commits.list();
```

Requirements:
- Generate from OpenAPI spec using openapi-generator-cli
- Builder pattern for query construction
- Stream support for large result sets
- WebSocket support for real-time updates
- Comprehensive JSDoc documentation
- Export all types
- Bundle for both Node.js and browsers
- Tree-shakeable exports

Testing:
- Unit tests for all methods
- Mock server for integration tests
- Type tests to ensure type safety
