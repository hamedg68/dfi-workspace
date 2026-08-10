# DFI

DFI is the factory management system used by Draje.

## Projects

### dfi-api/

Backend API.

- ASP.NET Core
- SQL Server
- Entity Framework Core
- Rahkaran integration

### dfi-web/

Frontend application.

- React
- Ant Design
- REST API client

## Architecture

The frontend in `dfi-web/` communicates with the backend in `dfi-api/`.

When working on features, always check whether changes are required
in both projects.

## Terminology

DFI = Draje Factory Information System

## Important

- `dfi-api/` and `dfi-web/` are separate Git repositories.
- Do not assume a frontend field exists in the backend DTO.
- When changing API contracts, inspect both projects.
