# OpenCode Architecture

OpenCode is an AI coding agent built for the terminal with a modern client/server architecture that prioritizes flexibility, performance, and extensibility.

## System Overview

OpenCode follows a **client/server architecture** that separates the AI logic and data processing (server) from the user interface (client). This design enables multiple client types, remote operation, and better separation of concerns.

### High-Level Architecture

```
                             OpenCode System Architecture
    
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                   User Layer                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │   TUI Client    │    │   CLI Commands  │    │  Future: Web    │             │
│  │   (Go/Bubble)   │    │  (TypeScript)   │    │     Client      │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│           │                       │                       │                     │
└───────────┼───────────────────────┼───────────────────────┼─────────────────────┘
            │                       │                       │
            │              HTTP/REST API                    │
            │                       │                       │
┌───────────┼───────────────────────┼───────────────────────┼─────────────────────┐
│           │                    Core Server                │                     │
│           │                  (TypeScript)                 │                     │
├───────────┼───────────────────────┼───────────────────────┼─────────────────────┤
│           ▼                       ▼                       ▼                     │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │  Session Mgmt   │    │   API Server    │    │  Authentication │             │
│  │   & State       │    │     (Hono)      │    │   (OpenAuth)    │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│           │                       │                       │                     │
│           ▼                       ▼                       ▼                     │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │   Tool System   │    │  File Operations│    │  LSP Integration│             │
│  │   Framework     │    │   & Storage     │    │   & Code Intel  │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│           │                       │                       │                     │
└───────────┼───────────────────────┼───────────────────────┼─────────────────────┘
            │                       │                       │
            │                   Provider Layer              │
            │                       │                       │
┌───────────┼───────────────────────┼───────────────────────┼─────────────────────┐
│           ▼                       ▼                       ▼                     │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │    Anthropic    │    │     OpenAI      │    │     Google      │             │
│  │   Claude 3.5    │    │    GPT-4o       │    │    Gemini       │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│           │                       │                       │                     │
│           ▼                       ▼                       ▼                     │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │  Local Models   │    │   Custom APIs   │    │  Future: More   │             │
│  │   (Ollama)      │    │   & Endpoints   │    │   Providers     │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Communication Flow

```
TUI Client                 Core Server               AI Provider
    │                          │                         │
    ├─ POST /sessions         ─┤                         │
    │◄─ 201 {sessionId}       ─┤                         │
    │                          │                         │
    ├─ POST /sessions/{id}/run ┤                         │
    │                          ├─ Tool execution        ─│
    │                          │◄─ Tool results         ─┤
    │                          │                         │
    │                          ├─ AI API call          ─┤
    │                          │◄─ Streaming response   ─┤
    │◄─ SSE stream             ┤                         │
```

## Design Principles

1. **Provider Agnostic**: Support multiple AI providers without vendor lock-in
2. **Terminal First**: Rich TUI experience optimized for developer workflows  
3. **Client/Server**: Flexible architecture enabling remote operation and multiple UIs
4. **Developer Experience**: Seamless integration with existing developer tools
5. **Open Source**: 100% open source with transparent development

## Core Components

### 1. Core Server (`packages/opencode`)

**Technology**: TypeScript with Bun runtime  
**Purpose**: Central orchestration, AI provider management, and API services

#### Key Modules:

- **CLI Interface** (`src/cli/`): Command-line entry point and argument parsing
- **API Server** (`src/server/`): Hono-based REST API with OpenAPI specifications
- **Session Management** (`src/session/`): User session and conversation state
- **Provider Abstraction** (`src/provider/`): AI provider integration layer
- **Tool System** (`src/tool/`): Extensible tool framework for code operations
- **LSP Integration** (`src/lsp/`): Language Server Protocol support
- **File Operations** (`src/file/`): File system operations and search
- **Configuration** (`src/config/`): Schema-based configuration management
- **Authentication** (`src/auth/`): User authentication and authorization

#### Key Features:

- **Multi-provider AI support**: Anthropic, OpenAI, Google, local models
- **RESTful API**: OpenAPI-documented endpoints for client communication
- **Tool ecosystem**: Extensible tools for code generation, editing, and analysis
- **Session persistence**: Conversation and context management
- **LSP integration**: Code intelligence and language-specific features

### 2. TUI Client (`packages/tui`)

**Technology**: Go with Charm libraries (Bubble Tea, Lipgloss, Bubbles)  
**Purpose**: Rich terminal user interface for interactive coding sessions

#### Key Modules:

- **Main Application** (`cmd/opencode/`): Entry point and application setup
- **TUI Framework** (`internal/tui/`): Bubble Tea application and state management
- **Components** (`internal/components/`): Reusable UI components
- **SDK Client** (`sdk/`): Generated OpenAPI client for server communication
- **Theming** (`internal/theme/`): Customizable appearance system
- **Utilities** (`internal/util/`): Helper functions and shared logic

#### Key Features:

- **Rich terminal UI**: Interactive file trees, code viewers, chat interface
- **Real-time updates**: Live streaming of AI responses and code changes
- **Customizable themes**: JSON-based theming with user overrides
- **Keyboard shortcuts**: Efficient navigation and command execution
- **Multi-pane layout**: Side-by-side code and conversation views

### 3. Web Documentation (`packages/web`)

**Technology**: Astro with Solid.js  
**Purpose**: Documentation website and API references

- **Static site generation**: Fast, SEO-friendly documentation
- **Interactive examples**: Code samples and usage demonstrations
- **API documentation**: Auto-generated from OpenAPI specs

### 4. Cloud Functions (`packages/function`)

**Technology**: Cloudflare Workers  
**Purpose**: Serverless deployment and edge computing

## Data Flow and Communication

### Client ↔ Server Communication

1. **TUI Initialization**: Go client connects to TypeScript server via HTTP API
2. **Session Creation**: Server creates session context and returns session ID
3. **Message Exchange**: Streaming communication for real-time AI responses
4. **Tool Execution**: Server executes tools and returns results to client
5. **State Synchronization**: Client and server maintain synchronized state

```
TUI Client                 Core Server               AI Provider
    │                          │                         │
    ├─ POST /sessions         ─┤                         │
    │◄─ 201 {sessionId}       ─┤                         │
    │                          │                         │
    ├─ POST /sessions/{id}/run ┤                         │
    │                          ├─ Tool execution        ─│
    │                          │◄─ Tool results         ─┤
    │                          │                         │
    │                          ├─ AI API call          ─┤
    │                          │◄─ Streaming response   ─┤
    │◄─ SSE stream             ┤                         │
```

### Key API Endpoints

- `POST /sessions` - Create new session
- `POST /sessions/{id}/run` - Execute AI request with tools
- `GET /sessions/{id}/events` - Stream session events
- `GET /models` - List available AI models
- `POST /auth/*` - Authentication endpoints

## Technology Stack

### Core Technologies

| Component | Technology | Rationale |
|-----------|------------|-----------|
| **Server Runtime** | Bun | Fast TypeScript execution, excellent DX |
| **TUI Framework** | Bubble Tea | Rich terminal UIs, Go ecosystem |
| **API Framework** | Hono | Lightweight, fast, OpenAPI support |
| **AI Integration** | Vercel AI SDK | Provider abstraction, streaming support |
| **Documentation** | Astro + Solid.js | Static generation with interactive components |
| **Deployment** | SST + Cloudflare | Serverless, edge deployment |

### Development Tools

- **Package Manager**: Bun (primary), npm fallback
- **Type Checking**: TypeScript with strict configuration
- **API Generation**: Stainless SDK generation
- **Validation**: Zod schemas throughout
- **Testing**: Bun test, Go testing framework

## Directory Structure

```
opencode/
├── packages/
│   ├── opencode/           # Core TypeScript server
│   │   ├── src/
│   │   │   ├── cli/        # Command-line interface
│   │   │   ├── server/     # HTTP API server
│   │   │   ├── session/    # Session management
│   │   │   ├── provider/   # AI provider integrations
│   │   │   ├── tool/       # Tool framework
│   │   │   ├── lsp/        # Language Server Protocol
│   │   │   ├── file/       # File operations
│   │   │   └── config/     # Configuration management
│   │   └── test/           # Test suites
│   │
│   ├── tui/                # Go terminal client
│   │   ├── cmd/opencode/   # Main application
│   │   ├── internal/       # Internal packages
│   │   │   ├── tui/        # TUI application logic
│   │   │   ├── components/ # UI components
│   │   │   └── theme/      # Theming system
│   │   └── sdk/            # Generated API client
│   │
│   ├── web/                # Documentation website
│   │   ├── src/
│   │   │   ├── content/    # Documentation content
│   │   │   └── components/ # UI components
│   │   └── public/         # Static assets
│   │
│   └── function/           # Cloudflare functions
│
├── infra/                  # Infrastructure configuration
├── scripts/                # Build and utility scripts
└── docs/                   # Additional documentation
```

## Configuration System

OpenCode uses a layered configuration approach:

1. **Default Configuration**: Built-in defaults for all settings
2. **Global Configuration**: User-wide settings in `~/.config/opencode/`
3. **Project Configuration**: Project-specific `.opencode/` directory
4. **Environment Variables**: Runtime overrides
5. **Command Arguments**: Session-specific overrides

Configuration is validated using Zod schemas with automatic type generation.

## Authentication and Security

- **OpenAuth Integration**: Secure authentication flow
- **Provider API Keys**: Encrypted storage of AI provider credentials
- **Session Security**: Secure session tokens and state isolation
- **File Permissions**: Respect file system permissions and user access

## Deployment Architecture

OpenCode supports multiple deployment scenarios:

### Local Development
```bash
bun run packages/opencode/src/index.ts  # Start server
go run ./cmd/opencode                   # Start TUI client
```

### Production Deployment
- **Server**: Deployed via SST to Cloudflare Workers
- **Client**: Distributed as compiled binaries via GitHub Releases
- **Documentation**: Static site hosted on Cloudflare Pages

## Extension Points

OpenCode is designed for extensibility:

1. **AI Providers**: Add new providers via the provider interface
2. **Tools**: Extend functionality with custom tools
3. **TUI Components**: Create new UI components for the terminal
4. **Themes**: Customize appearance with JSON theme files
5. **Commands**: Add new CLI commands and workflows

## Performance Considerations

- **Streaming Responses**: Real-time AI response streaming
- **Efficient Rendering**: Optimized terminal rendering with Bubble Tea
- **Caching**: Intelligent caching of AI responses and file operations
- **Lazy Loading**: On-demand loading of language servers and tools
- **Resource Management**: Proper cleanup of sessions and connections

## Development Workflow

1. **Setup**: Clone repository and install dependencies
2. **Development**: Use watch mode for iterative development
3. **Testing**: Run unit and integration tests
4. **API Changes**: Regenerate SDK when server APIs change
5. **Building**: Cross-platform builds via GitHub Actions
6. **Release**: Automated releases with semantic versioning

---

For more detailed information about specific components, see:
- **[API Architecture](docs/API.md)** - Detailed API design and communication patterns
- **[Deployment Guide](docs/DEPLOYMENT.md)** - Production deployment and infrastructure
- **[Server Development](packages/opencode/README.md)** - Server-side development guide
- **[TUI Development](packages/tui/AGENTS.md)** - TUI client development guide
- **[Contributing Guidelines](CONTRIBUTING.md)** - How to contribute to the project