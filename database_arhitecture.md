# Database architecture for the Finance Management web application

## 1. Purpose

This document defines the database architecture for the personal finance management application. The database is responsible for storing user accounts, financial transactions, categories, invoice records, and supporting data needed to calculate summaries and maintain data integrity.

The MVP uses a relational database for reliability, structured querying, and future growth. PostgreSQL is recommended for production, while SQLite may be used for local development and testing.

## 2. Database choice

### Production database

- PostgreSQL

### Development database

- SQLite

### Why PostgreSQL

PostgreSQL is appropriate because it offers:

- strong relational data integrity;
- support for transactions and constraints;
- mature SQL tooling;
- good performance for moderate application workloads;
- straightforward migration management;
- compatibility with Rust-based application stacks using SQLx.

## 3. Architectural approach

The database follows a relational model with normalized tables and clear ownership relationships. The system is designed around a single-user account model, where each record is tied to a user.

Key design principles:

- one user can have many transactions;
- one user can have many categories;
- one transaction can optionally reference one invoice;
- each table should enforce data integrity through constraints;
- foreign keys should be used to preserve data relationships.

## 4. Core entities

### 4.1 users

Stores user account information.

Columns:

- id: UUID or BIGSERIAL
- email: VARCHAR(255), unique
- password_hash: TEXT
- created_at: TIMESTAMP WITH TIME ZONE
- updated_at: TIMESTAMP WITH TIME ZONE

Constraints:

- email must be unique;
- email must be non-null;
- password_hash must be non-null.

### 4.2 categories

Stores categories that users can assign to transactions.

Columns:

- id: UUID or BIGSERIAL
- user_id: UUID or BIGINT, foreign key to users.id
- name: VARCHAR(100)
- description: TEXT, nullable
- created_at: TIMESTAMP WITH TIME ZONE

Constraints:

- user_id must exist in users;
- category name must be non-null;
- each user should not have duplicate category names unless explicitly allowed.

### 4.3 transactions

Stores financial transactions such as income and expenses.

Columns:

- id: UUID or BIGSERIAL
- user_id: UUID or BIGINT, foreign key to users.id
- category_id: UUID or BIGINT, foreign key to categories.id, nullable
- type: VARCHAR(20) or ENUM
- amount: DECIMAL(12,2)
- date: DATE
- description: TEXT, nullable
- invoice_reference: VARCHAR(255), nullable
- created_at: TIMESTAMP WITH TIME ZONE
- updated_at: TIMESTAMP WITH TIME ZONE

Constraints:

- user_id must exist in users;
- category_id may be null but, if provided, must exist in categories;
- type must be one of: income, expense;
- amount must be greater than zero;
- date must be non-null;

### 4.4 invoices

Stores invoice details associated with transactions or invoice-related entries.

Columns:

- id: UUID or BIGSERIAL
- user_id: UUID or BIGINT, foreign key to users.id
- transaction_id: UUID or BIGINT, foreign key to transactions.id, nullable
- invoice_number: VARCHAR(100), nullable
- amount: DECIMAL(12,2)
- issue_date: DATE
- due_date: DATE, nullable
- notes: TEXT, nullable
- created_at: TIMESTAMP WITH TIME ZONE

Constraints:

- user_id must exist in users;
- transaction_id may be null but must reference a valid transaction if provided;
- amount must be greater than zero.

## 5. Relationships

### User to categories

- One user can have many categories.
- Each category belongs to one user.

### User to transactions

- One user can have many transactions.
- Each transaction belongs to one user.

### Category to transactions

- One category can be associated with many transactions.
- Each transaction can have one category.

### User to invoices

- One user can have many invoices.
- Each invoice belongs to one user.

### Transaction to invoice

- One transaction may have zero or one invoice record.
- One invoice may belong to one transaction.

## 6. Data integrity rules

The database should enforce the following rules:

- user email must be unique;
- category_name must not be blank;
- transaction amount must be positive;
- invoice amount must be positive;
- transaction type must be limited to valid values;
- foreign keys must prevent orphaned records;
- date fields must not be null when required;
- soft-delete behavior may be considered later if needed.

## 7. Example SQL schema

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE categories (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE transactions (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    category_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    type VARCHAR(20) NOT NULL CHECK (type IN ('income', 'expense')),
    amount DECIMAL(12,2) NOT NULL CHECK (amount > 0),
    date DATE NOT NULL,
    description TEXT,
    invoice_reference VARCHAR(255),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE invoices (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    transaction_id UUID REFERENCES transactions(id) ON DELETE SET NULL,
    invoice_number VARCHAR(100),
    amount DECIMAL(12,2) NOT NULL CHECK (amount > 0),
    issue_date DATE NOT NULL,
    due_date DATE,
    notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

## 8. Query patterns

The database should support efficient access for common operations:

- list all transactions for a user ordered by date desc;
- filter transactions by type, category, or date range;
- calculate total income and total expenses for a period;
- retrieve a user’s category list;
- get invoice records associated with transactions;
- fetch a single transaction or invoice by id.

## 9. Indexing strategy

Indexes should be added for high-use access patterns, for example:

- users.email unique index;
- transactions.user_id index;
- transactions.date index;
- transactions.user_id + date index;
- categories.user_id index;
- invoices.user_id index;
- invoices.transaction_id index.

## 10. Migration strategy

The application should use migrations to evolve the schema safely.

Recommended process:

- define schema changes in migration files;
- run migration scripts during deployment;
- validate existing data before destructive changes;
- keep migration history versioned in source control.

## 11. Backup and recovery

The production database should include:

- regular automated backups;
- point-in-time recovery where supported;
- retention policies for backups;
- restoration testing to verify recoverability.

## 12. Performance considerations

The design should support simple performance goals for the MVP.

Recommended optimization measures:

- proper indexes on common search filters;
- limiting large result sets when listing transactions;
- using efficient aggregate queries for summary calculations;
- avoiding unnecessary joins in dashboard queries;
- monitoring slow query performance.

## 13. Security requirements for the database

The database layer must support application security requirements:

- use authentication for database access;
- restrict privileges to the application service account;
- store secrets outside source control;
- use SSL/TLS for production connections;
- avoid exposing raw database credentials to the frontend;
- validate all data before persistence.

## 14. Future extension considerations

The schema is designed for future enhancement without major redesign.

Possible additions:

- recurring transactions table;
- budgets table;
- tags or labels table;
- recurring invoice table;
- reporting snapshots or analytics tables;
- audit trail tables for data changes.

## 15. Summary

The database architecture for the Finance Management web application is a relational model built around a user-owned transaction ledger. PostgreSQL is the recommended production database, with SQLite suitable for local development. The schema is designed to maintain integrity, support common finance queries, and provide a clean path for future expansion without requiring a major redesign.
