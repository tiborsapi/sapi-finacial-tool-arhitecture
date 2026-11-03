# Software architecture for the Finance Management web application

## 1. Overview

This software is a personal finance management web application designed for a single user. The system allows the user to record income, expenses, and invoice-related financial records, review transaction history, and view a summary of their financial activity.

The architecture follows a modular web application model with a React frontend, a Rust backend, and a PostgreSQL relational database. The design prioritizes security, maintainability, testability, and extensibility while keeping the MVP focused and lightweight.

## 2. Architectural goals

The main architectural goals are:

- build a secure and user-friendly finance dashboard;
- support a simple single-user workflow;
- keep the application maintainable and easy to extend;
- separate concerns between UI, business logic, and data access;
- provide reliable persistence and validation for financial records;
- prepare the system for future enhancements such as reporting, recurring transactions, and exports.

## 3. Non-functional constraints

- The MVP will be a web application, not a mobile app.
- The system must support a single authenticated user.
- Financial and personal data must be protected through secure design.
- The application must support reliable transaction creation and editing.
- The architecture should be easy to test and deploy.
- The system must be compatible with a standard cloud deployment model.

## 4. High-level architecture

The application follows a three-tier architecture:

1. Presentation layer
   - React single-page application
   - Browser-based UI for login, dashboard, transaction forms, and summaries

2. Application layer
   - Rust backend using Axum
   - REST API endpoints for authentication, transactions, categories, and summaries
   - Business logic for validation, totals, and rules

3. Data layer
   - PostgreSQL for persistent storage
   - SQLx for data access
   - schema and migrations for database changes

### Basic component view

- Frontend UI
  - Dashboard page
  - Transaction list page
  - Add/edit transaction form
  - Invoice entry form
  - Summary and filter view

- Backend API
  - Auth service
  - Transaction service
  - Category service
  - Invoice service
  - Summary service
  - Validation layer
  - Repository/data access layer

- Database
  - users
  - transactions
  - categories
  - invoices
  - maybe recurring entries in future versions

## 5. Architecture style

This project uses a layered architecture with a service-oriented backend.

### Frontend architecture

The frontend is organized into the following concerns:

- pages and routes
- components and reusable UI elements
- hooks for API calls and state management
- validation and input rules
- API client layer
- utilities and formatting helpers

The frontend should use a clean React structure, with state normalized and server data handled in a predictable way.

### Backend architecture

The backend should be organized into modules by domain and responsibility.

Example backend structure:

- src/
  - main.rs
  - config/
  - api/
    - routes/
    - handlers/
    - dto/
  - domain/
    - models/
    - services/
    - validators/
  - infrastructure/
    - db/
    - repositories/
    - migrations/
  - security/
    - auth/
    - session/
    - middleware/
  - utils/

This keeps business logic separate from transport and persistence concerns.

## 6. Major components

### 6.1 Authentication and session service

This component manages:

- user registration;
- login and logout;
- password hashing;
- session creation or JWT handling;
- access checks for authenticated routes.

It must ensure that each user can only access their own data.

### 6.2 Transaction service

Responsible for:

- creating transactions;
- updating transactions;
- deleting transactions;
- retrieving transactions by date or category;
- validating transaction fields before persistence.

### 6.3 Category service

Responsible for:

- managing categories such as food, rent, salary, transport, or other custom entries;
- assigning categories to transactions;
- retrieving category lists for UI selection.

### 6.4 Invoice service

Responsible for:

- storing invoice metadata;
- associating invoice entries with transactions;
- retrieving invoice records for review and reporting.

### 6.5 Summary and reporting service

Responsible for:

- total income calculation;
- total expense calculation;
- net balance calculation;
- summary for selected periods;
- future reporting features such as filters and trend overview.

### 6.6 Repository and data access layer

This layer abstracts database access and handles:

- SQL queries;
- transaction persistence;
- result mapping to domain models;
- data filtering and sorting logic.

## 7. Data model

The database should include the following main entities:

### User

- id
- email
- password_hash
- created_at
- updated_at

### Category

- id
- user_id
- name
- description
- created_at

### Transaction

- id
- user_id
- category_id
- type
- amount
- date
- description
- invoice_reference
- created_at
- updated_at

### Invoice

- id
- user_id
- transaction_id
- invoice_number
- amount
- issue_date
- due_date
- notes
- created_at

### Design notes

- Each transaction belongs to one user.
- Each category belongs to one user.
- Invoices may be associated with a transaction when relevant.
- Amounts should be stored as precise decimal values, not floating-point text values.

## 8. Communication flow

### User registration flow

1. User submits registration form.
2. Frontend sends user data to the backend.
3. Backend validates data.
4. Backend hashes password.
5. Backend stores user in the database.
6. Backend returns success or validation error.

### Add transaction flow

1. User enters transaction information in the frontend.
2. Frontend validates form input.
3. Frontend sends data to the API.
4. Backend validates data and user permissions.
5. Backend stores the transaction in PostgreSQL.
6. Backend returns the created transaction.
7. Frontend updates the UI and recalculates summary values.

### Summary generation flow

1. User requests financial summary for a date range.
2. Backend queries transactions for the authenticated user.
3. Business logic calculates total income, total expenses, and net balance.
4. Backend returns JSON summary to frontend.
5. Frontend renders the financial overview.

## 9. API design

The backend should expose a REST API with JSON payloads.

### Example endpoints

- POST /api/auth/register
- POST /api/auth/login
- POST /api/auth/logout
- GET /api/transactions
- POST /api/transactions
- GET /api/transactions/:id
- PUT /api/transactions/:id
- DELETE /api/transactions/:id
- GET /api/categories
- POST /api/categories
- GET /api/summary
- GET /api/invoices
- POST /api/invoices

### API conventions

- Use HTTP status codes consistently.
- Validate request payloads before processing.
- Return structured responses for both success and failure.
- Protect all user-owned data routes with authentication middleware.

## 10. Security architecture

The application should follow secure design principles.

### Authentication and authorization

- Use secure password hashing.
- Use session tokens or JWTs if applicable.
- Restrict access to user-specific records.
- Ensure all protected routes check the authenticated identity.

### Transport security

- Use HTTPS in production.
- Enforce secure cookies if sessions are used.
- Add CSRF protection for cookie-based auth.

### Data security

- Store secrets in environment variables.
- Restrict database access to application services only.
- Do not log sensitive personal or financial payloads.
- Validate all input at the boundary.

### Web security headers

The backend should include standard HTTP security measures such as:

- HSTS
- X-Frame-Options
- X-Content-Type-Options
- Referrer Policy
- Content-Security-Policy

## 11. Deployment architecture

The system should be deployed as a cloud-based web application.

### Recommended deployment model

- Frontend: Vercel or Netlify
- Backend: Render, Railway, Azure App Service, or a container-based platform
- Database: managed PostgreSQL service
- CI/CD: GitHub Actions
- Monitoring: Sentry, logs, and uptime checks

This deployment model allows for simple staging and production environments with minimal operational complexity.

## 12. Observability and operations

The system must provide enough visibility for debugging and operational maintenance.

### Required operational features

- application logs with request IDs;
- error tracking for backend and frontend issues;
- database connection and query monitoring;
- performance tracking for critical endpoints;
- automated alerts for failed deployments or elevated error rates.

### Log examples

- authentication attempts;
- API failures;
- transaction validation errors;
- database errors;
- unusually slow request times.

## 13. Testing strategy

The architecture must support testability.

### Unit tests

- validation routines;
- summary calculation logic;
- amount and date handling;
- business rules.

### Integration tests

- API behavior;
- database operations;
- authorization rules;
- transaction workflows.

### End-to-end tests

- login flow;
- create/edit/delete transaction;
- summary view;
- validation errors and user feedback.

Use automated testing in CI before deployment.

## 14. Scalability and extensibility

The system is designed to support future enhancements without major structural changes.

Possible future features:

- recurring transactions;
- CSV/PDF export;
- rich analytics;
- multi-category budgets;
- multi-user or family support;
- localization and reporting enhancements.

The architecture can absorb these features by extending the existing service and repository layers.

## 15. Risks and assumptions

### Risks

- inadequate validation may lead to data integrity issues;
- poor session security may expose confidential financial data;
- missing monitoring may delay detection of production issues;
- weak architecture boundaries may make future features harder to add.

### Assumptions

- the application is used by one authenticated user per account;
- the MVP does not integrate with banking APIs;
- data is stored in a secure application environment;
- browser-based access is the primary delivery mode.

## 16. Summary

The proposed software architecture for the Finance Management web application is a modular, secure, and maintainable web system based on a React frontend and a Rust Axum backend with PostgreSQL persistence. It separates presentation, business logic, security, and data access into well-defined layers, enabling reliable transaction management and future extension with minimal structural risk.

This architecture aligns with the project goals, supports secure personal finance tracking, and provides a strong foundation for the MVP and future iterations.
