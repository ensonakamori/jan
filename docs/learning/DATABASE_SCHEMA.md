# Database Schema

**Complete data model reference for Jan AI**

**Documented:** November 18, 2025

---

## Storage Model

Jan AI uses **file-based storage** with JSON files. No database server required.

**Storage Location:** `~/jan/` (configurable)

```
~/jan/
├── threads/              # Chat conversations
│   ├── thread-{uuid}.json
│   └── thread-{uuid}.json
├── models/               # Downloaded models
│   └── {model-id}/
│       ├── model.gguf
│       └── metadata.json
├── settings/             # App configuration
│   └── settings.json
├── extensions/           # Installed extensions
│   └── {extension-name}/
│       └── package.json
└── store.json            # Tauri store (key-value)
```

---

## Thread Schema

**File:** `~/jan/threads/{thread-id}.json`

**TypeScript Interface:** [core/src/types/thread/threadEntity.ts](../../core/src/types/thread/threadEntity.ts)

### Full Schema

```json
{
  "id": "thread-550e8400-e29b-41d4-a716-446655440000",
  "title": "My Chat with AI",
  "assistantId": "assistant-jan",
  "createdAt": 1700000000000,
  "updatedAt": 1700000500000,
  "metadata": {
    "tags": ["important", "work"],
    "pinned": false
  }
}
```

### Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique thread ID (UUID v4) |
| `title` | string | ✅ | Thread title (user-visible) |
| `assistantId` | string | ❌ | Associated assistant ID |
| `createdAt` | number | ✅ | Unix timestamp (milliseconds) |
| `updatedAt` | number | ✅ | Unix timestamp (milliseconds) |
| `metadata` | object | ❌ | Additional key-value data |

### TypeScript Definition

```typescript
export interface Thread {
  id: string
  title: string
  assistantId?: string
  createdAt: number
  updatedAt: number
  metadata?: Record<string, any>
}
```

### Validation Rules

- `id`: Must be valid UUID v4
- `title`: Min 1 char, max 200 chars
- `createdAt`: Must be > 0
- `updatedAt`: Must be >= createdAt
- `assistantId`: If provided, must reference existing assistant

### Examples

**Minimal thread:**
```json
{
  "id": "thread-abc123",
  "title": "Untitled",
  "createdAt": 1700000000000,
  "updatedAt": 1700000000000
}
```

**Thread with metadata:**
```json
{
  "id": "thread-abc123",
  "title": "Project Planning",
  "assistantId": "assistant-gpt4",
  "createdAt": 1700000000000,
  "updatedAt": 1700005000000,
  "metadata": {
    "tags": ["work", "planning"],
    "pinned": true,
    "color": "#FF5733",
    "archived": false
  }
}
```

---

## Message Schema

**Storage:** Messages are stored **within thread files** as an array

⚠️ **UNCLEAR:** Current implementation may store messages separately or within threads. Check actual implementation.

### Schema

```json
{
  "id": "msg-550e8400-e29b-41d4-a716-446655440001",
  "thread_id": "thread-550e8400-e29b-41d4-a716-446655440000",
  "role": "user",
  "content": "Hello, AI!",
  "status": "completed",
  "createdAt": 1700000001000,
  "updatedAt": 1700000001000,
  "metadata": {
    "tokens": 5,
    "model": "gpt-4"
  }
}
```

### Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique message ID |
| `thread_id` | string | ✅ | Parent thread ID |
| `role` | enum | ✅ | `"user"` \| `"assistant"` \| `"system"` |
| `content` | string | ✅ | Message text content |
| `status` | enum | ❌ | `"queued"` \| `"in_progress"` \| `"completed"` \| `"failed"` |
| `createdAt` | number | ✅ | Unix timestamp (ms) |
| `updatedAt` | number | ✅ | Unix timestamp (ms) |
| `metadata` | object | ❌ | Additional data |

### TypeScript Definition

```typescript
export interface Message {
  id: string
  thread_id: string
  role: 'user' | 'assistant' | 'system'
  content: string
  status?: 'queued' | 'in_progress' | 'completed' | 'failed'
  createdAt: number
  updatedAt: number
  metadata?: MessageMetadata
}

export interface MessageMetadata {
  tokens?: number
  model?: string
  finish_reason?: 'stop' | 'length' | 'content_filter'
  [key: string]: any
}
```

### Examples

**User message:**
```json
{
  "id": "msg-user-1",
  "thread_id": "thread-123",
  "role": "user",
  "content": "What is the capital of France?",
  "status": "completed",
  "createdAt": 1700000001000,
  "updatedAt": 1700000001000
}
```

**Assistant message:**
```json
{
  "id": "msg-assistant-1",
  "thread_id": "thread-123",
  "role": "assistant",
  "content": "The capital of France is Paris.",
  "status": "completed",
  "createdAt": 1700000002000,
  "updatedAt": 1700000002000,
  "metadata": {
    "model": "llama-2-7b",
    "tokens": 12,
    "finish_reason": "stop",
    "inference_time_ms": 450
  }
}
```

**System message:**
```json
{
  "id": "msg-system-1",
  "thread_id": "thread-123",
  "role": "system",
  "content": "You are a helpful assistant.",
  "status": "completed",
  "createdAt": 1700000000000,
  "updatedAt": 1700000000000
}
```

---

## Model Metadata Schema

**File:** `~/jan/models/{model-id}/metadata.json`

### Schema

```json
{
  "id": "llama-2-7b-chat",
  "name": "Llama 2 7B Chat",
  "version": "1.0.0",
  "format": "gguf",
  "size": 3825082368,
  "engine": "llamacpp",
  "parameters": {
    "context_length": 4096,
    "temperature": 0.7,
    "top_p": 0.9,
    "max_tokens": 2048,
    "stream": true
  },
  "downloadedAt": 1700000000000,
  "source_url": "https://huggingface.co/meta-llama/Llama-2-7b-chat",
  "metadata": {
    "license": "Llama 2 Community License",
    "architecture": "llama",
    "quantization": "Q4_K_M"
  }
}
```

### Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique model identifier |
| `name` | string | ✅ | Human-readable name |
| `version` | string | ✅ | Model version (semver) |
| `format` | enum | ✅ | `"gguf"` \| `"onnx"` |
| `size` | number | ✅ | File size in bytes |
| `engine` | enum | ❌ | `"llamacpp"` \| `"onnx"` |
| `parameters` | object | ✅ | Inference parameters |
| `downloadedAt` | number | ❌ | Unix timestamp (ms) |
| `source_url` | string | ❌ | Download source |
| `metadata` | object | ❌ | Additional info |

### TypeScript Definition

```typescript
export interface Model {
  id: string
  name: string
  version: string
  format: 'gguf' | 'onnx'
  size: number
  engine?: 'llamacpp' | 'onnx'
  parameters: ModelParameters
  downloadedAt?: number
  source_url?: string
  metadata?: ModelMetadata
}

export interface ModelParameters {
  context_length: number
  temperature?: number
  top_p?: number
  top_k?: number
  max_tokens?: number
  stream?: boolean
  stop?: string[]
  frequency_penalty?: number
  presence_penalty?: number
}

export interface ModelMetadata {
  license?: string
  architecture?: string
  quantization?: string
  [key: string]: any
}
```

### Examples

**GGUF model:**
```json
{
  "id": "mistral-7b-instruct-v0.2",
  "name": "Mistral 7B Instruct v0.2",
  "version": "0.2.0",
  "format": "gguf",
  "size": 4368421888,
  "engine": "llamacpp",
  "parameters": {
    "context_length": 8192,
    "temperature": 0.7,
    "top_p": 0.95,
    "max_tokens": 4096
  },
  "downloadedAt": 1700500000000,
  "source_url": "https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2",
  "metadata": {
    "license": "Apache 2.0",
    "architecture": "mistral",
    "quantization": "Q5_K_M"
  }
}
```

---

## App Configuration Schema

**File:** `~/jan/settings/settings.json`

**TypeScript Interface:** [config/appConfigEntity.ts](../../core/src/types/config/appConfigEntity.ts)

### Schema

```json
{
  "data_folder": "/Users/me/jan",
  "theme": "dark",
  "language": "en",
  "notifications_enabled": true,
  "api_server": {
    "enabled": true,
    "port": 1337,
    "host": "127.0.0.1",
    "cors_enabled": false
  },
  "privacy": {
    "crash_report": true,
    "analytics": false
  },
  "model_settings": {
    "default_context_length": 4096,
    "gpu_acceleration": true
  }
}
```

### Field Definitions

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `data_folder` | string | ✅ | `~/jan` | Data storage path |
| `theme` | enum | ❌ | `"dark"` | `"light"` \| `"dark"` \| `"system"` |
| `language` | string | ❌ | `"en"` | ISO 639-1 code |
| `notifications_enabled` | boolean | ❌ | `true` | Enable notifications |
| `api_server` | object | ❌ | (see below) | API server config |
| `privacy` | object | ❌ | (see below) | Privacy settings |
| `model_settings` | object | ❌ | (see below) | Default model settings |

### TypeScript Definition

```typescript
export interface AppConfiguration {
  data_folder: string
  theme?: 'light' | 'dark' | 'system'
  language?: string
  notifications_enabled?: boolean
  api_server?: ApiServerConfig
  privacy?: PrivacyConfig
  model_settings?: ModelSettings
}

export interface ApiServerConfig {
  enabled: boolean
  port: number
  host: string
  cors_enabled?: boolean
}

export interface PrivacyConfig {
  crash_report: boolean
  analytics: boolean
}

export interface ModelSettings {
  default_context_length?: number
  gpu_acceleration?: boolean
}
```

---

## Extension Manifest Schema

**File:** `~/jan/extensions/{extension-name}/package.json`

### Schema

```json
{
  "name": "@janhq/my-extension",
  "version": "1.0.0",
  "description": "My custom extension",
  "main": "dist/index.js",
  "engine": {
    "jan": ">=0.5.0"
  },
  "dependencies": {
    "@janhq/core": "workspace:*"
  },
  "janExtension": {
    "id": "my-extension",
    "displayName": "My Extension",
    "type": "inference",
    "permissions": ["filesystem", "network"],
    "settings": [
      {
        "key": "api_key",
        "title": "API Key",
        "type": "string",
        "required": true
      }
    ]
  }
}
```

### Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | NPM package name |
| `version` | string | ✅ | Semver version |
| `main` | string | ✅ | Entry point file |
| `engine.jan` | string | ✅ | Compatible Jan version |
| `janExtension` | object | ✅ | Extension metadata |

### janExtension Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique extension ID |
| `displayName` | string | ✅ | User-visible name |
| `type` | enum | ✅ | `"inference"` \| `"model"` \| `"ui"` |
| `permissions` | array | ❌ | Required permissions |
| `settings` | array | ❌ | Configurable settings |

### TypeScript Definition

```typescript
export interface ExtensionManifest {
  name: string
  version: string
  description?: string
  main: string
  engine: {
    jan: string
  }
  dependencies?: Record<string, string>
  janExtension: JanExtensionMetadata
}

export interface JanExtensionMetadata {
  id: string
  displayName: string
  type: 'inference' | 'model' | 'ui'
  permissions?: string[]
  settings?: ExtensionSetting[]
}

export interface ExtensionSetting {
  key: string
  title: string
  type: 'string' | 'number' | 'boolean' | 'select'
  required?: boolean
  default?: any
  options?: Array<{ label: string; value: any }>
}
```

---

## MCP Configuration Schema

🆕 **NEW IN 2025:** Model Context Protocol server configuration

**File:** `~/jan/mcp-config.json` or in app settings

### Schema

```json
{
  "servers": [
    {
      "name": "filesystem",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"],
      "env": {
        "NODE_ENV": "production"
      },
      "enabled": true,
      "auto_start": true
    },
    {
      "name": "github",
      "command": "node",
      "args": ["github-mcp-server.js"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxxx"
      },
      "enabled": false,
      "auto_start": false
    }
  ]
}
```

### Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `servers` | array | ✅ | List of MCP servers |

### Server Object Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Unique server name |
| `command` | string | ✅ | Executable command |
| `args` | array | ❌ | Command arguments |
| `env` | object | ❌ | Environment variables |
| `enabled` | boolean | ❌ | Server is enabled |
| `auto_start` | boolean | ❌ | Start on app launch |

### TypeScript Definition

```typescript
export interface McpConfiguration {
  servers: McpServer[]
}

export interface McpServer {
  name: string
  command: string
  args?: string[]
  env?: Record<string, string>
  enabled?: boolean
  auto_start?: boolean
}
```

---

## Tauri Store Schema

**File:** `~/jan/store.json`

⚠️ **Implementation Detail:** Tauri's key-value store

### Schema

```json
{
  "last_active_thread": "thread-123",
  "sidebar_width": 280,
  "recent_models": ["llama-2-7b", "mistral-7b"],
  "onboarding_completed": true
}
```

**Type:** Generic key-value storage

```typescript
type StoreValue = string | number | boolean | object | array
type Store = Record<string, StoreValue>
```

---

## File Naming Conventions

### Thread Files

**Pattern:** `thread-{uuid}.json`

**Example:** `thread-550e8400-e29b-41d4-a716-446655440000.json`

**Rules:**
- Must start with `thread-`
- Followed by UUID v4
- Extension: `.json`

### Model Directories

**Pattern:** `{model-id}/`

**Example:** `llama-2-7b-chat/`

**Rules:**
- Lowercase, kebab-case
- No spaces or special characters
- Descriptive and unique

### Model Files

**Pattern:** `model.{extension}`

**Examples:**
- `model.gguf` (GGUF format)
- `model.onnx` (ONNX format)
- `metadata.json` (metadata)

---

## Data Integrity

### Atomic Writes

**Pattern:** Write to temp file, then rename

```rust
// Atomic write pattern
let temp_path = path.with_extension("tmp");
fs::write(&temp_path, content)?;
fs::rename(temp_path, path)?;
```

**Why:** Prevents corruption if write is interrupted

### Validation on Read

```rust
// Always validate after reading
let content = fs::read_to_string(path)?;
let thread: Thread = serde_json::from_str(&content)?;

// Validate fields
if thread.created_at == 0 {
    return Err("Invalid thread: created_at is 0".to_string());
}
```

---

## Migration Patterns

⚠️ **UNCLEAR:** No explicit migration system found

**Recommendation:** Add version field to schemas

```json
{
  "schema_version": "1.0.0",
  "id": "thread-123",
  ...
}
```

**Migration strategy:**
```rust
fn migrate_thread(data: Value) -> Result<Thread, String> {
    let version = data["schema_version"].as_str().unwrap_or("0.0.0");

    match version {
        "0.0.0" => migrate_v0_to_v1(data),
        "1.0.0" => serde_json::from_value(data).map_err(|e| e.to_string()),
        _ => Err(format!("Unknown schema version: {}", version)),
    }
}
```

---

## Backup & Recovery

### Manual Backup

**Entire data folder:**
```bash
cp -r ~/jan ~/jan-backup-2025-11-18
```

**Specific data:**
```bash
cp -r ~/jan/threads ~/backups/threads
cp -r ~/jan/settings ~/backups/settings
```

### Programmatic Backup

⚠️ **TODO:** Not yet implemented

**Proposed API:**
```typescript
await invoke('backup_data', {
  destination: '/path/to/backup',
  include: ['threads', 'settings'],
  exclude: ['models'], // Large files
})
```

---

## Performance Considerations

### Large Files

**Problem:** Threads with 1000+ messages may be slow to parse

**Solution:** Implement pagination or message chunking

```json
{
  "id": "thread-123",
  "title": "Long conversation",
  "message_count": 1500,
  "message_chunks": [
    "messages-0-499.json",
    "messages-500-999.json",
    "messages-1000-1499.json"
  ]
}
```

### Indexing

**Current:** No indexing (linear search)

**Future consideration:** SQLite for metadata

```sql
CREATE TABLE threads (
  id TEXT PRIMARY KEY,
  title TEXT,
  created_at INTEGER,
  updated_at INTEGER
);

CREATE INDEX idx_threads_updated ON threads(updated_at DESC);
```

---

## Schema Validation

### Runtime Validation (TypeScript)

```typescript
import { z } from 'zod'

const ThreadSchema = z.object({
  id: z.string().uuid(),
  title: z.string().min(1).max(200),
  createdAt: z.number().positive(),
  updatedAt: z.number().positive(),
  assistantId: z.string().optional(),
  metadata: z.record(z.any()).optional(),
})

// Validate
const thread = ThreadSchema.parse(data)
```

### Rust Validation

```rust
use validator::Validate;

#[derive(Deserialize, Validate)]
pub struct Thread {
    #[validate(length(min = 1))]
    pub id: String,

    #[validate(length(min = 1, max = 200))]
    pub title: String,

    #[validate(range(min = 1))]
    pub created_at: i64,

    #[validate(range(min = 1))]
    pub updated_at: i64,
}

// Validate
thread.validate()?;
```

---

## Example Queries

### Find All Threads (Rust)

```rust
use std::fs;

pub fn list_threads(data_folder: &Path) -> Result<Vec<Thread>, String> {
    let threads_dir = data_folder.join("threads");
    let mut threads = Vec::new();

    for entry in fs::read_dir(threads_dir).map_err(|e| e.to_string())? {
        let entry = entry.map_err(|e| e.to_string())?;
        let path = entry.path();

        if path.extension().and_then(|s| s.to_str()) == Some("json") {
            let content = fs::read_to_string(&path)
                .map_err(|e| e.to_string())?;
            let thread: Thread = serde_json::from_str(&content)
                .map_err(|e| e.to_string())?;
            threads.push(thread);
        }
    }

    // Sort by updated_at descending
    threads.sort_by(|a, b| b.updated_at.cmp(&a.updated_at));

    Ok(threads)
}
```

### Search Threads by Title (TypeScript)

```typescript
async function searchThreads(query: string): Promise<Thread[]> {
  const allThreads = await invoke<Thread[]>('list_threads')

  return allThreads.filter(thread =>
    thread.title.toLowerCase().includes(query.toLowerCase())
  )
}
```

---

## Next Steps

- **API Reference:** [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)
- **Integration:** [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)
- **Architecture:** [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)

**Questions?** Return to the [Learning Hub](./README.md)
