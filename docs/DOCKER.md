# Docker Setup Guide

This guide covers how to containerize and run the Acquisitions backend service using Docker and Docker Compose with different configurations for development and production environments.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Architecture Overview](#architecture-overview)
- [Development Setup](#development-setup)
- [Production Deployment](#production-deployment)
- [Common Commands](#common-commands)
- [Environment Variables](#environment-variables)
- [Database Migrations](#database-migrations)
- [Troubleshooting](#troubleshooting)

## Prerequisites

Before you begin, ensure you have the following installed:

- **Docker**: Version 20.10 or higher ([Install Docker](https://docs.docker.com/get-docker/))
- **Docker Compose**: Version 2.0 or higher (included with Docker Desktop)
- **Node.js**: Version 20 or higher (for local development without Docker)

## Architecture Overview

### Development Environment

- **Application Container**: Node.js Express backend with hot reload
- **Neon Local Container**: Local PostgreSQL database that mimics Neon's ephemeral branching
- **Network**: Bridge network for container communication
- **Volumes**: Source code mounted for hot reload, persistent logs

### Production Environment

- **Application Container**: Optimized Node.js Express backend
- **Database**: Neon Cloud (serverless PostgreSQL)
- **Network**: Bridge network
- **Volumes**: Logs only (no source code mounting)

## Development Setup

### Step 1: Create Development Environment File

Create a `.env.development` file in the project root:

```bash
cp .env.development.example .env.development
```

Edit `.env.development` with your development credentials:

```env
NODE_ENV=development
PORT=3000
DATABASE_URL=postgres://neon@neon-local:5432/main
ARCJET_KEY=your_dev_arcjet_key
JWT_SECRET=your_dev_jwt_secret_change_me
LOG_LEVEL=debug
```

> **Note**: The `DATABASE_URL` points to the Neon Local container. Do not change the hostname `neon-local`.

### Step 2: Start Development Environment

Using npm scripts (recommended):

```bash
npm run docker:dev
```

Or using Docker Compose directly:

```bash
docker-compose -f docker-compose.dev.yml up
```

To rebuild the containers (after dependency changes):

```bash
npm run docker:dev:build
```

Or:

```bash
docker-compose -f docker-compose.dev.yml up --build
```

### Step 3: Verify Development Setup

Once the containers are running, verify the setup:

1. **Check application health**:

   ```bash
   curl http://localhost:3000/health
   ```

2. **Check database connection**:

   ```bash
   docker-compose -f docker-compose.dev.yml exec neon-local psql -U neon -d main -c "SELECT version();"
   ```

3. **View logs**:
   ```bash
   npm run docker:logs
   ```

### Development Features

- **Hot Reload**: Source code changes are automatically detected and the application restarts
- **Persistent Logs**: Logs are stored in the `./logs` directory on your host machine
- **Database Persistence**: Database data persists between container restarts via Docker volumes
- **Isolated Environment**: All services run in an isolated Docker network

## Production Deployment

### Step 1: Create Production Environment File

Create a `.env.production` file in the project root:

```bash
cp .env.production.example .env.production
```

Edit `.env.production` with your production credentials:

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=postgres://user:password@ep-xxx-xxx.region.aws.neon.tech/dbname?sslmode=require
ARCJET_KEY=your_production_arcjet_key
JWT_SECRET=your_production_jwt_secret_use_strong_random_value
LOG_LEVEL=info
```

> **Important**:
>
> - Use your actual Neon Cloud connection string for `DATABASE_URL`
> - Generate a strong, random JWT secret (minimum 32 characters)
> - Never commit the `.env.production` file to version control

### Step 2: Build and Start Production Environment

Using npm scripts (recommended):

```bash
npm run docker:prod:build
```

Or using Docker Compose directly:

```bash
docker-compose -f docker-compose.prod.yml up --build
```

Run in detached mode (background):

```bash
docker-compose -f docker-compose.prod.yml up -d --build
```

### Step 3: Verify Production Deployment

1. **Check application health**:

   ```bash
   curl http://localhost:3000/health
   ```

2. **Monitor logs**:
   ```bash
   docker-compose -f docker-compose.prod.yml logs -f app
   ```

### Production Features

- **Multi-stage Build**: Optimized Docker image with minimal size
- **Non-root User**: Application runs as unprivileged user for security
- **Health Checks**: Automatic container health monitoring
- **Resource Limits**: CPU and memory constraints defined
- **Auto-restart**: Container automatically restarts on failure
- **Neon Cloud**: Serverless PostgreSQL with automatic scaling

## Common Commands

### Start Services

```bash
# Development
npm run docker:dev

# Production
npm run docker:prod
```

### Stop Services

```bash
# Stop all containers
npm run docker:down

# Or for specific environment
docker-compose -f docker-compose.dev.yml down
docker-compose -f docker-compose.prod.yml down
```

### View Logs

```bash
# All services
npm run docker:logs

# Specific service
docker-compose -f docker-compose.dev.yml logs -f app
docker-compose -f docker-compose.dev.yml logs -f neon-local
```

### Access Container Shell

```bash
# Development
docker-compose -f docker-compose.dev.yml exec app sh

# Production
docker-compose -f docker-compose.prod.yml exec app sh
```

### Access Database

```bash
# Development (Neon Local)
docker-compose -f docker-compose.dev.yml exec neon-local psql -U neon -d main
```

### Rebuild Containers

```bash
# Development
npm run docker:dev:build

# Production
npm run docker:prod:build
```

### Remove All Containers and Volumes

```bash
# Development
docker-compose -f docker-compose.dev.yml down -v

# Production
docker-compose -f docker-compose.prod.yml down -v
```

## Environment Variables

### Required Variables

| Variable       | Description                  | Development   | Production          |
| -------------- | ---------------------------- | ------------- | ------------------- |
| `NODE_ENV`     | Application environment      | `development` | `production`        |
| `PORT`         | Application port             | `3000`        | `3000`              |
| `DATABASE_URL` | PostgreSQL connection string | Neon Local    | Neon Cloud          |
| `ARCJET_KEY`   | Arcjet security API key      | Dev key       | Prod key            |
| `JWT_SECRET`   | JWT signing secret           | Any string    | Strong random value |
| `LOG_LEVEL`    | Winston log level            | `debug`       | `info` or `warn`    |

### Optional Variables

You can add additional environment variables to the `.env.development` or `.env.production` files as needed. They will be automatically passed to the application container.

## Database Migrations

### Running Migrations in Development

```bash
# Generate migration files
docker-compose -f docker-compose.dev.yml exec app npm run db:generate

# Apply migrations
docker-compose -f docker-compose.dev.yml exec app npm run db:migrate

# Open Drizzle Studio
docker-compose -f docker-compose.dev.yml exec app npm run db:studio
```

### Running Migrations in Production

```bash
# Apply migrations
docker-compose -f docker-compose.prod.yml exec app npm run db:migrate
```

> **Best Practice**: Run migrations as a separate step before deploying new application versions.

### Migration Strategy

1. **Local Development**:
   - Generate migrations against Neon Local
   - Test migrations thoroughly
   - Commit migration files to version control

2. **Production Deployment**:
   - Deploy migration files first
   - Run migrations using a separate job or init container
   - Deploy application containers

## Troubleshooting

### Container Won't Start

**Problem**: Container exits immediately after starting.

**Solution**:

1. Check logs for errors:
   ```bash
   docker-compose -f docker-compose.dev.yml logs app
   ```
2. Verify environment variables are set correctly
3. Check if port 3000 is already in use

### Database Connection Failed

**Problem**: Application can't connect to database.

**Solution**:

1. Verify `DATABASE_URL` is correct
2. For development, ensure `neon-local` container is running:
   ```bash
   docker-compose -f docker-compose.dev.yml ps
   ```
3. Check database container health:
   ```bash
   docker-compose -f docker-compose.dev.yml exec neon-local pg_isready -U neon
   ```

### Hot Reload Not Working (Development)

**Problem**: Code changes don't trigger application restart.

**Solution**:

1. Ensure source code is mounted correctly (check `docker-compose.dev.yml`)
2. Restart the containers:
   ```bash
   docker-compose -f docker-compose.dev.yml restart app
   ```

### Permission Denied for Logs Directory

**Problem**: Application can't write to logs directory.

**Solution**:

1. Ensure logs directory exists and has correct permissions:
   ```bash
   mkdir -p logs
   chmod 755 logs
   ```

### Out of Memory Errors

**Problem**: Container crashes due to memory limits.

**Solution**:

1. Increase memory limits in `docker-compose.prod.yml`:
   ```yaml
   deploy:
     resources:
       limits:
         memory: 1G
   ```

### Cleaning Up Docker Resources

If you're running out of disk space:

```bash
# Remove unused containers, networks, and volumes
docker system prune -a --volumes

# Or be more selective
docker container prune
docker volume prune
docker network prune
```

## Best Practices

1. **Never commit `.env.development` or `.env.production`** to version control
2. **Use strong, randomly generated secrets** for production JWT_SECRET
3. **Regularly update base Docker images** for security patches
4. **Monitor container resource usage** in production
5. **Implement proper logging and monitoring** solutions
6. **Use Docker secrets** or a secrets manager for sensitive production values
7. **Test migrations thoroughly** in development before production deployment
8. **Keep development and production configs separate** to prevent accidental data loss

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Neon Documentation](https://neon.tech/docs)
- [Neon Local Documentation](https://neon.tech/docs/local/neon-local)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

## Support

For issues or questions:

- Check the [Troubleshooting](#troubleshooting) section
- Review application logs: `npm run docker:logs`
- Consult the project README.md
