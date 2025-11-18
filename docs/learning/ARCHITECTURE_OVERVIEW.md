# Architecture Overview

**Understanding the Jan AI system from 10,000 feet**

**Documented:** November 18, 2025 | **For:** Understanding the complete system architecture

---

## Table of Contents

- [System Overview](#system-overview)
- [High-Level Architecture](#high-level-architecture)
- [Core Components](#core-components)
- [Data Flow](#data-flow)
- [Technology Stack Integration](#technology-stack-integration)
- [Extension System](#extension-system)
- [MCP Integration](#mcp-integration)
- [API Architecture](#api-architecture)
- [Next Steps](#next-steps)

---

## System Overview

### What is Jan AI?

Jan AI is an **open-source ChatGPT replacement** that runs AI models locally on your machine while also supporting cloud providers. Think of it as:

🧠 **Mental Model:**
```
Jan AI = Local AI Runtime + Cloud Integration + Extension Platform
```

**Key Characteristics:**
- **Privacy-first**: All data stays local (when using local models)
- **Multi-platform**: Desktop (Windows/macOS/Linux) + Mobile (iOS/Android)
- **Extensible**: Plugin architecture for new capabilities
- **API-compatible**: OpenAI-compatible API server
- **Model-agnostic**: Supports both local (llama.cpp) and cloud models

---

## High-Level Architecture

### The Big Picture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Jan AI Application                            │
│                         (Tauri Desktop App)                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌──────────────────────────┐          ┌────────────────────────┐  │
│  │    FRONTEND LAYER        │          │    BACKEND LAYER       │  │
│  │   (React 19 SPA)         │◄────IPC──►│   (Rust/Tauri 2)      │  │
│  │                          │          │                        │  │
│  │  • TanStack Router       │          │  • Tauri Commands      │  │
│  │  • Zustand Stores        │          │  • Event Emitters      │  │
│  │  • Service Layer         │          │  • Custom Plugins      │  │
│  │  • UI Components         │          │  • HTTP Server         │  │
│  └──────────────────────────┘          └────────────────────────┘  │
│           │                                       │                  │
│           │                                       │                  │
│  ┌────────▼─────────────┐              ┌─────────▼──────────────┐  │
│  │  EXTENSION SYSTEM    │              │   CORE FUNCTIONALITY   │  │
│  │                      │              │                        │  │
│  │  • Native Extensions │              │  • LLM Engines         │  │
│  │  • Web Extensions    │              │    - llama.cpp        │  │
│  │  • Provider System   │              │    - Cloud Providers  │  │
│  │  • MCP Servers       │              │  • Vector Database    │  │
│  └──────────────────────┘              │  • RAG System          │  │
│                                         │  • File System         │  │
│                                         │  • MCP Integration     │  │
│                                         └────────────────────────┘  │
│                                                                       │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    API SERVER (OpenAI-Compatible)             │  │
│  │                    localhost:1337                             │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
            ┌───────▼────────┐       ┌───────▼────────┐
            │  Local Models  │       │ Cloud Providers│
            │  (llama.cpp)   │       │ (OpenAI, etc.) │
            └────────────────┘       └────────────────┘
```

🌉 **Bridge from Web Apps:**
Traditional web app = React + Node.js Backend
Jan AI = React + Rust Backend (via Tauri)
Communication = HTTP/WebSocket → Tauri IPC (Inter-Process Communication)

---

## Core Components

### 1. Frontend Layer (React SPA)

**Location**: [`web-app/src/`](../../web-app/src/)

```
web-app/src/
├── routes/          # TanStack Router pages
├── services/        # Business logic layer
├── containers/      # Complex UI components
├── components/ui/   # Reusable UI primitives
├── hooks/           # Custom React hooks
└── lib/             # Utilities
```

**Key Technologies:**
- ✅ **React 19** - UI framework with Actions API
- ✅ **TanStack Router** - Type-safe file-based routing
- ✅ **Zustand** - Lightweight state management
- ✅ **TailwindCSS 4** - Utility-first styling
- ✅ **Radix UI** - Accessible component primitives

**Architecture Pattern**: Service Layer + Container/Presentational Components

```typescript
// Example flow:
User clicks button
  → Component calls service method
    → Service calls Tauri command (or web API)
      → Tauri command executes Rust code
        → Result returned to service
          → Service updates Zustand store
            → Component re-renders
```

💡 **Aha Moment:** The frontend doesn't directly call Tauri. It goes through the **service layer**, which provides different implementations for Tauri, Web, and Default (fallback).

**Service Pattern:**

```typescript
// web-app/src/services/app/types.ts
export interface AppService {
  getConfigurations(): Promise<Config>
  updateConfiguration(config: Config): Promise<void>
}

// web-app/src/services/app/tauri.ts
export const tauriAppService: AppService = {
  async getConfigurations() {
    return await invoke('get_app_configurations')
  }
}

// web-app/src/services/app/web.ts
export const webAppService: AppService = {
  async getConfigurations() {
    return await fetch('/api/config').then(r => r.json())
  }
}
```

🎯 **Remember This:** "Service Layer = Abstraction over Tauri/Web". One interface, multiple implementations.

---

### 2. Backend Layer (Rust/Tauri)

**Location**: [`src-tauri/src/`](../../src-tauri/src/)

```
src-tauri/src/
├── lib.rs            # Tauri app entry point
├── main.rs           # Main executable
└── core/             # Core backend modules
    ├── app/          # App configuration
    ├── downloads/    # Model downloads
    ├── extensions/   # Extension management
    ├── filesystem/   # File operations
    ├── mcp/          # Model Context Protocol
    ├── server/       # API server
    ├── system/       # System operations
    └── threads/      # Chat thread management
```

**Key Technologies:**
- ⚠️ **Rust 1.77.2** - Systems programming language (14 versions behind 1.91.1)
- ✅ **Tauri 2.8.5** - Desktop app framework
- ✅ **Tokio** - Async runtime
- ✅ **Serde** - Serialization/deserialization

**Tauri Command Pattern:**

```rust
// src-tauri/src/core/app/commands.rs
#[tauri::command]
pub async fn get_app_configurations(
    state: State<'_, AppState>
) -> Result<Config, String> {
    // Rust business logic
    let config = load_config(&state)?;
    Ok(config)
}
```

Frontend calls this via:
```typescript
const config = await invoke('get_app_configurations')
```

🌉 **Bridge from Node.js:**
- Node.js `async/await` → Rust `async/await` (similar!)
- Express routes → Tauri commands
- JSON → Serde JSON
- npm packages → Cargo crates

---

### 3. Core Library (@janhq/core)

**Location**: [`core/src/`](../../core/src/)

**Purpose**: Shared TypeScript interfaces and types used across all packages

```
core/src/
├── types/       # TypeScript type definitions
│   ├── model.ts      # Model interfaces
│   ├── thread.ts     # Thread/message interfaces
│   └── assistant.ts  # Assistant interfaces
└── browser/     # Browser-specific implementations
```

🧠 **Mental Model:** This is the "contract" between frontend, extensions, and backend. Everyone agrees on the same types.

---

### 4. Extension System

**Two Types of Extensions:**

#### A. Native Extensions (TypeScript)

**Location**: [`extensions/`](../../extensions/)

```
extensions/
├── assistant-extension/      # AI assistant management
├── conversational-extension/ # Chat functionality core
├── download-extension/       # Model download logic
├── llamacpp-extension/       # llama.cpp integration
├── rag-extension/            # RAG (Retrieval-Augmented Generation)
└── vector-db-extension/      # Vector database operations
```

**How they work:**
1. Built separately (`yarn build:extensions`)
2. Packaged as `.tgz` files
3. Loaded dynamically by the app
4. Provide functionality through interfaces defined in `@janhq/core`

#### B. Web Extensions (TypeScript)

**Location**: [`extensions-web/src/`](../../extensions-web/src/)

```
extensions-web/src/
├── conversational-web/  # Chat for web version
├── jan-provider-web/    # Provider integration (web)
└── mcp-web/             # MCP support (web)
```

**Difference:**
- **Native**: Full access to Tauri commands (file system, etc.)
- **Web**: Browser-only, no native access

🎯 **Remember This:** "Native extensions = Desktop features. Web extensions = Browser features."

---

### 5. Tauri Plugins (Rust)

**Location**: [`src-tauri/plugins/`](../../src-tauri/plugins/)

```
src-tauri/plugins/
├── tauri-plugin-llamacpp/   # LLM inference engine
├── tauri-plugin-vector-db/  # Vector database
├── tauri-plugin-rag/        # RAG functionality
└── tauri-plugin-hardware/   # System hardware info
```

**What are Tauri Plugins?**
Custom Rust modules that extend Tauri's capabilities. They:
- Register their own commands
- Can emit events
- Access system resources
- Are initialized in `lib.rs` ([src-tauri/src/lib.rs:30-48](../../src-tauri/src/lib.rs#L30-L48))

Example:
```rust
// lib.rs
.plugin(tauri_plugin_llamacpp::init())
.plugin(tauri_plugin_vector_db::init())
.plugin(tauri_plugin_rag::init())
```

---

## Data Flow

### Example 1: User Sends a Chat Message

```
┌──────────────────────────────────────────────────────────────────────┐
│                        MESSAGE SEND FLOW                              │
└──────────────────────────────────────────────────────────────────────┘

  1. User types message in chat UI
     ↓
  2. React component calls service method
     [web-app/src/routes/threads/$threadId.tsx]
     ↓
  3. Service calls Tauri command
     [web-app/src/services/threads/]
     ↓
  4. Tauri command received in Rust
     [src-tauri/src/core/threads/commands.rs]
     ↓
  5. Rust calls llama.cpp plugin
     [src-tauri/plugins/tauri-plugin-llamacpp/]
     ↓
  6. llama.cpp processes the message
     (LLM inference happens here)
     ↓
  7. Streaming response emitted as events
     (Tauri events: "message_chunk", "message_complete")
     ↓
  8. Frontend listens to events
     [web-app/src/services/events/]
     ↓
  9. Zustand store updated
     [web-app/src/services/threads/store.ts]
     ↓
  10. React component re-renders with new message
```

**Key Technologies in Play:**
- Frontend → Backend: Tauri IPC (invoke commands)
- Backend → Frontend: Tauri Events (streaming)
- State Management: Zustand
- UI Updates: React 19 (automatic batching)

---

### Example 2: Downloading a Model

```
  1. User clicks "Download Model" in Hub
     ↓
  2. Download service calls Tauri command
     invoke('download_model', { modelId, url })
     ↓
  3. Rust download manager starts
     [src-tauri/src/core/downloads/]
     ↓
  4. Progress events emitted periodically
     emit('download_progress', { id, progress: 45% })
     ↓
  5. Frontend event listener updates UI
     [web-app/src/services/downloads/]
     ↓
  6. Zustand store tracks download state
     ↓
  7. Progress bar updates in real-time
```

**Communication Pattern:**
- Command (Frontend → Backend): One-time call
- Events (Backend → Frontend): Continuous updates

🧠 **Mental Model:**
**Commands** = "Do this task" (Request/Response)
**Events** = "Here's what's happening" (Publish/Subscribe)

---

## Technology Stack Integration

### How Everything Works Together

```
┌──────────────────────────────────────────────────────────────┐
│                    Technology Stack Integration               │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  React 19                                                     │
│    └─► Renders UI based on Zustand state                     │
│                                                                │
│  TanStack Router                                              │
│    └─► Type-safe routing with auto-generated route tree      │
│        [web-app/src/routeTree.gen.ts]                        │
│                                                                │
│  Zustand                                                       │
│    └─► Stores managed per domain (threads, models, settings) │
│        Located in: web-app/src/services/*/store.ts           │
│                                                                │
│  TailwindCSS 4                                                │
│    └─► Styles components with utility classes                │
│        Config: web-app/tailwind.config.js                    │
│                                                                │
│  Tauri 2                                                       │
│    └─► IPC bridge between React and Rust                     │
│        Commands: Rust functions callable from JS             │
│        Events: Rust emits, JavaScript listens                │
│                                                                │
│  Vite 6                                                        │
│    └─► Fast dev server + HMR (Hot Module Replacement)        │
│        Config: web-app/vite.config.ts                        │
│                                                                │
│  Vitest 3                                                      │
│    └─► Unit tests for components and services                │
│        Tests: **/__tests__/*.test.tsx                        │
│                                                                │
└──────────────────────────────────────────────────────────────┘
```

---

## Extension System

### Extension Architecture

```
┌──────────────────────────────────────────────────────────┐
│                  EXTENSION LOADING FLOW                   │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  1. App starts                                            │
│     ↓                                                      │
│  2. Extension service initializes                         │
│     [web-app/src/services/extensions/]                   │
│     ↓                                                      │
│  3. Scans extension directories                           │
│     - pre-install/*.tgz (bundled)                        │
│     - ~/jan/extensions/ (user-installed)                 │
│     ↓                                                      │
│  4. Loads extension metadata                              │
│     (package.json, manifest)                              │
│     ↓                                                      │
│  5. Initializes active extensions                         │
│     (calls extension.onLoad())                            │
│     ↓                                                      │
│  6. Extension registers its capabilities                  │
│     - Models                                               │
│     - Providers                                            │
│     - Tools                                                │
│     ↓                                                      │
│  7. Extension ready for use                               │
│                                                            │
└──────────────────────────────────────────────────────────┘
```

**Extension Interface** (from `@janhq/core`):

```typescript
interface Extension {
  onLoad(): Promise<void>
  onUnload(): Promise<void>
  // Extension-specific methods
}
```

---

## MCP Integration

### Model Context Protocol (MCP)

**What is MCP?**
MCP (Model Context Protocol) enables AI models to interact with external tools and data sources.

✅ **CURRENT (Nov 2025):** Jan uses `@modelcontextprotocol/sdk` v1.17.5

**MCP Architecture in Jan:**

```
┌─────────────────────────────────────────────────────────┐
│                    MCP ARCHITECTURE                      │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  Frontend (React)                                        │
│    ↓                                                      │
│  MCP Service                                             │
│    [web-app/src/services/mcp/]                          │
│    ↓                                                      │
│  Tauri MCP Commands                                      │
│    [src-tauri/src/core/mcp/commands.rs]                 │
│    ↓                                                      │
│  rmcp Crate (Rust MCP Client)                           │
│    [Cargo.toml: rmcp = "0.8.5"]                         │
│    ↓                                                      │
│  MCP Servers (External)                                  │
│    - Filesystem MCP                                      │
│    - Database MCP                                        │
│    - Custom MCPs                                         │
│                                                           │
└─────────────────────────────────────────────────────────┘
```

**MCP Use Cases:**
- File system access for AI
- Database queries
- Web scraping
- API integrations
- Custom tool execution

**Configuration**: MCP servers are configured in settings ([web-app/src/routes/settings/mcp-servers.tsx](../../web-app/src/routes/settings/mcp-servers.tsx))

---

## API Architecture

### OpenAI-Compatible API Server

**Purpose**: Allow other applications to use Jan's LLMs via HTTP

**Port**: `localhost:1337`

**Implementation**: Rust HTTP server in Tauri ([src-tauri/src/core/server/](../../src-tauri/src/core/server/))

```
┌────────────────────────────────────────────────────────┐
│              API SERVER ARCHITECTURE                    │
├────────────────────────────────────────────────────────┤
│                                                          │
│  External App (e.g., VS Code plugin)                   │
│    │                                                     │
│    │ POST http://localhost:1337/v1/chat/completions    │
│    ↓                                                     │
│  Rust HTTP Server                                       │
│    [src-tauri/src/core/server/]                        │
│    ↓                                                     │
│  Request Router                                          │
│    ├─ /v1/chat/completions → Chat handler              │
│    ├─ /v1/models → List models                         │
│    └─ /v1/embeddings → Embeddings                      │
│    ↓                                                     │
│  llama.cpp Plugin                                       │
│    [tauri-plugin-llamacpp]                             │
│    ↓                                                     │
│  LLM Inference                                          │
│    ↓                                                     │
│  JSON Response (OpenAI format)                          │
│                                                          │
└────────────────────────────────────────────────────────┘
```

**Supported Endpoints:**
- ✅ `/v1/chat/completions` - Chat with models
- ✅ `/v1/models` - List available models
- ✅ `/v1/embeddings` - Generate embeddings
- ⚠️ **Check source for full list**: [src-tauri/src/core/server/](../../src-tauri/src/core/server/)

🌉 **Bridge from OpenAI API:**
Same request/response format as OpenAI's API. Existing tools/libraries work without modification!

---

## Security Model

### Tauri Security

**Key Security Features:**
1. **No Remote Code Execution**: Only whitelisted commands can be called from frontend
2. **Capability System**: Fine-grained permissions
3. **CSP (Content Security Policy)**: Restricts what the frontend can do
4. **IPC Validation**: All commands validated before execution

**Command Whitelist** ([src-tauri/src/lib.rs:51-100](../../src-tauri/src/lib.rs#L51-L100)):
```rust
.invoke_handler(tauri::generate_handler![
    core::filesystem::commands::join_path,
    core::app::commands::get_app_configurations,
    // ... only listed commands are callable
])
```

💡 **Aha Moment:** Frontend can ONLY call commands explicitly listed in `generate_handler![]`. This is like an API allowlist.

---

## State Management Strategy

### Zustand Store Organization

```
web-app/src/services/
├── threads/store.ts        # Chat threads and messages
├── models/store.ts         # Downloaded models
├── assistants/store.ts     # AI assistants
├── providers/store.ts      # Cloud providers
└── app/store.ts            # App settings
```

**Store Pattern:**

```typescript
// Example: threads/store.ts
interface ThreadsState {
  threads: Thread[]
  activeThreadId: string | null
  // State
}

interface ThreadsActions {
  addThread: (thread: Thread) => void
  deleteThread: (id: string) => void
  // Actions
}

export const useThreads = create<ThreadsState & ThreadsActions>((set) => ({
  threads: [],
  activeThreadId: null,

  addThread: (thread) => set((state) => ({
    threads: [...state.threads, thread]
  })),
  // ...
}))
```

🌉 **Bridge from Redux:**
- No reducers, no actions creators, no dispatch
- Direct state mutation (with Immer-like syntax)
- Less boilerplate, same reactivity

---

## Next Steps

Now that you understand the architecture, dive deeper:

### Explore Frontend
- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - React/TypeScript deep dive
- [Project Structure](./PROJECT_STRUCTURE.md) - Where everything lives
- [Code Tours](./CODE_TOURS.md) - Follow actual code flows

### Explore Backend
- [Backend Architecture](./BACKEND_ARCHITECTURE.md) - Rust/Tauri details
- [Integration Guide](./INTEGRATION_GUIDE.md) - Extensions and plugins

### Understand Data
- [Data Flow Guide](./DATA_FLOW_GUIDE.md) - End-to-end request flows
- [Database Architecture](./DATABASE_ARCHITECTURE.md) - Data persistence

### Start Building
- [How-To Guide](./HOW_TO_GUIDE.md) - Common development tasks
- [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md) - Code standards
- [Exercises](./EXERCISES.md) - Hands-on practice

---

**Key Takeaways:**

1. 🏗️ **Layered Architecture**: Frontend (React) → Service Layer → Backend (Rust)
2. 🔌 **Plugin-Based**: Extensions add features without modifying core
3. 🔄 **Event-Driven**: Commands for requests, events for updates
4. 🌐 **Multi-Platform**: Same codebase for desktop and web
5. 🔒 **Secure by Design**: Tauri's security model prevents unauthorized access

**Questions?** Check the [FAQ](./FAQ.md) or return to the [Learning Hub](./README.md).
