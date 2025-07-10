# Deployment Guide

This guide covers deploying OpenCode in various environments, from local development to production infrastructure.

## Overview

OpenCode consists of multiple deployable components:

- **Core Server**: TypeScript/Node.js API server
- **TUI Client**: Go binary distributed to end users
- **Documentation**: Static website
- **Cloud Functions**: Serverless edge functions

## Local Development

### Quick Start

```bash
# Install dependencies
bun install

# Start the server
bun run packages/opencode/src/index.ts

# In another terminal, run the TUI
cd packages/tui
go run ./cmd/opencode
```

### Development Configuration

Create a local configuration file:

```bash
mkdir -p ~/.config/opencode
cat > ~/.config/opencode/config.json << EOF
{
  "providers": {
    "anthropic": {
      "apiKey": "your-anthropic-key"
    },
    "openai": {
      "apiKey": "your-openai-key"
    }
  },
  "server": {
    "port": 3000,
    "host": "localhost"
  }
}
EOF
```

## Production Deployment

### Server Deployment (SST + Cloudflare)

OpenCode uses [SST](https://sst.dev) for infrastructure as code with Cloudflare as the deployment platform.

#### Prerequisites

```bash
# Install SST
npm install -g sst

# Configure Cloudflare credentials
export CLOUDFLARE_API_TOKEN="your-token"
```

#### Deployment Commands

```bash
# Deploy to development
sst deploy --stage dev

# Deploy to production  
sst deploy --stage production

# Check deployment status
sst status --stage production
```

#### Infrastructure Configuration

The `sst.config.ts` defines the infrastructure:

```typescript
export default $config({
  app(input) {
    return {
      name: "opencode",
      removal: input?.stage === "production" ? "retain" : "remove",
      protect: ["production"].includes(input?.stage),
      home: "cloudflare",
    }
  },
  async run() {
    const { api } = await import("./infra/app.js")
    return {
      api: api.url,
    }
  },
})
```

### TUI Client Distribution

The TUI client is distributed as compiled binaries through multiple channels:

#### GitHub Releases

Automated builds create binaries for multiple platforms:

```bash
# Release process (automated via GitHub Actions)
git tag v0.1.0
git push origin v0.1.0

# Binaries created for:
# - linux/amd64
# - linux/arm64  
# - darwin/amd64
# - darwin/arm64
# - windows/amd64
```

#### Package Managers

**npm/bun/pnpm/yarn:**
```bash
npm install -g opencode-ai@latest
```

**Homebrew (macOS):**
```bash
brew install sst/tap/opencode
```

**Arch Linux:**
```bash
paru -S opencode-bin
```

#### Direct Installation

```bash
# YOLO install script
curl -fsSL https://opencode.ai/install | bash
```

### Documentation Deployment

The documentation site is deployed as a static site:

```bash
cd packages/web
npm run build
npm run deploy  # Deploy to Cloudflare Pages
```

## Environment Configuration

### Environment Variables

**Server Environment:**
```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your-key
OPENAI_API_KEY=your-key
GOOGLE_API_KEY=your-key

# Server Configuration
PORT=3000
HOST=0.0.0.0
LOG_LEVEL=info

# Database (if applicable)
DATABASE_URL=postgresql://...

# Authentication
OPENAUTH_SECRET=your-secret
GITHUB_CLIENT_ID=your-id
GITHUB_CLIENT_SECRET=your-secret
```

**Client Environment:**
```bash
# Server Connection
OPENCODE_SERVER=https://api.opencode.ai
OPENCODE_API_KEY=user-token

# TUI Configuration
OPENCODE_THEME=dark
OPENCODE_CONFIG_DIR=~/.config/opencode
```

### Configuration Files

**Server Configuration** (`config.json`):
```json
{
  "server": {
    "port": 3000,
    "cors": {
      "origins": ["https://opencode.ai"]
    }
  },
  "providers": {
    "anthropic": {
      "baseURL": "https://api.anthropic.com",
      "models": ["claude-3-5-sonnet-20241022"]
    }
  },
  "tools": {
    "enabled": ["file_read", "file_write", "shell_exec"],
    "timeout": 30000
  }
}
```

## Monitoring and Observability

### Health Checks

Configure health checks for deployment monitoring:

```bash
# Health endpoint
curl https://api.opencode.ai/health

# Expected response
{
  "status": "healthy",
  "version": "0.1.0",
  "uptime": 86400
}
```

### Logging

**Server Logging:**
- Structured JSON logs
- Multiple log levels (debug, info, warn, error)
- Request/response logging
- Error tracking with stack traces

**Client Logging:**
```bash
# TUI logs location
~/.local/share/opencode/logs/tui.log

# Enable debug logging
opencode --debug
```

### Metrics and Analytics

**Server Metrics:**
- Request counts and latency
- Error rates by endpoint
- AI provider response times
- Active session counts

**Client Metrics:**
- Usage analytics (opt-in)
- Performance metrics
- Error reporting

## Security Considerations

### API Security

- **HTTPS everywhere**: All production traffic over TLS
- **API key authentication**: Secure token-based auth
- **Rate limiting**: Prevent abuse and DoS attacks
- **Input validation**: All inputs validated with Zod schemas

### Client Security

- **Secure credential storage**: API keys encrypted at rest
- **Permission model**: File system access controls
- **Update mechanism**: Secure binary updates with signatures

### Infrastructure Security

- **Cloudflare protection**: DDoS protection and WAF
- **Secret management**: Environment variables and secure storage
- **Access controls**: Principle of least privilege

## Scaling Considerations

### Horizontal Scaling

The server is designed to be stateless and can scale horizontally:

```typescript
// Session storage can be moved to external store
interface SessionStore {
  get(id: string): Promise<Session>
  set(id: string, session: Session): Promise<void>
  delete(id: string): Promise<void>
}
```

### Performance Optimization

- **Connection pooling**: Efficient AI provider connections
- **Caching**: Response and computation caching
- **Streaming**: Real-time response streaming
- **CDN**: Static asset delivery via Cloudflare

### Resource Management

- **Memory limits**: Proper cleanup of sessions
- **Connection limits**: Pool AI provider connections
- **Rate limiting**: Protect against abuse
- **Timeout handling**: Prevent hanging requests

## Disaster Recovery

### Backup Strategy

- **Configuration backup**: User settings and configurations
- **Session persistence**: Optional session history backup
- **Infrastructure backup**: IaC in version control

### Rollback Procedures

```bash
# Rollback server deployment
sst deploy --stage production --target previous-version

# Rollback client distribution
# Use GitHub release rollback or package version pinning
npm install -g opencode-ai@0.0.4
```

### Monitoring and Alerting

- **Uptime monitoring**: External service monitoring
- **Error rate alerts**: Automated alerting on error spikes
- **Performance alerts**: Latency and throughput monitoring
- **Dependency monitoring**: AI provider status tracking

---

For more details on specific deployment scenarios, see:
- [SST Documentation](https://sst.dev/docs)
- [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)