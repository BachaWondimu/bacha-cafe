---

# Environment Configuration Strategy

The Bacha Café Platform uses environment variables to manage configuration across local development and cloud deployments. Sensitive values such as database credentials, JWT secrets, and API keys are never stored directly in the codebase.

The configuration strategy follows a layered approach that keeps the application portable across environments.

---

## Local Development

During local development, configuration values are stored in a root-level `.env` file.

Example:

```text
bacha-cafe/
  .env
  .env.example
  services/
  frontend/
  docker/
```

Example `.env` file:

```bash
APP_ENV=local

DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=password

JWT_SECRET=local-dev-secret
JWT_EXPIRATION=3600000
```

This file is **not committed to Git** and is excluded using `.gitignore`.

---

## Environment Variable Reference File

An `.env.example` file is included in the repository to document required variables without exposing secrets.

Example:

```bash
APP_ENV=

DB_HOST=
DB_PORT=
DB_USER=
DB_PASSWORD=

JWT_SECRET=
JWT_EXPIRATION=
```

Developers copy this file and create their own `.env` during local setup.

---

## Spring Boot Configuration

Spring Boot services reference environment variables inside `application.yml`.

Example:

```yaml
spring:
  datasource:
    url: jdbc:mysql://${DB_HOST}:${DB_PORT}/auth_db
    username: ${DB_USER}
    password: ${DB_PASSWORD}

security:
  jwt:
    secret: ${JWT_SECRET}
    expiration: ${JWT_EXPIRATION}
```

Spring automatically resolves these values from the environment at runtime.

---

## Docker Local Environment

Docker Compose loads the root `.env` file to inject variables into containers.

Example:

```yaml
services:
  auth-service:
    build: ../services/auth-service
    env_file:
      - ../.env
```

This ensures consistent configuration across services during local development.

---

## Render Deployment

For cloud deployment on Render, environment variables are configured directly in the Render dashboard.

Example:

Render → Service → Environment Variables

```
DB_HOST
DB_USER
DB_PASSWORD
JWT_SECRET
```

Render securely injects these variables into the container runtime.

The `.env` file is **not uploaded to Render**.

---

## AWS Deployment (Future)

When deploying to AWS, secrets will be managed using:

• AWS Secrets Manager
• AWS Systems Manager Parameter Store

Example parameter paths:

```
/bacha-cafe/db/password
/bacha-cafe/jwt/secret
```

This approach allows secure secret management and centralized configuration across environments.

---

## Configuration Flow

The overall configuration flow looks like this:

```
Local Development
      ↓
.env file
      ↓
Docker Compose
      ↓
Spring Boot services
      ↓
Cloud Deployment
      ↓
Render environment variables
      ↓
AWS Secrets Manager (future)
```

---

## Security Best Practices

Sensitive configuration values such as database credentials, JWT secrets, and API keys must never be committed to source control.

The application always references environment variables rather than hardcoding secrets.

Example:

```
${JWT_SECRET}
${DB_PASSWORD}
```

This ensures the system remains secure, portable, and compatible with multiple deployment environments.

---