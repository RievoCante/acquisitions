# Docker Quick Start Guide

Get your Acquisitions backend running with Docker in under 5 minutes!

## Prerequisites

- Docker and Docker Compose installed
- Git (to clone the repository)

## Development Setup (3 Steps)

### 1. Create Environment File

Create `.env.development` in the project root:

```bash
cat > .env.development << 'EOF'
NODE_ENV=development
PORT=3000
DATABASE_URL=postgres://neon@neon-local:5432/main
ARCJET_KEY=your_dev_arcjet_key
JWT_SECRET=dev_secret_change_in_production
LOG_LEVEL=debug
EOF
```

### 2. Start the Application

```bash
npm run docker:dev:build
```

### 3. Verify It's Running

Open your browser or use curl:

```bash
curl http://localhost:3000/health
```

You should see:

```json
{
  "status": "ok",
  "timestamp": "2024-10-28T...",
  "uptime": 5.123
}
```

## What's Running?

- **App Container** (`acquisitions-app-dev`): Your Express backend on port 3000
- **Neon Local Container** (`acquisitions-neon-local`): PostgreSQL database on port 5432

## Common Commands

```bash
# Start (with build)
npm run docker:dev:build

# Start (without build)
npm run docker:dev

# Stop all containers
npm run docker:down

# View logs
npm run docker:logs

# Run migrations
docker-compose -f docker-compose.dev.yml exec app npm run db:migrate

# Access app container shell
docker-compose -f docker-compose.dev.yml exec app sh

# Access database
docker-compose -f docker-compose.dev.yml exec neon-local psql -U neon -d main
```

## Production Deployment

### 1. Create Production Environment File

Create `.env.production` with your Neon Cloud credentials:

```bash
cat > .env.production << 'EOF'
NODE_ENV=production
PORT=3000
DATABASE_URL=postgres://user:password@ep-xxx.neon.tech/dbname?sslmode=require
ARCJET_KEY=your_production_arcjet_key
JWT_SECRET=generate_a_strong_random_secret_here
LOG_LEVEL=info
EOF
```

**Important**: Replace all placeholder values with your actual production credentials!

### 2. Deploy

```bash
npm run docker:prod:build
```

## Troubleshooting

### Port already in use

```bash
# Stop existing containers
npm run docker:down

# Or find and stop the process using port 3000
lsof -ti:3000 | xargs kill -9
```

### Database connection failed

```bash
# Check if neon-local is healthy
docker-compose -f docker-compose.dev.yml ps

# Restart if needed
docker-compose -f docker-compose.dev.yml restart neon-local
```

### Clear everything and start fresh

```bash
npm run docker:down
docker-compose -f docker-compose.dev.yml down -v
npm run docker:dev:build
```

## Next Steps

- Read the full [DOCKER.md](./DOCKER.md) for detailed documentation
- Configure your actual Arcjet and JWT secrets
- Set up database migrations
- Configure CI/CD pipelines

## File Structure

```
acquisitions/
├── Dockerfile                 # Multi-stage build configuration
├── docker-compose.dev.yml     # Development with Neon Local
├── docker-compose.prod.yml    # Production with Neon Cloud
├── .dockerignore             # Files excluded from Docker build
├── .env.development          # Dev environment variables (create this)
├── .env.production           # Prod environment variables (create this)
└── DOCKER.md                 # Comprehensive documentation
```

## Support

Having issues? Check:

1. Docker is running: `docker ps`
2. No port conflicts: `lsof -ti:3000`
3. Environment file exists: `ls -la .env.development`
4. Container logs: `npm run docker:logs`

For more help, see [DOCKER.md](./DOCKER.md) troubleshooting section.
