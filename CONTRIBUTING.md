# Contributing to OpenCode

Thank you for your interest in contributing to OpenCode! This guide will help you get started with development and understand our contribution process.

## Development Setup

### Prerequisites

- **Bun** (recommended) or **Node.js 18+**
- **Go 1.24+**
- **Git**

### Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/sst/opencode.git
   cd opencode
   ```

2. **Install dependencies**:
   ```bash
   bun install  # or npm install
   ```

3. **Run the development server**:
   ```bash
   bun run packages/opencode/src/index.ts
   ```

4. **In another terminal, run the TUI client**:
   ```bash
   cd packages/tui
   go run ./cmd/opencode
   ```

## Project Structure

See [ARCHITECTURE.md](ARCHITECTURE.md) for a comprehensive overview of the system architecture and component organization.

## Development Guidelines

For detailed development workflows, debugging techniques, and common tasks, see **[Development Workflows](docs/DEVELOPMENT.md)**.

### Code Style

**TypeScript (Server)**:
- Use ESM modules with relative imports
- Prefer `const` over `let`, avoid `var`
- Use Zod schemas for validation
- Follow namespace-based organization
- Avoid unnecessary destructuring and `else` statements

**Go (TUI Client)**:
- Follow standard Go formatting (`gofmt`)
- Use table-driven tests
- Group imports: standard, third-party, local
- Return errors explicitly

### Testing

**TypeScript**:
```bash
bun test                                    # Run all tests
bun test test/tool/tool.test.ts            # Run specific test
```

**Go**:
```bash
go test ./...                               # Run all tests
go test ./internal/theme -run TestSpecific  # Run specific test
```

## Making Changes

### Before Starting

- Check existing issues or create a new one to discuss your feature/fix
- We're responsive on GitHub issues and appreciate early discussion
- For simple fixes, feel free to submit a PR directly

### API Changes

If you modify the TypeScript server API endpoints in `packages/opencode/src/server/server.ts`:

1. The OpenCode team needs to generate a new Stainless SDK
2. Open an issue requesting SDK regeneration before proceeding with client-side changes
3. Wait for the updated SDK before implementing TUI client changes

### Pull Request Process

1. **Fork and branch**: Create a feature branch from `main`
2. **Make changes**: Follow the coding guidelines above
3. **Test**: Ensure all tests pass and add tests for new features
4. **Commit**: Use clear, descriptive commit messages
5. **Pull Request**: Submit a PR with a clear description of changes

### Commit Messages

Use conventional commit format:
```
feat: add new AI provider integration
fix: resolve session timeout issue
docs: update API documentation
test: add unit tests for tool execution
```

## Development Commands

### TypeScript Server
```bash
# Development
bun run packages/opencode/src/index.ts    # Run development server
bun run typecheck                         # Type checking
bun test                                  # Run tests

# Building
bun run build                             # Build for production
```

### Go TUI Client
```bash
# Development
go run ./cmd/opencode                     # Run TUI client
go build ./cmd/opencode                   # Build binary

# Testing
go test ./...                             # Run all tests
go test -race ./...                       # Run with race detection
```

### Documentation Website
```bash
cd packages/web
npm run dev                               # Development server
npm run build                             # Build static site
```

## Architecture Guidelines

### Adding New Features

1. **AI Providers**: Implement the `Provider` interface in `src/provider/`
2. **Tools**: Extend the tool framework in `src/tool/`
3. **TUI Components**: Create reusable components in `internal/components/`
4. **CLI Commands**: Add commands in `src/cli/cmd/`

### Configuration

- Use Zod schemas for validation
- Add new config options to the main schema
- Document configuration options
- Provide sensible defaults

### Error Handling

- Use typed errors and Result patterns in TypeScript
- Return errors explicitly in Go
- Provide helpful error messages for users
- Log detailed error information for debugging

## Release Process

1. **Version Bump**: Update version in `package.json`
2. **Changelog**: Update `CHANGELOG.md` with changes
3. **Tag Release**: Create a git tag following semantic versioning
4. **GitHub Release**: Automated builds create release artifacts
5. **Distribution**: Packages are published to npm, brew, etc.

## Getting Help

- **Issues**: GitHub issues for bugs and feature requests
- **Discussions**: GitHub discussions for questions and ideas
- **Community**: Join our [YouTube](https://www.youtube.com/c/sst-dev) and [X.com](https://x.com/SST_dev) community

## Code of Conduct

Please be respectful and constructive in all interactions. We're building this project together and want everyone to feel welcome to contribute.

---

Thank you for contributing to OpenCode! 🚀