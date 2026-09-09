# DFI — Agent Instructions

DFI (Draje Factory Information System) is a factory management system used by Draje.

This workspace contains the frontend and backend of DFI as two independent Git repositories.

## Mandatory Instruction Loading

At the start of every new chat/session, before planning, searching, inspecting code, or running
commands, read these instruction files completely, in this order. All paths are relative to the
workspace root:

1. `AGENTS.md`
2. `dfi-api/AGENTS.md`
3. `dfi-web/AGENTS.md`

This applies even when the initial request appears to concern only the frontend or backend. Treat
the child instructions as mandatory supplements to this workspace-level file.

Within the same chat/session, do not reread all three files for every new task. Reread the relevant
instruction files only when one has changed, the conversation context has been compacted or reset,
or the task enters a repository whose instructions have not yet been read in the current session.

---

## Workspace Structure

```text
DFI/
├── AGENTS.md
├── DFI.code-workspace
├── dfi-api/
└── dfi-web/
```

### `dfi-api/`

Backend API.

Main technologies:

* ASP.NET Core
* Entity Framework Core
* SQL Server
* REST API
* Rahkaran ERP integration

Git repository:

```text
hamedg68/dfi-api
```

### `dfi-web/`

Frontend application.

Main technologies:

* React
* Ant Design
* Axios
* REST API client

Git repository:

```text
hamedg68/dfi-web
```

---

# Architecture

The frontend in `dfi-web/` communicates with the backend in `dfi-api/` through REST APIs.

A feature may therefore involve changes in:

1. frontend components,
2. frontend service functions,
3. backend controllers,
4. backend models or DTOs,
5. Entity Framework queries,
6. SQL Server objects such as tables, views, functions, or stored procedures.

When investigating a feature or bug, do not assume the problem belongs only to the repository where it was first observed.

Inspect both repositories when necessary.

---

# General Rules

## Cross-project investigation

When a frontend component consumes an API:

* Find the frontend service function that performs the request.
* Determine the exact endpoint being called.
* Find the corresponding backend controller/action.
* Inspect the response model or DTO.
* Trace the backend query or stored procedure when relevant.

Do not guess the backend behavior based only on frontend code.

Likewise, do not assume a backend property is actually used by the frontend without checking `dfi-web/`.

---

## API Contract Changes

Whenever changing an API contract, inspect both projects.

Examples include:

* adding a response property,
* removing a property,
* renaming a property,
* changing its type,
* changing request parameters,
* changing route names,
* changing enum/status values,
* changing nullable behavior.

Before modifying an API contract, search `dfi-web/` for existing consumers.

After modifying it, verify that frontend usage still matches the backend response.

---

# Backend Guidelines

Backend source is located in:

```text
dfi-api/
```

When investigating backend behavior, trace the flow when applicable:

```text
HTTP Request
    ↓
Controller
    ↓
DbContext / Service / LINQ
    ↓
Entity Framework
    ↓
SQL Server / Stored Procedure / Rahkaran
```

Before changing backend behavior:

* inspect the relevant controller,
* inspect associated models/DTOs,
* identify the correct `DbContext`,
* inspect any stored procedure involved,
* check whether the endpoint is consumed by the frontend.

Do not assume all database data belongs to the DFI database.

Some functionality integrates with Rahkaran and may depend on Rahkaran schemas, tables, or stored procedures.

---

# Frontend Guidelines

Frontend source is located in:

```text
dfi-web/
```

When investigating frontend behavior, trace the flow when applicable:

```text
React Component
    ↓
Frontend service function
    ↓
Axios request
    ↓
DFI API endpoint
```

Before modifying frontend data handling:

* inspect the component,
* inspect the service function making the API request,
* determine the shape of the backend response,
* check transformations, grouping, filtering, and mapping performed after the request.

Do not invent frontend fields that do not exist in the backend response.

---

# Local Service Execution and Authenticated API Testing

A dedicated DFI test account is available through these environment variables:

```text
DFI_TEST_USERNAME
DFI_TEST_PASSWORD
```

When testing protected API endpoints, authenticate with:

```text
POST /api/Login/Login/authenticate
```

Store the returned JWT only in `/tmp` and send it through the `Authorization: Bearer` header.
Never print, log, commit, or expose the username, password, or JWT. Before claiming that
authentication is unavailable, check that both environment variables are configured without
displaying their values.

When a relevant test requires local services and they are not already running, the agent may start
them with these commands:

```bash
# From dfi-api
dotnet run --no-build --project Draje.csproj --launch-profile Draje

# From dfi-web
npm start
```

After backend source changes, run `dotnet build Draje.csproj --no-restore` successfully before using
`dotnet run --no-build`; otherwise the running API may use stale binaries. If frontend dependencies
are missing, run `npm ci` before `npm start`.

Backend ports are `5000`/`5001`; the frontend port is `3000`. First check whether a port is already
in use and reuse the running service when possible. A sandbox approval may still be required to bind
or access local ports: instructions in this file authorize attempting the in-scope command, but do
not grant or bypass the product's approval mechanism. When an approval prompt is shown, request the
narrow reusable command prefix and let the user decide whether to persist it.

For backend-only API investigations, do not start the frontend unless UI behavior also needs testing.

---

# Database Changes

## Read-only database inspection

For direct DFI SQL Server inspection, use the repository tool:

```text
dfi-api/Tools/DfiDbQuery/dfi-db
```

The tool supports two explicit database profiles:

```text
dfi       -> DFI_DB_CONNECTION_STRING
rahkaran  -> RAHKARAN_DB_CONNECTION_STRING
```

Never print, log, commit, or expose either variable's value. Prefer reusable `.sql` files or stdin;
do not place credentials in commands or files inside the workspace.

Before claiming that database access is unavailable, run:

```bash
dfi-api/Tools/DfiDbQuery/dfi-db --database dfi --check
dfi-api/Tools/DfiDbQuery/dfi-db --database rahkaran --check
```

Always specify `--database dfi` or `--database rahkaran` explicitly in agent-driven inspections.

Use this tool only for read-only inspection unless the user explicitly requests and authorizes a
different database operation. The configured SQL login should have `db_datareader` permissions only.
The tool accepts only `SELECT` and CTE queries and rejects common mutating statements locally.

Full setup and usage instructions are in:

```text
dfi-api/Tools/DfiDbQuery/README.md
```

Database behavior may be implemented using:

* Entity Framework,
* LINQ,
* SQL Server tables,
* SQL Server views,
* SQL Server stored procedures,
* Rahkaran database objects.

When modifying a stored procedure:

* preserve existing behavior unless the requested change explicitly requires otherwise,
* identify every result column consumed by the backend,
* check the backend result model,
* check frontend consumers when applicable.

Do not rename or remove SQL result columns without tracing their consumers.

---

# Git Repositories

`dfi-api/` and `dfi-web/` are independent Git repositories.

A change in one repository must not automatically be assumed to belong in the other repository.

Before suggesting Git commands, determine which repository the command should run in.

For example:

```bash
cd dfi-api
git status
```

and:

```bash
cd dfi-web
git status
```

are independent.

Do not commit, reset, rebase, checkout, restore, or otherwise modify Git history unless explicitly requested.

Never discard existing uncommitted user changes.

---

# Existing Code Changes

The working tree may contain uncommitted changes.

Treat existing modifications as intentional unless there is clear evidence otherwise.

Before proposing destructive changes:

* inspect the current implementation,
* distinguish existing user changes from the requested changes,
* avoid overwriting unrelated code.

Prefer minimal, targeted changes over broad rewrites unless a refactor is explicitly requested.

---

# Coding Style

Follow the style already used in the surrounding code.

Before introducing a new:

* abstraction,
* helper,
* dependency,
* library,
* architecture pattern,
* state-management solution,

first inspect whether the project already has an established approach for the same problem.

Prefer consistency with the existing codebase.

Do not refactor unrelated code while solving a specific problem.

---

# Dependencies

Do not add or upgrade packages automatically.

Before recommending a new dependency:

1. inspect the existing dependencies,
2. check whether the project already has a suitable solution,
3. explain why an additional dependency is necessary.

Backend and frontend dependencies must be treated independently.

---

# Debugging

When debugging, find the root cause before changing code.

Prefer tracing the actual data flow over making speculative fixes.

For frontend/backend issues, inspect:

```text
UI
→ component
→ service
→ HTTP request
→ controller
→ query/SP
→ database
```

and then trace the returned data in the opposite direction.

If logs, API responses, SQL output, or error messages are available, use them as evidence rather than guessing.

---

# Terminology

DFI means:

```text
Draje Factory Information System
```

Draje is the factory/company for which DFI is developed.

Use the following workspace terminology consistently:

```text
DFI
├── dfi-api   → backend
└── dfi-web   → frontend
```

---

# Agent Behavior

When answering development questions in this workspace:

1. Inspect relevant files before proposing changes.
2. Search both repositories when the issue crosses the API boundary.
3. Do not ask for code that already exists in the workspace and can be inspected.
4. Do not assume APIs, DTOs, database columns, or component behavior without checking them.
5. Prefer concrete file paths and exact locations when explaining changes.
6. When suggesting code changes, clearly state which file should be modified.
7. Preserve existing application behavior unless a behavior change is explicitly requested.
8. Do not modify unrelated code.
9. Point out uncertainty when the available code does not establish a fact.
10. When multiple implementations are possible, prefer the one most consistent with the existing project.

---

# Change Explanations

When explaining a proposed modification, prefer this format:

```text
File:
dfi-web/src/...

Change:
Explain exactly what needs to change.

Reason:
Explain why this change is needed.
```

For changes spanning both projects:

```text
Backend:
dfi-api/...

Frontend:
dfi-web/...
```

This makes cross-project changes easier to review.

---

# Important

The Persian setup and migration guide for preparing DFI on a new system is located at:

```text
SETUP.fa.md
```

Consult it when diagnosing missing SDKs, environment variables, database connectivity, PCH storage,
or frontend/backend startup on a new machine.

* `dfi-api/` and `dfi-web/` are separate Git repositories.
* The workspace root itself is not application source code.
* Frontend and backend must be inspected together for API-related changes.
* Do not assume frontend fields exist in backend DTOs.
* Do not assume backend response properties are used by the frontend.
* Trace database-backed behavior to its actual query or stored procedure when necessary.
* Preserve unrelated user changes.
* Prefer evidence from the codebase over assumptions.
