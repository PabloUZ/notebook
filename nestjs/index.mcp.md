# NestJS Documentation Index for MCP

This file provides instructions for MCP (Model Context Protocol) agents on how to access and use the NestJS documentation.

## Base URL Structure

All documentation files are accessible using the following pattern:
```
<base_url>/nestjs/<path_to_file>
```

Where `<base_url>` is the static URL prefix and `<path_to_file>` is the relative path from the `nestjs/` directory.

---

## Available Documentation

### Setup (Initial Configuration)

**When user wants to:** Create a new NestJS project
**Use file:** `setup/create-project.md`
**Full path:** `<base_url>/nestjs/setup/create-project.md`

**When user wants to:** Dockerize their application
**Use file:** `setup/dockerization.md`
**Full path:** `<base_url>/nestjs/setup/dockerization.md`

**When user wants to:** Configure environment variables
**Use file:** `setup/environment-variables.md`
**Full path:** `<base_url>/nestjs/setup/environment-variables.md`

---

### Database (Data Persistence)

**When user wants to:** Dockerize a database (MySQL, PostgreSQL, or MongoDB)
**Use file:** `database/docker-setup.md`
**Full path:** `<base_url>/nestjs/database/docker-setup.md`

#### TypeORM (For SQL Databases)

**When user wants to:** Configure TypeORM
**Use file:** `database/typeorm/configuration.md`
**Full path:** `<base_url>/nestjs/database/typeorm/configuration.md`

**When user wants to:** Use TypeORM (entities, decorators, queries)
**Use file:** `database/typeorm/usage.md`
**Full path:** `<base_url>/nestjs/database/typeorm/usage.md`

#### Mongoose (For MongoDB)

**When user wants to:** Configure Mongoose
**Use file:** `database/mongoose/configuration.md`
**Full path:** `<base_url>/nestjs/database/mongoose/configuration.md`

**When user wants to:** Use Mongoose (schemas, decorators, queries)
**Use file:** `database/mongoose/usage.md`
**Full path:** `<base_url>/nestjs/database/mongoose/usage.md`

---

### Validation (Data Validation)

**When user wants to:** Validate request data with DTOs
**Use file:** `validation/dto.md`
**Full path:** `<base_url>/nestjs/validation/dto.md`

---

### Security (Application Security)

**When user wants to:** Hash passwords with bcrypt
**Use file:** `security/password-hashing.md`
**Full path:** `<base_url>/nestjs/security/password-hashing.md`

---

### Documentation (API Documentation)

**When user wants to:** Set up Swagger documentation
**Use file:** `documentation/swagger.md`
**Full path:** `<base_url>/nestjs/documentation/swagger.md`

---

## How to Use This Index

1. **Identify the user's need** based on their question or request
2. **Find the matching documentation** in the sections above
3. **Complete the URL** by replacing `<base_url>` with the actual base URL
4. **Fetch the content** using the complete URL path

### Example Usage

**User request:** "How do I create a new NestJS project?"
**Action:** Fetch `<base_url>/nestjs/setup/create-project.md`

**User request:** "I need to configure MongoDB in my NestJS app"
**Action:** Fetch `<base_url>/nestjs/database/mongoose/configuration.md`

**User request:** "How do I validate request data?"
**Action:** Fetch `<base_url>/nestjs/validation/dto.md`

---

## Complete File List

```
nestjs/
├── index.md                              # Main documentation index
├── setup/
│   ├── create-project.md                 # Create new project
│   ├── dockerization.md                  # Docker setup
│   └── environment-variables.md          # Environment configuration
├── database/
│   ├── docker-setup.md                   # Database dockerization
│   ├── typeorm/
│   │   ├── configuration.md              # TypeORM setup
│   │   └── usage.md                      # TypeORM usage
│   └── mongoose/
│       ├── configuration.md              # Mongoose setup
│       └── usage.md                      # Mongoose usage
├── validation/
│   └── dto.md                            # DTO validation
├── security/
│   └── password-hashing.md               # Password hashing
└── documentation/
    └── swagger.md                        # Swagger setup
```

---

## Keywords for Quick Reference

- **Create project** → `setup/create-project.md`
- **Docker** → `setup/dockerization.md`
- **Environment variables**, **env**, **.env** → `setup/environment-variables.md`
- **Database docker** → `database/docker-setup.md`
- **TypeORM config** → `database/typeorm/configuration.md`
- **TypeORM entity**, **TypeORM usage** → `database/typeorm/usage.md`
- **Mongoose config** → `database/mongoose/configuration.md`
- **Mongoose schema**, **Mongoose usage** → `database/mongoose/usage.md`
- **DTO**, **validation**, **class-validator** → `validation/dto.md`
- **Password**, **bcrypt**, **hash** → `security/password-hashing.md`
- **Swagger**, **API docs** → `documentation/swagger.md`
