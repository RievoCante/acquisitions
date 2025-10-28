# Docker Setup Summary

This document provides a quick overview of the Docker containerization setup for the Acquisitions backend service.

## What Was Added

### Core Files

1. **Dockerfile** - Multi-stage production-ready container image
2. **docker-compose.dev.yml** - Development environment with Neon Local
3. **docker-compose.prod.yml** - Production environment with Neon Cloud
4. **.dockerignore** - Excludes unnecessary files from Docker builds

### Documentation

1. **DOCKER.md** - Comprehensive Docker documentation
2. **DOCKER_QUICKSTART.md** - Quick start guide (5 minutes)
3. **SETUP_ENV.md** - Environment variables setup guide
4. **README_DOCKER.md** - This file

### Package.json Scripts

New npm scripts added for Docker operations:

- `npm start` - Production start command
- `npm run docker:dev` - Start development environment
- `npm run docker:dev:build` - Build and start development
- `npm run docker:prod` - Start production environment
- `npm run docker:prod:build` - Build and start production
- `npm run docker:down` - Stop all containers
- `npm run docker:logs` - View container logs

## Architecture

### Development Environment

```
┌─────────────────────────────────────┐
│   Docker Compose (Development)      │
│                                     │
│  ┌──────────────┐  ┌─────────────┐ │
│  │     App      │  │ Neon Local  │ │
│  │  (Express)   │──│ (Postgres)  │ │
│  │  Port: 3000  │  │ Port: 5432  │ │
│  └──────────────┘  └─────────────┘ │
│         │                           │
│    Hot Reload                       │
│    ./src mounted                    │
└─────────────────────────────────────┘
```

### Production Environment

```
┌─────────────────────────────────────┐
│   Docker Compose (Production)       │
│                                     │
│  ┌──────────────┐                  │
│  │     App      │─────────────────┼──→ Neon Cloud
│  │  (Express)   │  (External DB)   │   (Postgres)
│  │  Port: 3000  │                  │
│  └──────────────┘                  │
│                                     │
│  Optimized build                    │
│  No source mounting                 │
└─────────────────────────────────────┘
```

## Quick Start

### 1. Set Up Environment Variables

Create `.env.development`:

```bash
cat > .env.development << 'EOF'
NODE_ENV=development
PORT=3000
DATABASE_URL=postgres://neon@neon-local:5432/main
ARCJET_KEY=your_dev_arcjet_key
JWT_SECRET=dev_secret_change_me
LOG_LEVEL=debug
EOF
```

See [SETUP_ENV.md](./SETUP_ENV.md) for detailed instructions.

### 2. Start Development Environment

```bash
npm run docker:dev:build
```

### 3. Verify

```bash
curl http://localhost:3000/health
```

## Key Features

### Multi-Stage Dockerfile

- **Stage 1**: Installs production dependencies
- **Stage 2**: Creates optimized production image
- **Benefits**: Smaller image size, faster deployments

### Security

- Non-root user (nodejs:nodejs)
- Minimal Alpine Linux base
- No unnecessary dependencies
- Proper signal handling with dumb-init

### Development Experience

- Hot reload with `--watch` flag
- Source code mounted for instant updates
- Persistent logs directory
- Isolated database with Neon Local

### Production Ready

- Health checks configured
- Resource limits defined
- Auto-restart on failure
- Optimized for cloud deployment

## Environment Variables

| Variable       | Development   | Production    | Description             |
| -------------- | ------------- | ------------- | ----------------------- |
| `NODE_ENV`     | `development` | `production`  | Application environment |
| `PORT`         | `3000`        | `3000`        | Server port             |
| `DATABASE_URL` | Neon Local    | Neon Cloud    | PostgreSQL connection   |
| `ARCJET_KEY`   | Dev key       | Prod key      | Arcjet security API key |
| `JWT_SECRET`   | Any string    | Strong random | JWT signing secret      |
| `LOG_LEVEL`    | `debug`       | `info`        | Winston log level       |

## Common Tasks

### Development

```bash
# Start with build
npm run docker:dev:build

# Start without build
npm run docker:dev

# View logs
npm run docker:logs

# Run migrations
docker-compose -f docker-compose.dev.yml exec app npm run db:migrate

# Access container
docker-compose -f docker-compose.dev.yml exec app sh

# Access database
docker-compose -f docker-compose.dev.yml exec neon-local psql -U neon -d main
```

### Production

```bash
# Deploy
npm run docker:prod:build

# Run in background
docker-compose -f docker-compose.prod.yml up -d --build

# View logs
docker-compose -f docker-compose.prod.yml logs -f app

# Run migrations
docker-compose -f docker-compose.prod.yml exec app npm run db:migrate
```

### Cleanup

```bash
# Stop all containers
npm run docker:down

# Remove containers and volumes
docker-compose -f docker-compose.dev.yml down -v

# Clean Docker system
docker system prune -a --volumes
```

## File Structure

```
acquisitions/
├── Dockerfile                    # Multi-stage container image
├── docker-compose.dev.yml        # Development with Neon Local
├── docker-compose.prod.yml       # Production with Neon Cloud
├── .dockerignore                # Excluded files from build
├── .env.development             # Dev environment vars (create this)
├── .env.production              # Prod environment vars (create this)
│
├── DOCKER.md                    # Comprehensive documentation
├── DOCKER_QUICKSTART.md         # Quick start guide
├── SETUP_ENV.md                 # Environment setup guide
└── README_DOCKER.md             # This file
```

## Differences: Development vs Production

| Aspect       | Development            | Production               |
| ------------ | ---------------------- | ------------------------ |
| Database     | Neon Local (container) | Neon Cloud (external)    |
| Source Code  | Mounted (hot reload)   | Copied (immutable)       |
| Command      | `node --watch`         | `node`                   |
| Logging      | Debug level            | Info level               |
| Restart      | `unless-stopped`       | `always`                 |
| Resources    | No limits              | CPU/Memory limits        |
| Health Check | Via Dockerfile         | Via compose + Dockerfile |

## Troubleshooting

### Container won't start

```bash
# Check logs
npm run docker:logs

# Verify environment file
cat .env.development

# Restart
npm run docker:down
npm run docker:dev:build
```

### Database connection failed

```bash
# Check neon-local health
docker-compose -f docker-compose.dev.yml ps

# Test connection
docker-compose -f docker-compose.dev.yml exec neon-local pg_isready -U neon
```

### Hot reload not working

```bash
# Restart app container
docker-compose -f docker-compose.dev.yml restart app
```

## Best Practices

1. **Never commit** `.env.development` or `.env.production`
2. **Use strong secrets** for production JWT_SECRET
3. **Update base images** regularly for security patches
4. **Test migrations** in development before production
5. **Monitor logs** and resource usage in production
6. **Use Docker secrets** or a secrets manager for sensitive values

## Next Steps

1. **Read Documentation**: Start with [DOCKER_QUICKSTART.md](./DOCKER_QUICKSTART.md)
2. **Set Up Secrets**: Follow [SETUP_ENV.md](./SETUP_ENV.md)
3. **Deploy**: Use [DOCKER.md](./DOCKER.md) for deployment guide
4. **Configure CI/CD**: Integrate with your pipeline

## Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Neon Database](https://neon.tech/docs)
- [Neon Local](https://neon.tech/docs/local/neon-local)

## Support

For help:

1. Check [DOCKER.md](./DOCKER.md) troubleshooting section
2. Review logs: `npm run docker:logs`
3. Verify environment setup: [SETUP_ENV.md](./SETUP_ENV.md)
