# 🔌 inGitDB Server API Reference

> Executive overview and endpoint index for [`openapi-spec.yaml`](./openapi-spec.yaml)
>
> Part of the [main inGitDB repository](../../README.md) — the central hub for project planning and architecture documentation.

---

## Executive Overview

The inGitDB Server API is a RESTful interface (OpenAPI 3.0.3, v1.0.0) that exposes a versioned database backed by Git.
Data lives in text files (JSON, YAML, or columnar format) and every mutation is tracked as a Git commit, giving teams full history, branching, and conflict-aware merging out of the box.

### Key Concepts

| Concept      | Description                                                                    |
| ------------ | ------------------------------------------------------------------------------ |
| **Database** | A named collection of tables stored as a Git repository or subdirectory        |
| **Table**    | A collection of records validated against a JSON Schema                        |
| **Record**   | An individual data entry with auto-generated `_id`, `_version`, and timestamps |
| **Branch**   | A Git branch for isolating data changes                                        |
| **Commit**   | A Git commit representing a point-in-time snapshot of database state           |

### Design Highlights

- **Authentication** — JWT bearer tokens on all endpoints except health check
- **Pagination** — Consistent page/limit parameters across every list endpoint (default 20, max 100)
- **Filtering & sorting** — MongoDB-style filter syntax and `field:direction` sort on record queries
- **Field projection** — Return only the fields you need via the `fields` query parameter
- **Storage formats** — JSON (default), YAML, or columnar per database
- **Merge strategies** — merge, squash, or rebase with automatic conflict detection

### Servers

| Environment | Base URL                       |
| ----------- | ------------------------------ |
| Local dev   | `http://localhost:3000/api/v1` |
| Production  | `https://api.ingitdb.io/v1`    |

### Error Model

All errors return a consistent structure:

```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Human-readable description",
    "details": {}
  }
}
```

Standard HTTP status codes: `400` Bad Request, `401` Unauthorized, `404` Not Found, `409` Conflict.

---

## Endpoint Index

### Health (1 endpoint)

| Method | Path      | Operation   | Summary                        |
| ------ | --------- | ----------- | ------------------------------ |
| `GET`  | `/health` | `getHealth` | Check server health and uptime |

### Databases (4 endpoints)

| Method   | Path                  | Operation        | Summary                                                               |
| -------- | --------------------- | ---------------- | --------------------------------------------------------------------- |
| `GET`    | `/databases`          | `listDatabases`  | List all databases with pagination                                    |
| `POST`   | `/databases`          | `createDatabase` | Create a new database with optional storage format and initial branch |
| `GET`    | `/databases/{dbName}` | `getDatabase`    | Get database details (branch, table count, metadata)                  |
| `DELETE` | `/databases/{dbName}` | `deleteDatabase` | Permanently delete a database and all its data                        |

### Tables (5 endpoints)

| Method   | Path                                     | Operation     | Summary                                              |
| -------- | ---------------------------------------- | ------------- | ---------------------------------------------------- |
| `GET`    | `/databases/{dbName}/tables`             | `listTables`  | List all tables in a database                        |
| `POST`   | `/databases/{dbName}/tables`             | `createTable` | Create a table with JSON Schema and optional indexes |
| `GET`    | `/databases/{dbName}/tables/{tableName}` | `getTable`    | Get table schema, metadata, and record count         |
| `PUT`    | `/databases/{dbName}/tables/{tableName}` | `updateTable` | Update table schema, description, or indexes         |
| `DELETE` | `/databases/{dbName}/tables/{tableName}` | `deleteTable` | Delete a table and all its records                   |

### Records (6 endpoints)

| Method   | Path                                                        | Operation       | Summary                                                                 |
| -------- | ----------------------------------------------------------- | --------------- | ----------------------------------------------------------------------- |
| `GET`    | `/databases/{dbName}/tables/{tableName}/records`            | `queryRecords`  | Query records with filter, sort, field projection, and pagination       |
| `POST`   | `/databases/{dbName}/tables/{tableName}/records`            | `insertRecords` | Insert one or more records (batch); returns inserted records and commit |
| `GET`    | `/databases/{dbName}/tables/{tableName}/records/{recordId}` | `getRecord`     | Get a single record by ID                                               |
| `PUT`    | `/databases/{dbName}/tables/{tableName}/records/{recordId}` | `updateRecord`  | Full replacement of a record                                            |
| `PATCH`  | `/databases/{dbName}/tables/{tableName}/records/{recordId}` | `patchRecord`   | Partial update of specific fields                                       |
| `DELETE` | `/databases/{dbName}/tables/{tableName}/records/{recordId}` | `deleteRecord`  | Delete a record                                                         |

### Version Control (8 endpoints)

| Method   | Path                                        | Operation        | Summary                                                                |
| -------- | ------------------------------------------- | ---------------- | ---------------------------------------------------------------------- |
| `GET`    | `/databases/{dbName}/branches`              | `listBranches`   | List all branches and identify the current one                         |
| `POST`   | `/databases/{dbName}/branches`              | `createBranch`   | Create a branch from HEAD or a specific commit, with optional checkout |
| `GET`    | `/databases/{dbName}/branches/{branchName}` | `getBranch`      | Get branch details including latest commit                             |
| `DELETE` | `/databases/{dbName}/branches/{branchName}` | `deleteBranch`   | Delete a branch (cannot delete current or main)                        |
| `GET`    | `/databases/{dbName}/commits`               | `listCommits`    | List commit history, optionally filtered by branch                     |
| `GET`    | `/databases/{dbName}/commits/{sha}`         | `getCommit`      | Get commit details including author, parents, and change stats         |
| `POST`   | `/databases/{dbName}/merge`                 | `mergeBranches`  | Merge branches with strategy selection and conflict detection          |
| `POST`   | `/databases/{dbName}/checkout`              | `checkoutBranch` | Switch the active branch                                               |

---

## Summary

| Category        | Endpoints | Auth Required |
| --------------- | --------- | ------------- |
| Health          | 1         | No            |
| Databases       | 4         | Yes           |
| Tables          | 5         | Yes           |
| Records         | 6         | Yes           |
| Version Control | 8         | Yes           |
| **Total**       | **24**    |               |

---

## Related

- [← Back to Documentation Index](../README.md)
- [← Back to Project Root](../../README.md)
- [Development Plan](../dev-plan/)
- [TypeScript Client Architecture](../ingitdb-ts-architecture.md)
