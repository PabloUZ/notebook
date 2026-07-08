# NestJS Complete Guide

> [!IMPORTANT]
> If something in the code is surrounded by "< >" symbols, you must replace it with its proper value.

This is a comprehensive, step-by-step guide to building production-ready NestJS applications. Follow the sections in order for the best learning experience, or jump to specific topics as needed.

---

## Getting Started

These are the essential steps to set up your NestJS project:

### 1. Setup
Initial project configuration and environment setup.

- **[Create Project](./setup/create-project.md)**
  Create a new NestJS project

- **[Configure Scripts](./setup/configure-scripts.md)**
  Configure package.json scripts for development and production

- **[Configure EditorConfig, Prettier and ESLint](./setup/configure-rules-editor.md)**
  Set up editor rules, code formatting and linting for code consistency

- **[Dockerization](./setup/dockerization.md)**
  Set up Docker for development and production environments

- **[Environment Variables](./setup/environment-variables.md)**
  Configure and validate environment variables with ConfigModule---

## Database

Choose your database and learn how to integrate it with NestJS.

### Database Setup
- **[Docker Database Setup](./database/docker-setup.md)**
  Dockerize MySQL, PostgreSQL, or MongoDB

### TypeORM (For SQL Databases)
Use TypeORM for MySQL, PostgreSQL, SQLite, and other SQL databases.

- **[Configure TypeORM](./database/typeorm/configuration.md)**
  Install and configure TypeORM in your NestJS application

- **[Using TypeORM](./database/typeorm/usage.md)**
  Create entities, use decorators, and interact with your database

- **[Migrations](./database/typeorm/migrations.md)**
  Manage database schema changes with TypeORM migrations

### Mongoose (For MongoDB)
Use Mongoose for MongoDB document databases.

- **[Configure Mongoose](./database/mongoose/configuration.md)**
  Install and configure Mongoose in your NestJS application

- **[Using Mongoose](./database/mongoose/usage.md)**
  Create schemas, use decorators, and interact with MongoDB

---

## Validation

Ensure data integrity with DTOs and validation.

- **[Data Transfer Objects (DTOs)](./validation/dto.md)**
  Validate and transform request data using class-validator

---

## Security

Protect your application and user data.

- **[Password Hashing](./security/password-hashing.md)**
  Hash passwords with bcrypt for secure authentication

---

## Documentation

Generate interactive API documentation.

- **[Swagger](./documentation/swagger.md)**
  Set up Swagger for automatic API documentation

---

## How to Use This Guide

### For Beginners
Follow the guide from top to bottom:
1. **Setup** → Create and dockerize your project
2. **Database** → Choose and configure your database
3. **Validation** → Add data validation
4. **Security** → Implement security features
5. **Documentation** → Document your API

### For Experienced Developers
Jump directly to the topics you need. Each section is self-contained with all necessary information.

---

## Best Practices

Throughout this guide, you'll learn:
- How to structure your NestJS projects
- Environment configuration best practices
- Database connection and ORM usage
- Data validation techniques
- Security implementation
- API documentation standards