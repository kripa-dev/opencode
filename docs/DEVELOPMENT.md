# Development Workflows

This document outlines common development workflows and best practices for working with the OpenCode codebase.

## Development Environment Setup

### Complete Setup Checklist

- [ ] **Prerequisites installed**: Bun, Go 1.24+, Git
- [ ] **Repository cloned** and dependencies installed
- [ ] **AI provider keys** configured in local config
- [ ] **Development server** running successfully
- [ ] **TUI client** connecting to local server
- [ ] **Tests passing** for both TypeScript and Go components

### Recommended Development Tools

**Code Editors:**
- **VS Code** with TypeScript/Go extensions
- **Neovim** with LSP support (dogfooding OpenCode itself!)
- **GoLand/WebStorm** for full IDE experience

**Terminal Tools:**
- **tmux/screen** for managing multiple sessions
- **ripgrep** for fast code search
- **fd** for fast file finding
- **bat** for syntax highlighting

## Common Development Tasks

### Working on the Server (TypeScript)

```bash
# Start development server with auto-reload
bun run packages/opencode/src/index.ts

# Run specific tests
bun test test/tool/tool.test.ts

# Type checking
bun run typecheck

# Add new dependency
cd packages/opencode
bun add package-name

# Run linting (if configured)
bun run lint
```

### Working on the TUI (Go)

```bash
cd packages/tui

# Build and run
go build ./cmd/opencode && ./opencode

# Run with debugging
go run -tags debug ./cmd/opencode

# Test specific package
go test ./internal/theme -v

# Run tests with race detection
go test -race ./...

# Update dependencies
go mod tidy && go mod download
```

### Working on Documentation (Astro)

```bash
cd packages/web

# Development server
npm run dev

# Build static site
npm run build

# Preview built site
npm run preview
```

## Feature Development Workflow

### 1. Planning Phase

- [ ] **Create/discuss issue** on GitHub
- [ ] **Define requirements** and acceptance criteria
- [ ] **Identify affected components** (server, TUI, docs)
- [ ] **Plan API changes** if needed

### 2. Implementation Phase

#### For Server-Side Features

1. **Define API contract** in `src/server/server.ts`
2. **Create Zod schemas** for validation
3. **Implement business logic** in appropriate modules
4. **Add unit tests** for new functionality
5. **Update OpenAPI documentation**

#### For Client-Side Features

1. **Wait for API changes** to be deployed (if needed)
2. **Update TUI components** in `internal/`
3. **Handle new API responses** and errors
4. **Add Go tests** for new functionality
5. **Test integration** with server

#### For Cross-Component Features

1. **Start with server-side** API definition
2. **Get SDK regenerated** by OpenCode team
3. **Implement client-side** consumption
4. **Test end-to-end** functionality

### 3. Testing Phase

```bash
# Run all tests
bun test                    # TypeScript tests
go test ./...              # Go tests

# Integration testing
# 1. Start server
bun run packages/opencode/src/index.ts

# 2. Test TUI against server
cd packages/tui && go run ./cmd/opencode

# 3. Test specific scenarios
# - AI provider integrations
# - Tool executions
# - File operations
# - Error handling
```

### 4. Documentation Phase

- [ ] **Update API documentation** if endpoints changed
- [ ] **Add code comments** for complex logic
- [ ] **Update user-facing docs** if behavior changed
- [ ] **Update CHANGELOG.md** with changes

## Debugging Workflows

### Server Debugging

```bash
# Enable debug logging
DEBUG=* bun run packages/opencode/src/index.ts

# Log specific modules
DEBUG=session,tool bun run packages/opencode/src/index.ts

# Check logs
tail -f ~/.local/share/opencode/logs/server.log
```

### TUI Debugging

```bash
# Build with debug info
go build -tags debug ./cmd/opencode

# View TUI logs
tail -f ~/.local/share/opencode/logs/tui.log

# Use Go debugging tools
dlv debug ./cmd/opencode
```

### API Debugging

```bash
# Test API endpoints directly
curl -X POST http://localhost:3000/sessions \
  -H "Content-Type: application/json" \
  -d '{"config":{"provider":"anthropic"}}'

# Monitor network traffic
# Use browser dev tools or tools like mitmproxy
```

## Performance Optimization

### Profiling Server Performance

```bash
# Memory profiling
node --inspect packages/opencode/src/index.ts

# CPU profiling with Bun
bun run --hot packages/opencode/src/index.ts
```

### Profiling TUI Performance

```bash
# Go performance profiling
go test -cpuprofile cpu.prof -memprofile mem.prof -bench .

# Analyze profiles
go tool pprof cpu.prof
go tool pprof mem.prof
```

### Load Testing

```bash
# Simple load test
ab -n 1000 -c 10 http://localhost:3000/health

# More complex scenarios with tools like wrk or k6
```

## Release Workflow

### Version Management

1. **Update version** in `package.json` and relevant files
2. **Update CHANGELOG.md** with changes
3. **Create version tag**: `git tag v0.1.0`
4. **Push tag**: `git push origin v0.1.0`

### Automated Release Process

GitHub Actions handles:
- **Cross-platform builds** for TUI client
- **Docker image creation** (if applicable)
- **npm package publishing**
- **GitHub release creation**
- **Distribution updates** (brew, AUR, etc.)

### Manual Release Steps

```bash
# Verify release artifacts
gh release view v0.1.0

# Test installation from release
curl -fsSL https://opencode.ai/install | bash
opencode --version

# Verify package managers
npm install -g opencode-ai@0.1.0
brew install sst/tap/opencode
```

## Troubleshooting Common Issues

### "API Client Out of Sync"

**Problem**: TUI client errors when communicating with server  
**Solution**: Regenerate SDK after server API changes

```bash
# Request SDK regeneration from OpenCode team
# Or temporarily use API directly while waiting
```

### "Tests Failing After Dependencies Update"

**Problem**: Tests break after dependency updates  
**Solution**: Check for breaking changes and update code

```bash
# Check what changed
bun run typecheck
go mod why -m package-name

# Update imports and usage as needed
```

### "TUI Rendering Issues"

**Problem**: Terminal display problems  
**Solution**: Check terminal compatibility

```bash
# Test with different terminals
# Check terminal size and capabilities
echo $TERM
stty size
```

### "Performance Degradation"

**Problem**: Slow response times  
**Solution**: Profile and identify bottlenecks

```bash
# Check server metrics
curl http://localhost:3000/health

# Profile critical paths
# Optimize database queries, API calls, etc.
```

## Best Practices

### Code Organization

- **Keep functions small** and focused
- **Use TypeScript types** extensively
- **Prefer composition** over inheritance
- **Write self-documenting code**

### Testing Strategy

- **Unit tests** for individual functions
- **Integration tests** for component interaction
- **End-to-end tests** for user workflows
- **Performance tests** for critical paths

### Git Workflow

- **Feature branches** for all changes
- **Descriptive commit messages**
- **Regular rebasing** to keep history clean
- **PR reviews** before merging

### Documentation

- **Code comments** for complex logic
- **API documentation** for all endpoints
- **User guides** for new features
- **Architecture docs** for system changes

---

For more specific guidance, see:
- [Contributing Guidelines](../CONTRIBUTING.md)
- [Architecture Overview](../ARCHITECTURE.md)
- [API Documentation](API.md)
- [Deployment Guide](DEPLOYMENT.md)