# OpenCode - Technical Overview

This document provides a comprehensive overview of OpenCode's architecture, capabilities, and usage.

---

## What is OpenCode?

**OpenCode** is an open-source AI coding agent that provides an interactive AI assistant for software development. It helps developers with coding tasks, file editing, code analysis, and development workflows through multiple interfaces.

### Key Features

- **AI-Powered Development Assistant**: Uses AI models to help with coding tasks
- **Multiple Agent Modes**:
  - `build` - Full access agent for active development work
  - `plan` - Read-only agent for code exploration and analysis
  - `general` - Subagent for complex searches and multi-step tasks
- **Provider-Agnostic**: Works with multiple AI providers (Claude, OpenAI, Google, Azure, AWS Bedrock, and many more)
- **Built-in LSP Support**: Integrated Language Server Protocol support for code intelligence
- **MCP Integration**: Model Context Protocol support for extended capabilities
- **Multiple Interfaces**: Terminal UI, Desktop App, Web Interface, VS Code Extension

---

## Tech Stack

### Core Technologies

**Languages:**
- **TypeScript** - Main development language for the entire codebase
- **Rust** - Used for the Tauri desktop application backend
- **Bash** - Installation scripts and shell utilities

**Runtime & Build:**
- **Bun** (v1.3.5) - Primary JavaScript/TypeScript runtime and package manager
- **Node.js** - Alternative runtime support
- **Turbo** (Turborepo) - Monorepo build system
- **Vite** - Build tool and dev server

### Frontend

- **SolidJS** - Reactive UI framework for TUI and web interfaces
- **OpenTUI** (@opentui/core, @opentui/solid) - Terminal UI framework
- **Ghostty-web** - Terminal emulator component for browser
- **TailwindCSS** - Styling framework
- **Astro** - Static site generator for documentation/marketing website
- **Tauri v2** - Desktop application framework

### Backend & Infrastructure

- **Hono** - Web framework for HTTP server/API
- **SST (Serverless Stack)** - Infrastructure as code
- **Cloudflare Workers** - Serverless deployment platform
- **Zod** - Schema validation and type safety

### AI & Language Model Integration

- **Vercel AI SDK** - Core AI/LLM integration framework
- **Multiple AI Provider SDKs**:
  - @ai-sdk/anthropic (Claude)
  - @ai-sdk/openai
  - @ai-sdk/google, @ai-sdk/google-vertex
  - @ai-sdk/azure
  - @ai-sdk/amazon-bedrock
  - @ai-sdk/cohere, @ai-sdk/cerebras, @ai-sdk/deepinfra
  - @ai-sdk/groq, @ai-sdk/mistral, @ai-sdk/perplexity
  - @ai-sdk/togetherai, @ai-sdk/xai
  - @openrouter/ai-sdk-provider

### Development Tools

- **Language Server Protocol**: VSCode Language Server types and JSON-RPC
- **Tree-sitter**: Code parsing (tree-sitter-bash, web-tree-sitter)
- **Parcel Watcher**: File system watching
- **Model Context Protocol SDK** (@modelcontextprotocol/sdk)
- **Agent Client Protocol SDK** (@agentclientprotocol/sdk)

---

## Project Structure

```
/home/user/opencode/
├── packages/
│   ├── opencode/        # Core OpenCode CLI and server
│   │   ├── src/
│   │   │   ├── agent/   # AI agent logic
│   │   │   ├── cli/     # CLI commands and TUI
│   │   │   ├── provider/# AI provider integrations
│   │   │   ├── tool/    # Tool implementations
│   │   │   ├── lsp/     # Language Server Protocol integration
│   │   │   ├── mcp/     # Model Context Protocol integration
│   │   │   ├── server/  # HTTP/WebSocket server
│   │   │   └── session/ # Session management
│   │   └── bin/         # Binary wrapper
│   ├── app/             # Shared frontend application (browser/desktop)
│   ├── desktop/         # Tauri desktop application
│   ├── web/             # Documentation/marketing website
│   ├── ui/              # Shared UI components
│   └── sdk/             # JavaScript SDK
├── sdks/vscode/         # VS Code extension
├── install              # Installation script (Bash)
└── infra/               # Infrastructure configuration (SST)
```

---

## Main Entry Points

### 1. CLI Entry Point
**Location**: `packages/opencode/src/index.ts`
- Main CLI application entry point
- Uses Yargs for command parsing
- Defines all available commands: `run`, `serve`, `web`, `attach`, `auth`, `agent`, `mcp`, `github`, `pr`, `session`, etc.

### 2. Binary Wrapper
**Location**: `packages/opencode/bin/opencode`
- Node.js wrapper script that locates and executes the platform-specific binary
- Supports darwin (macOS), linux, and windows
- Handles architecture detection (x64, arm64)

### 3. Run Command
**Location**: `packages/opencode/src/cli/cmd/run.ts`
- Main interactive mode entry point
- Handles message input, file attachments, session management
- Supports both default (formatted) and JSON output formats

### 4. TUI (Terminal UI)
**Location**: `packages/opencode/src/cli/cmd/tui/`
- SolidJS-based terminal interface
- Components include threads, workers, spawning, and event handling

### 5. Server
**Location**: `packages/opencode/src/server/server.ts`
- Hono-based HTTP/WebSocket server
- Provides REST API and SSE endpoints
- Routes for sessions, agents, providers, project management

### 6. Desktop Application
**Entry**: `packages/desktop/`
- Tauri-based desktop app
- Frontend uses the shared `@opencode-ai/app` package
- Rust backend in `src-tauri/`

### 7. Web Application
**Entry**: `packages/web/`
- Astro-based documentation and marketing site
- SolidJS components

### 8. VS Code Extension
**Entry**: `sdks/vscode/`
- Provides OpenCode integration within VS Code
- Keyboard shortcuts: Cmd+Escape (Mac) / Ctrl+Escape (Windows/Linux)

---

## Artifact Generation Support

**OpenCode does NOT support Claude-style artifact generation** (separate rendered outputs like HTML pages, React components, Mermaid diagrams, etc.).

### Message Part Types

OpenCode's content is organized into these part types:

- `text` - Text responses
- `reasoning` - Extended thinking content (from models with reasoning capabilities)
- `file` - File attachments
- `tool` - Tool call executions (bash, edit, read, write, etc.)
- `step-start/step-finish` - Multi-step execution boundaries
- `patch` - File change patches
- `agent` - Agent/subagent information
- `retry` - Retry attempts
- `compaction` - Message compaction events
- `subtask` - Subtask delegation

### What OpenCode DOES Support

1. **Extended Thinking/Reasoning**: Models with reasoning capabilities can output thinking content that's displayed separately from the final response
2. **File Diffs**: Changes to files are shown as inline diffs in the response summary
3. **Tool Outputs**: Tool executions are shown as collapsible steps

The focus is on **direct code manipulation** rather than generating standalone artifacts for preview.

---

## File Creation & Manipulation

### Yes, OpenCode Can Create New Files

OpenCode has full file manipulation capabilities through dedicated tools.

### Available Tools

#### 1. Write Tool
**Location**: `packages/opencode/src/tool/write.ts:17`

Creates new files or overwrites existing ones:

```typescript
await WriteTool.execute({
  filePath: "/absolute/path/to/file.ts",
  content: "file contents here"
})
```

**Features:**
- Creates files directly on disk
- Requests permission before writing
- Integrates with LSP for diagnostics
- Checks for errors after file creation

**Guidelines:**
- Prefers editing existing files over creating new ones
- Won't proactively create documentation files unless requested
- Requires absolute file paths (or converts relative paths)

#### 2. Edit Tool
**Location**: `packages/opencode/src/tool/edit.ts:25`

Edits existing files with string replacement:

```typescript
await EditTool.execute({
  filePath: "/path/to/file.ts",
  oldString: "old code",
  newString: "new code",
  replaceAll: false  // optional
})
```

#### 3. Other File Tools

- **`multiedit`** - Make multiple edits in one operation
- **`bash`** - Run shell commands (can also create files)
- **`read`** - Read file contents
- **`glob`** - Find files by pattern
- **`grep`** - Search file contents

### IDE Integration

#### VSCode Extension
**Location**: `sdks/vscode/src/extension.ts`

The VSCode extension is a **simple terminal wrapper** that:
- Opens OpenCode CLI in a split terminal (Cmd+Esc / Ctrl+Esc)
- Sends file references from your current selection
- Doesn't directly manipulate VSCode's file system

**File Creation Workflow:**

1. Files are written **directly to disk** via filesystem operations
2. **VSCode automatically detects** the new files (filesystem watching)
3. Files appear in the VSCode explorer immediately
4. LSP integration provides diagnostics for the new files

**Example:**
```
User: "Create a new React component in src/components/Button.tsx"
→ OpenCode uses Write tool to create file on disk
→ VSCode's file watcher detects the new file
→ File appears in VSCode's explorer
→ LSP checks for TypeScript errors
→ User can immediately edit it in VSCode
```

---

## Browser & Web Support

### Yes! OpenCode Supports Working in a Browser

OpenCode provides multiple ways to access it through a web browser.

### 1. `opencode web` Command (Primary Browser Interface)

**Location**: `packages/opencode/src/cli/cmd/web.ts:30`

Starts a headless server with a full web interface:

```bash
opencode web
```

**What it does:**
- Starts an HTTP server (default port: 4096)
- Automatically opens your browser to the web UI
- Provides a full SolidJS-based application
- Includes a terminal emulator (ghostty-web) right in the browser

**Configuration:**
```bash
opencode web --port 4096 --hostname 0.0.0.0 --cors https://example.com
```

**Options:**
- `--port` - Server port (default: 4096)
- `--hostname` - Bind address (use `0.0.0.0` for network access)
- `--cors` - Additional origins for CORS
- `--mdns` - Enable mDNS discovery (accessible via `opencode.local`)

**Network Access:**
When using `--hostname 0.0.0.0`, the command displays:
- Local access URL: `http://localhost:4096`
- Network access URLs: `http://<network-ip>:4096`
- mDNS URL (if enabled): `http://opencode.local:4096`

### 2. `opencode serve` Command (Headless API Server)

**Location**: `packages/opencode/src/server/server.ts`

For programmatic access or custom web clients:

```bash
opencode serve --hostname 0.0.0.0 --port 4096 --cors http://localhost:5173
```

**Provides:**
- Full REST API with OpenAPI 3.1 spec
- Server-sent events (SSE) for real-time updates
- WebSocket support
- API documentation accessible at `/doc`

### 3. Web Application Architecture

**Location**: `packages/app/src/`

The browser interface includes:
- **SolidJS** frontend framework
- **Ghostty-web** terminal emulator embedded in browser
- Full session management UI
- File diff viewer with syntax highlighting
- Real-time message streaming via SSE/WebSocket
- Permission request handling
- Model and provider selection
- MCP server management

**Key Components:**
- `packages/app/src/pages/session.tsx` - Main session interface
- `packages/app/src/components/terminal.tsx` - Browser terminal
- `packages/app/src/components/prompt-input.tsx` - Message input
- `packages/ui/src/components/session-turn.tsx` - Message display

### 4. REST API Endpoints

**Full API documentation**: `packages/web/src/content/docs/server.mdx`

Key endpoints include:

**Sessions:**
- `GET /session` - List all sessions
- `POST /session` - Create a new session
- `POST /session/:id/message` - Send a message

**Files:**
- `GET /file/content?path=<path>` - Read a file
- `GET /find?pattern=<pattern>` - Search for text in files
- `GET /find/file?query=<query>` - Find files by name

**Project:**
- `GET /project/current` - Get current project info
- `GET /config` - Get configuration

**Events:**
- `GET /event` - Server-sent events stream for real-time updates

### 5. Use Cases

#### Remote Development
```bash
# On your server
opencode web --hostname 0.0.0.0 --port 4096

# Access from anywhere
http://your-server-ip:4096
```

#### Team Collaboration
```bash
# Start server with CORS for your domain
opencode serve --cors https://team.example.com
```

#### Custom Integrations
Use the REST API to build custom browser-based tools on top of OpenCode.

### 6. Available Interfaces Summary

OpenCode provides **5 different interfaces**:

1. **Terminal (TUI)** - Native terminal interface
2. **Desktop** - Tauri-based native app (`packages/desktop/`)
3. **Web** - Browser-based interface (`packages/app/`)
4. **VSCode** - Extension for VSCode (`sdks/vscode/`)
5. **Server** - Headless API server

All interfaces connect to the same OpenCode server backend, so you can:
- Mix and match interfaces
- Switch between them seamlessly
- Use multiple interfaces simultaneously
- Build custom clients using the API

---

## Available Tools

OpenCode provides the following built-in tools:

- `task` - Delegate work to subagents
- `write` - Create or overwrite files
- `edit` - Edit existing files with string replacement
- `multiedit` - Make multiple edits at once
- `read` - Read file contents
- `bash` - Execute shell commands
- `glob` - Find files by pattern
- `grep` - Search file contents
- `list` - List directory contents
- `websearch` - Search the web
- `webfetch` - Fetch web content
- `todowrite` - Manage todo lists
- `todoread` - Read todo lists
- `lsp` - Language Server Protocol operations
- `codesearch` - Semantic code search
- `skill` - Execute skills
- `batch` - Batch operations
- `patch` - Apply patches

---

## Extended Thinking & Reasoning Support

OpenCode supports extended thinking/reasoning modes through the provider transform system.

### Supported Providers

**Provider Transform Tests**: `packages/opencode/test/provider/transform.test.ts`

Models with reasoning capabilities include:

#### Anthropic
- Claude Sonnet 4+
- Uses `thinking` configuration with `budgetTokens`
- Supports "high" and "max" variants

#### OpenAI
- o1, o1-mini, o1-pro
- GPT-4o and newer models
- Uses `reasoningEffort` levels: none, minimal, low, medium, high, xhigh

#### Google
- Gemini 2.0+ models
- Uses `thinkingLevel` or `thinkingConfig`
- Supports `includeThoughts` option

#### DeepSeek
- DeepSeek Chat
- Uses interleaved `reasoning_content` field

#### Other Providers
- AWS Bedrock - `reasoningConfig`
- Azure - `reasoningEffort` with `reasoningSummary`
- Cerebras, TogetherAI, xAI, DeepInfra - `reasoningEffort`
- Groq - `thinkingLevel`

### Reasoning Display

Reasoning content is displayed separately from final responses in the UI and can be toggled on/off.

---

## Development & Build

### Running OpenCode

```bash
# Dev mode
bun dev

# Build standalone binary
./packages/opencode/script/build.ts --single

# Type checking
bun turbo typecheck

# Run tests
bun test
```

### Deployment

OpenCode can be deployed to:
- **Cloudflare Workers** (using SST framework)
- **Self-hosted servers** (via `opencode serve` or `opencode web`)
- **Desktop** (Tauri packaged apps)
- **Package managers** (npm, brew, chocolatey, scoop, etc.)

---

## Configuration

OpenCode can be configured via:
- `opencode.json` - Project-level configuration
- Environment variables
- CLI flags
- Interactive configuration commands (`/connect`, `/auth`)

See `packages/web/src/content/docs/config.mdx` for full configuration options.

---

## Enterprise Features

- **SSO Integration** - Via central config
- **Internal AI Gateway** - Route all requests through your gateway
- **Private NPM Registry** - Support for JFrog Artifactory, Nexus, etc.
- **Data Security** - Code never stored by OpenCode (except optional `/share` feature)

See `packages/web/src/content/docs/enterprise.mdx` for details.

---

## Resources

- **Website**: https://opencode.ai
- **GitHub**: https://github.com/sst/opencode
- **Documentation**: https://opencode.ai/docs
- **OpenAPI Spec**: `http://localhost:4096/doc` (when server is running)
- **Issues**: https://github.com/sst/opencode/issues

---

## License

MIT License - See repository for full license text.

---

*This document was generated based on analysis of the OpenCode codebase as of January 2026.*
