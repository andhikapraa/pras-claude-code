---
description: Generate deployment configuration using Nixpacks for various platforms
model: claude-opus-4-5
---

Generate deployment configuration and guide for the following application.

## Deployment Target

$ARGUMENTS

---

## MCP Tools Integration

**IMPORTANT:** Use these tools for deployment planning:

### Required Tools
1. **Sequential Thinking** - Plan deployment steps systematically
2. **Context7** - Fetch latest deployment best practices for the framework

---

## What is Nixpacks?

Nixpacks is a build system that automatically detects your project type and creates optimized Docker images. It's used by Railway, Render, and can be self-hosted.

**Key Benefits:**
- Zero-config builds (auto-detects framework)
- Reproducible builds with Nix
- Optimized caching
- Multi-platform support

---

## Nixpacks Configuration

### Basic `nixpacks.toml`

```toml
# nixpacks.toml

# Providers (auto-detected, but can be explicit)
providers = ["node"]

# Nix packages to install
[phases.setup]
nixPkgs = ["nodejs_20", "pnpm"]
# aptPkgs = ["libpq-dev"]  # For system libraries

# Install dependencies
[phases.install]
cmds = ["pnpm install --frozen-lockfile"]

# Build the application
[phases.build]
cmds = ["pnpm build"]

# Start command
[start]
cmd = "pnpm start"
```

### Framework-Specific Configs

#### Next.js

```toml
# nixpacks.toml for Next.js
providers = ["node"]

[phases.setup]
nixPkgs = ["nodejs_20", "pnpm"]

[phases.install]
cmds = ["pnpm install --frozen-lockfile"]

[phases.build]
cmds = ["pnpm build"]

# Environment variables available at build time
[variables]
NEXT_TELEMETRY_DISABLED = "1"

[start]
cmd = "pnpm start"
```

#### Node.js API

```toml
# nixpacks.toml for Node.js API
providers = ["node"]

[phases.setup]
nixPkgs = ["nodejs_20", "pnpm"]

[phases.install]
cmds = ["pnpm install --frozen-lockfile --prod"]

[start]
cmd = "node dist/index.js"
```

#### Python (FastAPI/Django)

```toml
# nixpacks.toml for Python
providers = ["python"]

[phases.setup]
nixPkgs = ["python311", "poetry"]

[phases.install]
cmds = ["poetry install --no-dev"]

[start]
cmd = "uvicorn main:app --host 0.0.0.0 --port $PORT"
```

#### Rust

```toml
# nixpacks.toml for Rust
providers = ["rust"]

[phases.setup]
nixPkgs = ["rustc", "cargo"]

[phases.build]
cmds = ["cargo build --release"]

[start]
cmd = "./target/release/app"
```

---

## Platform Deployment Guides

### Railway (Native Nixpacks)

```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Deploy
railway up
```

**railway.json** (optional):
```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### Render

```yaml
# render.yaml
services:
  - type: web
    name: my-app
    env: node
    buildCommand: pnpm install && pnpm build
    startCommand: pnpm start
    envVars:
      - key: NODE_ENV
        value: production
    healthCheckPath: /api/health
```

### Fly.io

```bash
# Install flyctl
curl -L https://fly.io/install.sh | sh

# Login
fly auth login

# Launch (creates fly.toml)
fly launch

# Deploy
fly deploy
```

**fly.toml**:
```toml
app = "my-app"
primary_region = "sea"

[build]
  builder = "nixpacks"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true

[[services]]
  protocol = "tcp"
  internal_port = 3000

  [[services.ports]]
    port = 80
    handlers = ["http"]

  [[services.ports]]
    port = 443
    handlers = ["tls", "http"]

  [services.concurrency]
    type = "connections"
    hard_limit = 25
    soft_limit = 20

  [[services.http_checks]]
    interval = 10000
    grace_period = "5s"
    method = "get"
    path = "/api/health"
    protocol = "http"
    timeout = 2000
```

### Docker (Self-Hosted)

```bash
# Build with Nixpacks
nixpacks build . --name my-app

# Run locally
docker run -p 3000:3000 my-app

# Push to registry
docker tag my-app registry.example.com/my-app:latest
docker push registry.example.com/my-app:latest
```

**docker-compose.yml** (for local dev/staging):
```yaml
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
```

---

## Environment Variables

### `.env.example`

```bash
# App
NODE_ENV=production
PORT=3000

# Database
DATABASE_URL=postgresql://user:pass@host:5432/db

# Auth
AUTH_SECRET=generate-with-openssl-rand-base64-32
NEXTAUTH_URL=https://your-domain.com

# External Services
STRIPE_SECRET_KEY=sk_live_xxx
OPENAI_API_KEY=sk-xxx

# Feature Flags
ENABLE_ANALYTICS=true
```

### Platform Environment Setup

```bash
# Railway
railway variables set KEY=value

# Fly.io
fly secrets set KEY=value

# Render (via dashboard or render.yaml)
```

---

## Health Checks

```typescript
// app/api/health/route.ts (Next.js)
import { NextResponse } from 'next/server';

export async function GET() {
  try {
    // Check database connection
    // await db.query('SELECT 1');

    return NextResponse.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      version: process.env.npm_package_version,
    });
  } catch (error) {
    return NextResponse.json(
      { status: 'unhealthy', error: String(error) },
      { status: 503 }
    );
  }
}
```

---

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Railway
        uses: railwayapp/railway-deploy@v1
        with:
          token: ${{ secrets.RAILWAY_TOKEN }}
          service: my-app
```

---

## Output Format

### Deployment Checklist

```markdown
## Pre-Deployment
- [ ] Environment variables configured
- [ ] Database migrations ready
- [ ] Health check endpoint working
- [ ] Tests passing
- [ ] Build successful locally

## Deployment Files Created
- [ ] nixpacks.toml
- [ ] Platform config (fly.toml/railway.json/render.yaml)
- [ ] .env.example updated
- [ ] Health check endpoint

## Post-Deployment
- [ ] Verify health check
- [ ] Test critical flows
- [ ] Monitor logs
- [ ] Set up alerts
```

---

## Best Practices

**DO:**
- Use environment variables for all secrets
- Implement health checks
- Set up monitoring and logging
- Use staging environment first
- Enable automatic rollbacks
- Use immutable deployments

**DON'T:**
- Hardcode secrets
- Skip health checks
- Deploy without testing
- Ignore resource limits
- Forget to set up backups

---

Generate complete deployment configuration for the target platform.

---
*Originally created by [Pras](https://github.com/andhikapraa/pras-claude-code)*
