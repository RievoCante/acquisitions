# Environment Variables Setup

This guide helps you create the necessary environment variable files for Docker development and production.

## Why You Need This

The `.env.development` and `.env.production` files are excluded from version control for security. You need to create them manually before running the Docker containers.

## Development Environment

### Create `.env.development`

Run this command in the project root:

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

Or create the file manually with this content:

```env
NODE_ENV=development
PORT=3000
DATABASE_URL=postgres://neon@neon-local:5432/main
ARCJET_KEY=your_dev_arcjet_key
JWT_SECRET=dev_secret_change_in_production
LOG_LEVEL=debug
```

### Configuration Notes

- **DATABASE_URL**: Points to `neon-local` container. Do NOT change this hostname.
- **ARCJET_KEY**: Get your development key from [Arcjet Dashboard](https://app.arcjet.com)
- **JWT_SECRET**: Any string is fine for development (but change it for production!)
- **LOG_LEVEL**: Set to `debug` for verbose logging during development

## Production Environment

### Create `.env.production`

Run this command in the project root:

```bash
cat > .env.production << 'EOF'
NODE_ENV=production
PORT=3000
DATABASE_URL=postgres://user:password@ep-xxx-xxx.region.aws.neon.tech/dbname?sslmode=require
ARCJET_KEY=your_production_arcjet_key
JWT_SECRET=your_production_jwt_secret_use_strong_random_value
LOG_LEVEL=info
EOF
```

Or create the file manually with this content:

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=postgres://user:password@ep-xxx-xxx.region.aws.neon.tech/dbname?sslmode=require
ARCJET_KEY=your_production_arcjet_key
JWT_SECRET=your_production_jwt_secret_use_strong_random_value
LOG_LEVEL=info
```

### Critical Production Configuration

#### 1. DATABASE_URL (Neon Cloud)

Get your connection string from [Neon Console](https://console.neon.tech):

1. Go to your Neon project
2. Navigate to "Connection Details"
3. Copy the connection string
4. Ensure it includes `?sslmode=require`

Example format:

```
postgres://username:password@ep-cool-river-123456.us-east-2.aws.neon.tech/dbname?sslmode=require
```

#### 2. ARCJET_KEY (Production)

Get your production API key from [Arcjet Dashboard](https://app.arcjet.com):

1. Create a new site/application for production
2. Copy the production API key
3. Paste it in `.env.production`

#### 3. JWT_SECRET (Strong Random Value)

Generate a secure random secret (minimum 32 characters):

```bash
# Using OpenSSL
openssl rand -base64 32

# Using Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"

# Using /dev/urandom (Linux/Mac)
head -c 32 /dev/urandom | base64
```

**Important**: Never reuse your development JWT secret in production!

#### 4. LOG_LEVEL

Set to `info` or `warn` for production to reduce log noise:

- `error`: Only errors
- `warn`: Warnings and errors
- `info`: Info, warnings, and errors (recommended)
- `debug`: All logs (not recommended for production)

## Local Development (Without Docker)

If you're running the app locally without Docker, create a `.env` file:

```bash
cat > .env << 'EOF'
NODE_ENV=development
PORT=3000
DATABASE_URL=postgres://user:password@localhost:5432/acquisitions
ARCJET_KEY=your_arcjet_key
JWT_SECRET=your_jwt_secret
LOG_LEVEL=debug
EOF
```

Replace `DATABASE_URL` with your local PostgreSQL or Neon Cloud connection string.

## Verify Your Setup

### Check File Exists

```bash
# Development
ls -la .env.development

# Production
ls -la .env.production
```

### Test Configuration

```bash
# Development
npm run docker:dev

# Production
npm run docker:prod
```

## Security Best Practices

1. **Never commit** `.env.development` or `.env.production` to version control
2. **Use different secrets** for development and production
3. **Rotate secrets** regularly (especially after team member changes)
4. **Use a secrets manager** (AWS Secrets Manager, HashiCorp Vault, etc.) for production
5. **Restrict access** to production environment files
6. **Audit** who has access to production credentials

## Troubleshooting

### "Environment variable not set" error

**Problem**: Docker container can't read environment variables.

**Solution**:

1. Ensure `.env.development` or `.env.production` exists
2. Check file has correct formatting (no quotes around values unless needed)
3. Restart Docker containers: `npm run docker:down && npm run docker:dev`

### "Cannot connect to database" error

**Problem**: Wrong DATABASE_URL format or incorrect credentials.

**Solution**:

1. Verify DATABASE_URL format is correct
2. For development, ensure it points to `neon-local:5432`
3. For production, verify Neon Cloud credentials are correct
4. Check database is accessible: `docker-compose -f docker-compose.dev.yml exec neon-local pg_isready -U neon`

### "JWT secret too short" warning

**Problem**: JWT_SECRET is not secure enough.

**Solution**:
Generate a new secure secret (minimum 32 characters):

```bash
openssl rand -base64 32
```

## Additional Resources

- [12-Factor App: Configuration](https://12factor.net/config)
- [Neon Console](https://console.neon.tech)
- [Arcjet Dashboard](https://app.arcjet.com)
- [Docker Environment Variables](https://docs.docker.com/compose/environment-variables/)

## Next Steps

After creating your environment files:

1. Start development: `npm run docker:dev:build`
2. Read [DOCKER_QUICKSTART.md](./DOCKER_QUICKSTART.md) for quick commands
3. Read [DOCKER.md](./DOCKER.md) for comprehensive documentation
