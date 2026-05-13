# Bootstrap

## Bootstrap Standards

- Every new project must bootstrap foundational infrastructure before feature development
- Bootstrap order matters — dependencies must be initialized before consumers
- Configuration must be loaded before any service starts

---

## Instructions

### Step 1 - Identification

> Understand what the service needs before bootstrapping

- Identify the service type (API server, worker, cron job, event consumer)
- Identify required infrastructure (database, cache, message queue, file storage)
- Identify required cross-cutting concerns (logging, auth, rate limiting, CORS)
- Identify the deployment target (Docker, serverless, VM)

---

### Step 2 - Bootstrap Checklist

> Evaluate each item and include only what is needed

#### Logging

- Structured logging (JSON format for production)
- Log levels: error, warn, info, debug
- Request/response logging with correlation IDs
- Never log secrets, tokens, or PII

#### Routing

- Route registration pattern (centralized file vs per-module)
- Route versioning strategy (URL-based vs header-based)
- Health check and readiness endpoints

#### Controllers / Handlers

- Consistent request/response shape
- Input validation at the controller level
- Error handling that never leaks internal details

#### Configs

- Environment-specific configuration (dev, staging, production)
- Validation of required config values at startup
- Fail fast if required config is missing

#### Env and YAML Variable Loader

- Load `.env` for local development
- Load YAML/JSON config files for structured configuration
- Never commit `.env` files to version control
- Provide `.env.example` with all required variables documented

#### Middlewares (Optional)

- Authentication / Authorization
- Rate limiting
- CORS
- Request timeout
- Compression
- Only include middlewares that the service actually needs

---

### Step 3 - Folder Structure

> Provide the bootstrapped folder structure

```
src/
  config/
    env.ts
    database.ts
    logger.ts
  controllers/
  services/
  repositories/
  middlewares/
  routes/
  app.ts
  server.ts
.env.example
```

---

### Step 4 - Bootstrap Order

> The initialization sequence must follow dependency order

1. Load environment variables and validate config
2. Initialize logger
3. Initialize database connections
4. Initialize cache connections
5. Initialize message queue connections
6. Register middlewares
7. Register routes
8. Start server

---

### Step 5 - Output

> Provide the bootstrap recommendation with the actual folder structure

- List which bootstrap items are included and excluded (with justification)
- Provide the folder structure
- Provide the initialization order

---

## Edge Cases

### 1. Service That Does Not Need HTTP

> Worker processes, cron jobs, or event consumers may not need controllers or routing

**Example:**

- A queue consumer that processes background jobs

**Rule of Thumb:**

- Only bootstrap what the service type requires
- A worker needs: config, logging, database, queue connection
- A worker does NOT need: routing, controllers, CORS, rate limiting

**Practical Trade-off:**

- Shared bootstrap between service types vs specialized bootstrap per service
- Specialized bootstrap is cleaner and faster to start

---

### 2. Graceful Shutdown

> The service must shut down cleanly without losing in-flight requests

**Example:**

- SIGTERM received, drain connections, finish processing, then exit

**Rule of Thumb:**

- Always implement graceful shutdown for production services
- Close database connections, message queue connections, and HTTP server in reverse order of initialization

**Practical Trade-off:**

- Extra shutdown logic vs potential data loss on deployment

---

## Bad Example

### 1. Everything in One File

```typescript
// server.ts
import express from "express";
import dotenv from "dotenv";
import { Client } from "pg";
import Redis from "ioredis";

dotenv.config();

const app = express();
const db = new Client(process.env.DATABASE_URL);
const redis = new Redis(process.env.REDIS_URL);

app.get("/users", async (req, res) => {
  const result = await db.query("SELECT * FROM users");
  res.json(result.rows);
});

app.listen(3000, () => console.log("Server running on port 3000"));
```

### 2. Missing Config Validation

```typescript
// No validation — silently uses undefined
const dbUrl = process.env.DATABASE_URL;
// Crashes at runtime when DB URL is missing
```

---

## Good Example

### 1. Structured Bootstrap

```typescript
// config/env.ts
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z.enum(["development", "staging", "production"]),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url().optional(),
});

export const env = envSchema.parse(process.env);
```

```typescript
// app.ts
import { env } from "./config/env";
import { logger } from "./config/logger";
import { database } from "./config/database";
import { createApp } from "./app";

async function bootstrap() {
  const db = await database.connect(env.DATABASE_URL);
  const app = createApp({ db, logger });

  app.listen(env.PORT, () => {
    logger.info(`Server running on port ${env.PORT}`);
  });
}

bootstrap().catch((err) => {
  logger.error("Failed to start server", err);
  process.exit(1);
});
```

### 2. Graceful Shutdown

```typescript
// server.ts
async function gracefulShutdown(signal: string) {
  logger.info(`${signal} received, shutting down gracefully`);
  server.close(() => {
    db.disconnect();
    logger.info("Server closed");
    process.exit(0);
  });

  setTimeout(() => {
    logger.error("Forced shutdown after timeout");
    process.exit(1);
  }, 10000);
}

process.on("SIGTERM", () => gracefulShutdown("SIGTERM"));
process.on("SIGINT", () => gracefulShutdown("SIGINT"));
```
