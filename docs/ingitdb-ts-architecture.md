# ingitdb-ts - TypeScript Client Architecture

## Overview

The `ingitdb-ts` package is a type-safe TypeScript client library for interacting with the inGitDB server API. It provides a high-level, ergonomic interface for all database operations with full TypeScript support, automatic retries, and comprehensive error handling.

## Design Goals

1. **Type Safety**: Full TypeScript support with generated types from OpenAPI specification
2. **Developer Experience**: Intuitive, chainable API design
3. **Error Handling**: Comprehensive error handling with custom error types
4. **Performance**: Connection pooling, request batching, and caching
5. **Testing**: Easy to mock and test
6. **Compatibility**: Support both Node.js and browser environments
7. **Size**: Tree-shakeable with minimal bundle size

## Architecture

### Layer Structure

```
┌─────────────────────────────────────┐
│   Public API (Fluent Interface)     │  ← User-facing API
├─────────────────────────────────────┤
│   Business Logic Layer               │  ← Query builders, validation
├─────────────────────────────────────┤
│   HTTP Client Layer                  │  ← Request/response handling
├─────────────────────────────────────┤
│   Generated OpenAPI Client           │  ← Auto-generated from spec
└─────────────────────────────────────┘
```

### Directory Structure

```
ingitdb-ts/
├── src/
│   ├── index.ts                    # Main entry point
│   ├── client.ts                   # Main InGitDB client class
│   ├── config.ts                   # Configuration types and defaults
│   ├── types/                      # TypeScript type definitions
│   │   ├── index.ts                # Re-exports all types
│   │   ├── database.ts             # Database-related types
│   │   ├── table.ts                # Table-related types
│   │   ├── record.ts               # Record-related types
│   │   ├── version-control.ts      # Git-related types
│   │   └── query.ts                # Query builder types
│   ├── resources/                  # Resource-specific classes
│   │   ├── database.ts             # Database operations
│   │   ├── table.ts                # Table operations
│   │   ├── record.ts               # Record CRUD operations
│   │   ├── branch.ts               # Branch operations
│   │   └── commit.ts               # Commit operations
│   ├── query/                      # Query builder implementation
│   │   ├── filter.ts               # Filter builder
│   │   ├── sort.ts                 # Sort builder
│   │   └── query-builder.ts        # Main query builder
│   ├── http/                       # HTTP client implementation
│   │   ├── client.ts               # HTTP client wrapper
│   │   ├── retry.ts                # Retry logic
│   │   └── interceptors.ts         # Request/response interceptors
│   ├── errors/                     # Custom error classes
│   │   ├── base.ts                 # Base error class
│   │   ├── api-error.ts            # API-specific errors
│   │   ├── validation-error.ts     # Validation errors
│   │   └── network-error.ts        # Network errors
│   ├── utils/                      # Utility functions
│   │   ├── validation.ts           # Input validation
│   │   ├── serialization.ts        # JSON/YAML serialization
│   │   └── helpers.ts              # General helpers
│   └── generated/                  # Auto-generated OpenAPI client
│       ├── api.ts                  # Generated API client
│       └── models.ts               # Generated TypeScript models
├── test/
│   ├── unit/                       # Unit tests
│   ├── integration/                # Integration tests
│   └── fixtures/                   # Test data
├── examples/                       # Usage examples
│   ├── basic-usage.ts
│   ├── advanced-queries.ts
│   ├── version-control.ts
│   └── error-handling.ts
├── docs/                           # Documentation
│   ├── api/                        # API reference
│   └── guides/                     # User guides
├── package.json
├── tsconfig.json
├── rollup.config.js                # Build configuration
├── .eslintrc.js
├── .prettierrc
└── README.md
```

## Core Components

### 1. Main Client Class

```typescript
// src/client.ts
import { InGitDBConfig, DefaultConfig } from './config';
import { DatabaseResource } from './resources/database';
import { HttpClient } from './http/client';

export class InGitDB {
  private httpClient: HttpClient;
  private config: InGitDBConfig;

  constructor(config: Partial<InGitDBConfig>) {
    this.config = { ...DefaultConfig, ...config };
    this.httpClient = new HttpClient(this.config);
  }

  /**
   * Access database operations
   */
  get databases(): DatabaseResource {
    return new DatabaseResource(this.httpClient);
  }

  /**
   * Access a specific database
   */
  database(name: string): DatabaseOperations {
    return new DatabaseOperations(name, this.httpClient);
  }

  /**
   * Health check
   */
  async health(): Promise<HealthStatus> {
    return this.httpClient.get('/health');
  }
}
```

### 2. Configuration

```typescript
// src/config.ts
export interface InGitDBConfig {
  /**
   * Base URL of the inGitDB server
   * @example 'https://api.ingitdb.io/v1'
   */
  baseUrl: string;

  /**
   * API authentication token
   */
  apiKey?: string;

  /**
   * Request timeout in milliseconds
   * @default 5000
   */
  timeout?: number;

  /**
   * Retry configuration
   */
  retry?: {
    /**
     * Maximum number of retries
     * @default 3
     */
    maxRetries?: number;
    
    /**
     * Initial retry delay in milliseconds
     * @default 1000
     */
    retryDelay?: number;
    
    /**
     * Exponential backoff multiplier
     * @default 2
     */
    backoffMultiplier?: number;
    
    /**
     * HTTP status codes to retry
     * @default [408, 429, 500, 502, 503, 504]
     */
    retryableStatusCodes?: number[];
  };

  /**
   * Custom headers to include in all requests
   */
  headers?: Record<string, string>;

  /**
   * Enable debug logging
   * @default false
   */
  debug?: boolean;
}

export const DefaultConfig: Partial<InGitDBConfig> = {
  timeout: 5000,
  retry: {
    maxRetries: 3,
    retryDelay: 1000,
    backoffMultiplier: 2,
    retryableStatusCodes: [408, 429, 500, 502, 503, 504],
  },
  debug: false,
};
```

### 3. Resource Classes

```typescript
// src/resources/database.ts
import { HttpClient } from '../http/client';
import { Database, CreateDatabaseRequest } from '../types';

export class DatabaseResource {
  constructor(private httpClient: HttpClient) {}

  /**
   * List all databases
   */
  async list(options?: ListOptions): Promise<PaginatedResponse<Database>> {
    const params = new URLSearchParams();
    if (options?.page) params.append('page', options.page.toString());
    if (options?.limit) params.append('limit', options.limit.toString());
    
    return this.httpClient.get(`/databases?${params}`);
  }

  /**
   * Create a new database
   */
  async create(request: CreateDatabaseRequest): Promise<Database> {
    return this.httpClient.post('/databases', request);
  }

  /**
   * Get database by name
   */
  async get(name: string): Promise<Database> {
    return this.httpClient.get(`/databases/${name}`);
  }

  /**
   * Delete a database
   */
  async delete(name: string): Promise<void> {
    return this.httpClient.delete(`/databases/${name}`);
  }
}

// src/resources/database-operations.ts
export class DatabaseOperations {
  constructor(
    private dbName: string,
    private httpClient: HttpClient
  ) {}

  /**
   * Access table operations
   */
  get tables(): TableResource {
    return new TableResource(this.dbName, this.httpClient);
  }

  /**
   * Access a specific table
   */
  table(tableName: string): TableOperations {
    return new TableOperations(this.dbName, tableName, this.httpClient);
  }

  /**
   * Access branch operations
   */
  get branches(): BranchResource {
    return new BranchResource(this.dbName, this.httpClient);
  }

  /**
   * Access commit history
   */
  get commits(): CommitResource {
    return new CommitResource(this.dbName, this.httpClient);
  }

  /**
   * Merge branches
   */
  async merge(request: MergeRequest): Promise<MergeResult> {
    return this.httpClient.post(`/databases/${this.dbName}/merge`, request);
  }

  /**
   * Switch branch
   */
  async checkout(branchName: string): Promise<CheckoutResult> {
    return this.httpClient.post(`/databases/${this.dbName}/checkout`, {
      branch: branchName,
    });
  }
}
```

### 4. Query Builder

```typescript
// src/query/query-builder.ts
export class QueryBuilder<T = any> {
  private filters: FilterExpression[] = [];
  private sortFields: SortExpression[] = [];
  private selectedFields: string[] = [];
  private pageNumber: number = 1;
  private pageLimit: number = 20;

  /**
   * Add a filter condition
   */
  where(field: string, operator: Operator, value: any): this {
    this.filters.push({ field, operator, value });
    return this;
  }

  /**
   * Add an equals filter (shorthand)
   */
  whereEquals(field: string, value: any): this {
    return this.where(field, '=', value);
  }

  /**
   * Add a greater than filter
   */
  whereGreaterThan(field: string, value: any): this {
    return this.where(field, '>', value);
  }

  /**
   * Add a contains filter (for string fields)
   */
  whereContains(field: string, value: string): this {
    return this.where(field, 'contains', value);
  }

  /**
   * Sort by field
   */
  sortBy(field: string, direction: 'asc' | 'desc' = 'asc'): this {
    this.sortFields.push({ field, direction });
    return this;
  }

  /**
   * Select specific fields
   */
  select(...fields: string[]): this {
    this.selectedFields = fields;
    return this;
  }

  /**
   * Set page number
   */
  page(page: number): this {
    this.pageNumber = page;
    return this;
  }

  /**
   * Set page limit
   */
  limit(limit: number): this {
    this.pageLimit = limit;
    return this;
  }

  /**
   * Build query parameters
   */
  build(): QueryParams {
    const params: QueryParams = {
      page: this.pageNumber,
      limit: this.pageLimit,
    };

    if (this.filters.length > 0) {
      params.filter = JSON.stringify(this.buildFilterObject());
    }

    if (this.sortFields.length > 0) {
      params.sort = this.sortFields
        .map(s => `${s.field}:${s.direction}`)
        .join(',');
    }

    if (this.selectedFields.length > 0) {
      params.fields = this.selectedFields.join(',');
    }

    return params;
  }

  private buildFilterObject(): any {
    // Convert filter expressions to MongoDB-style query object
    const filter: any = {};
    
    for (const f of this.filters) {
      const operator = this.mapOperator(f.operator);
      
      if (operator === '=') {
        filter[f.field] = f.value;
      } else {
        filter[f.field] = { [operator]: f.value };
      }
    }
    
    return filter;
  }

  private mapOperator(operator: Operator): string {
    const operatorMap: Record<Operator, string> = {
      '=': '=',
      '>': '$gt',
      '>=': '$gte',
      '<': '$lt',
      '<=': '$lte',
      '!=': '$ne',
      'contains': '$contains',
      'in': '$in',
      'not_in': '$nin',
    };
    
    return operatorMap[operator];
  }
}
```

### 5. Record Operations with Query Builder

```typescript
// src/resources/record.ts
export class RecordResource<T = any> {
  constructor(
    private dbName: string,
    private tableName: string,
    private httpClient: HttpClient
  ) {}

  /**
   * Create a query builder
   */
  query(): RecordQuery<T> {
    return new RecordQuery<T>(this.dbName, this.tableName, this.httpClient);
  }

  /**
   * Get all records (with optional simple filtering)
   */
  async list(options?: ListOptions): Promise<PaginatedResponse<Record<T>>> {
    const params = new URLSearchParams();
    if (options?.page) params.append('page', options.page.toString());
    if (options?.limit) params.append('limit', options.limit.toString());
    
    return this.httpClient.get(
      `/databases/${this.dbName}/tables/${this.tableName}/records?${params}`
    );
  }

  /**
   * Get a single record by ID
   */
  async get(id: string): Promise<Record<T>> {
    return this.httpClient.get(
      `/databases/${this.dbName}/tables/${this.tableName}/records/${id}`
    );
  }

  /**
   * Insert one or more records
   */
  async insert(
    records: T | T[],
    options?: InsertOptions
  ): Promise<InsertRecordsResponse<T>> {
    const recordArray = Array.isArray(records) ? records : [records];
    
    return this.httpClient.post(
      `/databases/${this.dbName}/tables/${this.tableName}/records`,
      {
        records: recordArray,
        commit_message: options?.commitMessage,
      }
    );
  }

  /**
   * Update a record (full replacement)
   */
  async update(
    id: string,
    data: T,
    options?: UpdateOptions
  ): Promise<Record<T>> {
    return this.httpClient.put(
      `/databases/${this.dbName}/tables/${this.tableName}/records/${id}`,
      {
        data,
        commit_message: options?.commitMessage,
      }
    );
  }

  /**
   * Patch a record (partial update)
   */
  async patch(
    id: string,
    data: Partial<T>,
    options?: UpdateOptions
  ): Promise<Record<T>> {
    return this.httpClient.patch(
      `/databases/${this.dbName}/tables/${this.tableName}/records/${id}`,
      {
        data,
        commit_message: options?.commitMessage,
      }
    );
  }

  /**
   * Delete a record
   */
  async delete(id: string): Promise<void> {
    return this.httpClient.delete(
      `/databases/${this.dbName}/tables/${this.tableName}/records/${id}`
    );
  }
}

// Extended query class for executing queries
export class RecordQuery<T = any> extends QueryBuilder<T> {
  constructor(
    private dbName: string,
    private tableName: string,
    private httpClient: HttpClient
  ) {
    super();
  }

  /**
   * Execute the query
   */
  async execute(): Promise<PaginatedResponse<Record<T>>> {
    const params = this.build();
    const queryString = new URLSearchParams(params as any).toString();
    
    return this.httpClient.get(
      `/databases/${this.dbName}/tables/${this.tableName}/records?${queryString}`
    );
  }

  /**
   * Get the first result
   */
  async first(): Promise<Record<T> | null> {
    const result = await this.limit(1).execute();
    return result.records[0] || null;
  }

  /**
   * Get all results (handles pagination automatically)
   */
  async all(): Promise<Record<T>[]> {
    const allRecords: Record<T>[] = [];
    let currentPage = 1;
    let hasMore = true;

    while (hasMore) {
      const result = await this.page(currentPage).execute();
      allRecords.push(...result.records);
      
      hasMore = currentPage < result.pagination.total_pages;
      currentPage++;
    }

    return allRecords;
  }

  /**
   * Count total matching records
   */
  async count(): Promise<number> {
    const result = await this.limit(1).execute();
    return result.pagination.total;
  }
}
```

### 6. Error Handling

```typescript
// src/errors/base.ts
export class InGitDBError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode?: number,
    public details?: any
  ) {
    super(message);
    this.name = 'InGitDBError';
    Object.setPrototypeOf(this, InGitDBError.prototype);
  }
}

// src/errors/api-error.ts
export class ApiError extends InGitDBError {
  constructor(
    message: string,
    code: string,
    statusCode: number,
    details?: any
  ) {
    super(message, code, statusCode, details);
    this.name = 'ApiError';
    Object.setPrototypeOf(this, ApiError.prototype);
  }

  static fromResponse(response: any): ApiError {
    return new ApiError(
      response.error.message,
      response.error.code,
      response.status,
      response.error.details
    );
  }
}

// src/errors/validation-error.ts
export class ValidationError extends InGitDBError {
  constructor(message: string, details?: any) {
    super(message, 'VALIDATION_ERROR', 400, details);
    this.name = 'ValidationError';
    Object.setPrototypeOf(this, ValidationError.prototype);
  }
}

// src/errors/network-error.ts
export class NetworkError extends InGitDBError {
  constructor(message: string, public originalError?: Error) {
    super(message, 'NETWORK_ERROR');
    this.name = 'NetworkError';
    Object.setPrototypeOf(this, NetworkError.prototype);
  }
}
```

### 7. HTTP Client with Retry Logic

```typescript
// src/http/client.ts
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios';
import { InGitDBConfig } from '../config';
import { RetryHandler } from './retry';
import { ApiError, NetworkError } from '../errors';

export class HttpClient {
  private axiosInstance: AxiosInstance;
  private retryHandler: RetryHandler;

  constructor(private config: InGitDBConfig) {
    this.axiosInstance = axios.create({
      baseURL: config.baseUrl,
      timeout: config.timeout,
      headers: {
        'Content-Type': 'application/json',
        ...config.headers,
      },
    });

    // Add auth interceptor
    if (config.apiKey) {
      this.axiosInstance.interceptors.request.use((config) => {
        config.headers.Authorization = `Bearer ${this.config.apiKey}`;
        return config;
      });
    }

    // Add response interceptor for error handling
    this.axiosInstance.interceptors.response.use(
      (response) => response,
      (error) => this.handleError(error)
    );

    this.retryHandler = new RetryHandler(config.retry || {});
  }

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    return this.request('GET', url, undefined, config);
  }

  async post<T>(
    url: string,
    data?: any,
    config?: AxiosRequestConfig
  ): Promise<T> {
    return this.request('POST', url, data, config);
  }

  async put<T>(
    url: string,
    data?: any,
    config?: AxiosRequestConfig
  ): Promise<T> {
    return this.request('PUT', url, data, config);
  }

  async patch<T>(
    url: string,
    data?: any,
    config?: AxiosRequestConfig
  ): Promise<T> {
    return this.request('PATCH', url, data, config);
  }

  async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    return this.request('DELETE', url, undefined, config);
  }

  private async request<T>(
    method: string,
    url: string,
    data?: any,
    config?: AxiosRequestConfig
  ): Promise<T> {
    return this.retryHandler.execute(async () => {
      try {
        const response = await this.axiosInstance.request<T>({
          method,
          url,
          data,
          ...config,
        });
        
        return response.data;
      } catch (error) {
        throw this.handleError(error);
      }
    });
  }

  private handleError(error: any): Error {
    if (axios.isAxiosError(error)) {
      if (error.response) {
        // Server responded with error status
        return ApiError.fromResponse({
          error: error.response.data.error,
          status: error.response.status,
        });
      } else if (error.request) {
        // Request made but no response
        return new NetworkError('No response from server', error);
      }
    }
    
    return new NetworkError('Network request failed', error);
  }
}
```

## Usage Examples

### Basic Usage

```typescript
import { InGitDB } from 'ingitdb-ts';

// Initialize client
const db = new InGitDB({
  baseUrl: 'http://localhost:3000/api/v1',
  apiKey: 'your-api-key',
});

// Create a database
await db.databases.create({
  name: 'my-database',
  description: 'My first database',
  storage_format: 'json',
});

// Create a table
await db.database('my-database').tables.create({
  name: 'users',
  schema: {
    type: 'object',
    properties: {
      id: { type: 'string' },
      name: { type: 'string' },
      email: { type: 'string', format: 'email' },
      age: { type: 'number', minimum: 0 },
    },
    required: ['id', 'name', 'email'],
  },
});

// Insert records
await db.database('my-database').table('users').records.insert([
  { id: '1', name: 'John Doe', email: 'john@example.com', age: 30 },
  { id: '2', name: 'Jane Smith', email: 'jane@example.com', age: 25 },
], { commitMessage: 'Added initial users' });
```

### Advanced Queries

```typescript
// Query with filters
const adults = await db
  .database('my-database')
  .table('users')
  .records
  .query()
  .whereGreaterThan('age', 18)
  .whereContains('name', 'John')
  .sortBy('age', 'desc')
  .select('id', 'name', 'age')
  .limit(10)
  .execute();

// Get first matching record
const user = await db
  .database('my-database')
  .table('users')
  .records
  .query()
  .whereEquals('email', 'john@example.com')
  .first();

// Count records
const count = await db
  .database('my-database')
  .table('users')
  .records
  .query()
  .whereGreaterThan('age', 18)
  .count();
```

### Version Control

```typescript
// Create a branch
await db.database('my-database').branches.create({
  name: 'feature-branch',
  checkout: true,
});

// Make changes on the branch
await db.database('my-database').table('users').records.insert({
  id: '3',
  name: 'Bob Wilson',
  email: 'bob@example.com',
  age: 35,
});

// Switch back to main
await db.database('my-database').checkout('main');

// Merge feature branch
const result = await db.database('my-database').merge({
  source_branch: 'feature-branch',
  target_branch: 'main',
  commit_message: 'Merge feature branch',
  strategy: 'merge',
});

// View commit history
const commits = await db.database('my-database').commits.list({
  limit: 10,
});
```

### Error Handling

```typescript
import { ApiError, ValidationError, NetworkError } from 'ingitdb-ts';

try {
  await db.database('my-database').table('users').records.insert({
    id: '4',
    name: 'Invalid User',
    // Missing required email field
  });
} catch (error) {
  if (error instanceof ValidationError) {
    console.error('Validation failed:', error.message);
    console.error('Details:', error.details);
  } else if (error instanceof ApiError) {
    console.error('API error:', error.code, error.message);
    console.error('Status:', error.statusCode);
  } else if (error instanceof NetworkError) {
    console.error('Network error:', error.message);
  } else {
    console.error('Unknown error:', error);
  }
}
```

## Type Safety

### Generic Type Support

```typescript
// Define your data types
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

interface Product {
  id: string;
  name: string;
  price: number;
  category: string;
}

// Use typed operations
const userTable = db.database('my-database').table<User>('users');

// TypeScript knows the record type
const users: PaginatedResponse<Record<User>> = await userTable.records.list();

// Type-safe queries
const query = userTable.records
  .query()
  .whereGreaterThan('age', 18)  // TypeScript validates field names
  .sortBy('name', 'asc');        // and types
```

## Build Configuration

### package.json

```json
{
  "name": "ingitdb-ts",
  "version": "1.0.0",
  "description": "TypeScript client for inGitDB",
  "main": "dist/cjs/index.js",
  "module": "dist/esm/index.js",
  "types": "dist/types/index.d.ts",
  "exports": {
    ".": {
      "require": "./dist/cjs/index.js",
      "import": "./dist/esm/index.js",
      "types": "./dist/types/index.d.ts"
    }
  },
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "npm run build:cjs && npm run build:esm && npm run build:types",
    "build:cjs": "tsc --module commonjs --outDir dist/cjs",
    "build:esm": "tsc --module esnext --outDir dist/esm",
    "build:types": "tsc --declaration --emitDeclarationOnly --outDir dist/types",
    "generate": "openapi-generator-cli generate -i ../openapi-spec.yaml -g typescript-axios -o src/generated",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src --ext .ts",
    "format": "prettier --write \"src/**/*.ts\"",
    "docs": "typedoc src/index.ts"
  },
  "dependencies": {
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "@types/jest": "^29.5.0",
    "@types/node": "^20.0.0",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.0.0",
    "jest": "^29.5.0",
    "prettier": "^3.0.0",
    "ts-jest": "^29.1.0",
    "typedoc": "^0.25.0",
    "typescript": "^5.0.0",
    "@openapitools/openapi-generator-cli": "^2.7.0"
  },
  "keywords": [
    "ingitdb",
    "database",
    "git",
    "versioned",
    "collaboration",
    "typescript"
  ],
  "repository": {
    "type": "git",
    "url": "https://github.com/ingitdb/ingitdb-ts.git"
  },
  "license": "MIT"
}
```

## Testing Strategy

### Unit Tests

```typescript
// test/unit/query-builder.test.ts
import { QueryBuilder } from '../../src/query/query-builder';

describe('QueryBuilder', () => {
  let builder: QueryBuilder;

  beforeEach(() => {
    builder = new QueryBuilder();
  });

  test('should build simple filter', () => {
    const params = builder
      .whereEquals('status', 'active')
      .build();

    expect(params.filter).toBe('{"status":"active"}');
  });

  test('should build complex filter', () => {
    const params = builder
      .whereGreaterThan('age', 18)
      .whereContains('name', 'John')
      .build();

    const filter = JSON.parse(params.filter!);
    expect(filter.age).toEqual({ $gt: 18 });
    expect(filter.name).toEqual({ $contains: 'John' });
  });

  test('should build sort expression', () => {
    const params = builder
      .sortBy('name', 'asc')
      .sortBy('age', 'desc')
      .build();

    expect(params.sort).toBe('name:asc,age:desc');
  });
});
```

### Integration Tests

```typescript
// test/integration/database.test.ts
import { InGitDB } from '../../src';
import { setupTestServer, teardownTestServer } from '../helpers';

describe('Database Operations', () => {
  let client: InGitDB;

  beforeAll(async () => {
    await setupTestServer();
    client = new InGitDB({
      baseUrl: 'http://localhost:3000/api/v1',
      apiKey: 'test-key',
    });
  });

  afterAll(async () => {
    await teardownTestServer();
  });

  test('should create database', async () => {
    const db = await client.databases.create({
      name: 'test-db',
      description: 'Test database',
    });

    expect(db.name).toBe('test-db');
    expect(db.description).toBe('Test database');
  });

  test('should list databases', async () => {
    const result = await client.databases.list();

    expect(result.databases).toBeInstanceOf(Array);
    expect(result.pagination.total).toBeGreaterThan(0);
  });
});
```

## Performance Optimizations

### Connection Pooling

```typescript
// Axios automatically handles connection pooling
// Can be configured via httpAgent/httpsAgent
const agent = new https.Agent({
  keepAlive: true,
  maxSockets: 50,
});

const client = new InGitDB({
  baseUrl: 'https://api.ingitdb.io/v1',
  httpAgent: agent,
});
```

### Request Caching

```typescript
// Optional caching layer
import { CacheManager } from './cache';

class CachedHttpClient extends HttpClient {
  private cache: CacheManager;

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const cacheKey = `${url}:${JSON.stringify(config)}`;
    
    const cached = await this.cache.get(cacheKey);
    if (cached) return cached;

    const result = await super.get<T>(url, config);
    await this.cache.set(cacheKey, result, 60); // Cache for 60 seconds
    
    return result;
  }
}
```

## Browser Support

### Webpack Configuration

```javascript
// webpack.config.js
module.exports = {
  entry: './src/index.ts',
  output: {
    filename: 'ingitdb.min.js',
    library: 'InGitDB',
    libraryTarget: 'umd',
  },
  resolve: {
    extensions: ['.ts', '.js'],
    fallback: {
      // Polyfills for Node.js modules in browser
      buffer: require.resolve('buffer/'),
      stream: require.resolve('stream-browserify'),
    },
  },
};
```

## Deployment

### NPM Publishing

```bash
# Build the package
npm run build

# Run tests
npm test

# Publish to NPM
npm publish
```

### GitHub Packages

```yaml
# .github/workflows/publish.yml
name: Publish Package

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: 'https://registry.npmjs.org'
      
      - run: npm ci
      - run: npm test
      - run: npm run build
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## Documentation Generation

### TypeDoc Configuration

```json
// typedoc.json
{
  "entryPoints": ["src/index.ts"],
  "out": "docs",
  "exclude": ["**/*.test.ts", "**/generated/**"],
  "plugin": ["typedoc-plugin-markdown"],
  "readme": "README.md",
  "excludePrivate": true,
  "excludeProtected": true
}
```

## Future Enhancements

1. **WebSocket Support**: Real-time updates for records
2. **Stream Support**: Handle large result sets efficiently
3. **Offline Support**: Queue operations when offline
4. **GraphQL Client**: Alternative to REST API
5. **React Hooks**: Convenience hooks for React applications
6. **Vue Composables**: Convenience composables for Vue applications
7. **Batch Operations**: Optimize multiple operations
8. **Transaction Support**: Multi-operation transactions

## Conclusion

The ingitdb-ts client provides a robust, type-safe, and developer-friendly interface to the inGitDB server. With its fluent API, comprehensive error handling, and full TypeScript support, it enables developers to build applications that leverage git-based version control for their data with ease.
