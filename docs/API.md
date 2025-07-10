# API Architecture

This document details the API architecture and communication patterns between OpenCode components.

## Server API Overview

The OpenCode server provides a RESTful API built with Hono and documented with OpenAPI specifications. The API follows REST principles with additional streaming support for real-time interactions.

### Base Architecture

```
HTTP Client (TUI) ←→ Hono API Server ←→ AI Providers
                            ↓
                      Tool Execution
                            ↓
                      File Operations
                            ↓
                     LSP Integration
```

## Core Endpoints

### Session Management

#### Create Session
```http
POST /sessions
Content-Type: application/json

{
  "config": {
    "provider": "anthropic",
    "model": "claude-3-5-sonnet-20241022"
  }
}
```

**Response:**
```http
201 Created
{
  "id": "session-uuid",
  "config": { ... },
  "created": "2025-07-10T06:12:00Z"
}
```

#### Execute Request
```http
POST /sessions/{sessionId}/run
Content-Type: application/json

{
  "messages": [
    {
      "role": "user", 
      "content": "Create a React component"
    }
  ],
  "tools": ["file_write", "file_read"]
}
```

**Response:** Server-Sent Events stream with tool executions and AI responses.

### Real-time Communication

The API uses Server-Sent Events (SSE) for streaming responses:

```http
GET /sessions/{sessionId}/events
Accept: text/event-stream
```

**Event Types:**
- `tool_start` - Tool execution begins
- `tool_result` - Tool execution completes  
- `message_start` - AI response begins
- `message_chunk` - AI response chunk
- `message_end` - AI response complete
- `session_end` - Session terminated

### Tool System

Tools are the primary mechanism for code operations:

```typescript
interface Tool {
  name: string
  description: string
  parameters: ZodSchema
  execute(context: ToolContext): Promise<ToolResult>
}
```

**Built-in Tools:**
- `file_read` - Read file contents
- `file_write` - Write file contents
- `file_search` - Search within files
- `shell_exec` - Execute shell commands
- `lsp_hover` - Get symbol information
- `lsp_definition` - Go to definition

### Authentication

Authentication uses OpenAuth with support for multiple providers:

```http
POST /auth/authorize
{
  "provider": "github",
  "redirect_uri": "http://localhost:3000/callback"
}
```

## Client SDK Architecture

The Go client uses a generated SDK from OpenAPI specifications:

### SDK Generation

```bash
# Regenerate SDK when server API changes
./scripts/stainless
```

### Client Usage

```go
client := opencode.NewClient(
    option.WithBaseURL("http://localhost:3000"),
    option.WithAPIKey("user-token"),
)

session, err := client.Sessions.New(ctx, opencode.SessionNewParams{
    Config: opencode.SessionConfig{
        Provider: "anthropic",
        Model:    "claude-3-5-sonnet",
    },
})
```

## Error Handling

### Error Response Format

```json
{
  "error": {
    "type": "validation_error",
    "message": "Invalid request parameters",
    "details": {
      "field": "model",
      "reason": "Model not supported"
    }
  }
}
```

### Error Types

- `validation_error` - Invalid request parameters
- `auth_error` - Authentication/authorization failure
- `provider_error` - AI provider communication error
- `tool_error` - Tool execution failure
- `session_error` - Session management error

## Rate Limiting

The API implements rate limiting to prevent abuse:

- **Per-session**: 100 requests per minute
- **Per-user**: 1000 requests per hour
- **Tool execution**: 50 concurrent executions

Headers:
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1625097600
```

## API Versioning

The API uses URL path versioning:

```
/v1/sessions     # Current stable API
/v2/sessions     # Next version (when available)
```

Version compatibility:
- **v1**: Current stable version
- Breaking changes require new version
- Deprecation notices provided 6 months in advance

## WebSocket Support (Future)

While currently using SSE, WebSocket support is planned for bidirectional communication:

```javascript
// Future WebSocket API
const ws = new WebSocket('ws://localhost:3000/sessions/{id}/ws')
ws.onmessage = (event) => {
  const data = JSON.parse(event.data)
  // Handle real-time updates
}
```

## Monitoring and Observability

### Health Endpoints

```http
GET /health
{
  "status": "healthy",
  "version": "0.1.0",
  "uptime": 86400,
  "dependencies": {
    "database": "healthy",
    "ai_providers": "healthy"
  }
}
```

### Metrics

Key metrics tracked:
- Request/response times
- Error rates by endpoint
- AI provider latency
- Tool execution times
- Active session count

### Logging

Structured logging with context:
```json
{
  "level": "info",
  "service": "api",
  "session_id": "sess_123",
  "tool": "file_write",
  "duration_ms": 150,
  "message": "Tool execution completed"
}
```