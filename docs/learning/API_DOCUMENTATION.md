# API Documentation

**Complete API reference for Jan AI**

**Documented:** November 18, 2025

---

## API Layers

Jan AI has 3 main API layers:

```
┌─────────────────────────────────┐
│  Frontend (React Components)    │
├─────────────────────────────────┤
│  Services Layer                 │  ← Platform abstraction
│  (Tauri / Web / Default)        │
├─────────────────────────────────┤
│  Tauri Commands                 │  ← Backend (Rust)
│  (IPC boundary)                 │
├─────────────────────────────────┤
│  Core Types (@janhq/core)       │  ← Shared types
└─────────────────────────────────┘
```

---

## Tauri Commands

**Location:** [src-tauri/src/lib.rs](../../src-tauri/src/lib.rs#L51-L117)

All commands accessible via `invoke('command_name', { params })`

### App Configuration

#### `get_app_configurations()`

Get application configuration

**Returns:** `AppConfiguration`

```typescript
import { invoke } from '@tauri-apps/api/core'

interface AppConfiguration {
  data_folder: string
  // ... other fields
}

const config = await invoke<AppConfiguration>('get_app_configurations')
console.log('Data folder:', config.data_folder)
```

**Source:** [commands.rs:10-49](../../src-tauri/src/core/app/commands.rs#L10-L49)

---

#### `update_app_configuration(configuration: AppConfiguration)`

Update application configuration

**Parameters:**
- `configuration`: AppConfiguration object

**Returns:** `Result<(), string>`

```typescript
await invoke('update_app_configuration', {
  configuration: {
    data_folder: '/new/path',
    // ... other fields
  }
})
```

**Source:** [commands.rs:51-64](../../src-tauri/src/core/app/commands.rs#L51-L64)

---

#### `get_jan_data_folder_path()`

Get the current Jan data folder path

**Returns:** `string`

```typescript
const dataFolder = await invoke<string>('get_jan_data_folder_path')
// Returns: "~/jan" or user-configured path
```

**Source:** [commands.rs:66-80](../../src-tauri/src/core/app/commands.rs#L66-L80)

---

#### `change_app_data_folder(new_data_folder: string)`

Change the data folder and migrate files

**Parameters:**
- `new_data_folder`: Absolute path to new folder

**Returns:** `Result<(), string>`

**Security:** Prevents subdirectory recursion

```typescript
try {
  await invoke('change_app_data_folder', {
    new_data_folder: '/Users/me/new-jan-data'
  })
  // Files are copied automatically
} catch (error) {
  console.error('Migration failed:', error)
}
```

**Source:** [commands.rs:150-190](../../src-tauri/src/core/app/commands.rs#L150-L190)

---

#### `app_token()`

Get the app authentication token

**Returns:** `Option<string>`

```typescript
const token = await invoke<string | null>('app_token')
// Used for authenticating with local API server
```

**Source:** [commands.rs:193-195](../../src-tauri/src/core/app/commands.rs#L193-L195)

---

### Filesystem Commands

⚠️ **Deprecated:** Will be replaced by Tauri's built-in fs plugin

#### `join_path(paths: string[])`

Join path segments

**Parameters:**
- `paths`: Array of path segments

**Returns:** `string`

```typescript
const fullPath = await invoke<string>('join_path', {
  paths: ['/home/user', 'jan', 'models']
})
// Returns: "/home/user/jan/models"
```

---

#### `exists_sync(path: string)`

Check if file/directory exists

**Parameters:**
- `path`: Path to check

**Returns:** `boolean`

```typescript
const exists = await invoke<boolean>('exists_sync', {
  path: '/home/user/jan/models/llama-2-7b'
})
```

---

#### `readdir_sync(path: string)`

List directory contents

**Parameters:**
- `path`: Directory path

**Returns:** `string[]` (file/directory names)

```typescript
const files = await invoke<string[]>('readdir_sync', {
  path: '/home/user/jan/threads'
})
// Returns: ["thread-1.json", "thread-2.json", ...]
```

---

#### `read_file_sync(path: string)`

Read file contents

**Parameters:**
- `path`: File path

**Returns:** `string`

```typescript
const content = await invoke<string>('read_file_sync', {
  path: '/home/user/jan/settings.json'
})
const settings = JSON.parse(content)
```

---

#### `write_file_sync(path: string, data: string)`

Write file contents

**Parameters:**
- `path`: File path
- `data`: Content to write

**Returns:** `Result<(), string>`

```typescript
await invoke('write_file_sync', {
  path: '/home/user/jan/my-file.txt',
  data: 'Hello, world!'
})
```

---

#### `rm(path: string)`

Delete file or directory

**Parameters:**
- `path`: Path to delete

**Returns:** `Result<(), string>`

```typescript
await invoke('rm', {
  path: '/home/user/jan/old-thread.json'
})
```

---

#### `mv(source: string, destination: string)`

Move/rename file or directory

**Parameters:**
- `source`: Source path
- `destination`: Destination path

**Returns:** `Result<(), string>`

```typescript
await invoke('mv', {
  source: '/home/user/jan/old-name.json',
  destination: '/home/user/jan/new-name.json'
})
```

---

### Thread Commands

#### `list_threads()`

List all threads

**Returns:** `Thread[]`

```typescript
interface Thread {
  id: string
  title: string
  created_at: number
  updated_at: number
  // ... other fields
}

const threads = await invoke<Thread[]>('list_threads')
```

---

#### `create_thread(thread: Thread)`

Create a new thread

**Parameters:**
- `thread`: Thread object

**Returns:** `Result<Thread, string>`

```typescript
const newThread = await invoke<Thread>('create_thread', {
  thread: {
    id: crypto.randomUUID(),
    title: 'My Chat',
    created_at: Date.now(),
    updated_at: Date.now(),
  }
})
```

---

#### `modify_thread(thread: Thread)`

Update an existing thread

**Parameters:**
- `thread`: Updated thread object

**Returns:** `Result<Thread, string>`

```typescript
await invoke('modify_thread', {
  thread: {
    ...existingThread,
    title: 'Updated Title',
    updated_at: Date.now(),
  }
})
```

---

#### `delete_thread(id: string)`

Delete a thread

**Parameters:**
- `id`: Thread ID

**Returns:** `Result<(), string>`

```typescript
await invoke('delete_thread', { id: 'thread-123' })
```

---

#### `list_messages(thread_id: string)`

Get messages for a thread

**Parameters:**
- `thread_id`: Thread ID

**Returns:** `Message[]`

```typescript
interface Message {
  id: string
  role: 'user' | 'assistant' | 'system'
  content: string
  created_at: number
}

const messages = await invoke<Message[]>('list_messages', {
  thread_id: 'thread-123'
})
```

---

#### `create_message(thread_id: string, message: Message)`

Add message to thread

**Parameters:**
- `thread_id`: Thread ID
- `message`: Message object

**Returns:** `Result<Message, string>`

```typescript
await invoke('create_message', {
  thread_id: 'thread-123',
  message: {
    id: crypto.randomUUID(),
    role: 'user',
    content: 'Hello!',
    created_at: Date.now(),
  }
})
```

---

### Extension Commands

#### `get_jan_extensions_path()`

Get extensions directory path

**Returns:** `string`

```typescript
const extensionsPath = await invoke<string>('get_jan_extensions_path')
// Returns: "~/jan/extensions"
```

---

#### `install_extensions(extensions: Extension[])`

Install one or more extensions

**Parameters:**
- `extensions`: Array of extension metadata

**Returns:** `Result<(), string>`

```typescript
await invoke('install_extensions', {
  extensions: [
    {
      name: 'my-extension',
      version: '1.0.0',
      // ... other metadata
    }
  ]
})
```

---

#### `get_active_extensions()`

Get list of active extensions

**Returns:** `Extension[]`

```typescript
const extensions = await invoke<Extension[]>('get_active_extensions')
```

---

### System Commands

#### `relaunch()`

Restart the application

**Returns:** Never (app exits)

```typescript
await invoke('relaunch')
// App restarts
```

---

#### `open_app_directory()`

Open app data directory in file explorer

**Returns:** `Result<(), string>`

```typescript
await invoke('open_app_directory')
// Opens ~/jan in Finder/Explorer
```

---

#### `open_file_explorer(path: string)`

Open specific path in file explorer

**Parameters:**
- `path`: Path to open

**Returns:** `Result<(), string>`

```typescript
await invoke('open_file_explorer', {
  path: '/home/user/jan/models'
})
```

---

#### `factory_reset()`

Reset app to factory defaults

**Returns:** `Result<(), string>`

⚠️ **Warning:** Deletes all user data

```typescript
if (confirm('Delete all data?')) {
  await invoke('factory_reset')
}
```

---

#### `read_logs()`

Read application logs

**Returns:** `string`

```typescript
const logs = await invoke<string>('read_logs')
console.log('App logs:', logs)
```

---

#### `is_library_available(library: string)`

Check if native library is available

**Parameters:**
- `library`: Library name (e.g., "cuda", "metal")

**Returns:** `boolean`

```typescript
const hasCuda = await invoke<boolean>('is_library_available', {
  library: 'cuda'
})
```

---

### Server Commands

#### `start_server(config: ServerConfig)`

Start local API server

**Parameters:**
- `config`: Server configuration

**Returns:** `Result<(), string>`

```typescript
await invoke('start_server', {
  config: {
    port: 1337,
    host: '127.0.0.1',
  }
})
```

---

#### `stop_server()`

Stop local API server

**Returns:** `Result<(), string>`

```typescript
await invoke('stop_server')
```

---

#### `get_server_status()`

Get server status

**Returns:** `ServerStatus`

```typescript
interface ServerStatus {
  running: boolean
  port?: number
}

const status = await invoke<ServerStatus>('get_server_status')
if (status.running) {
  console.log('Server running on port', status.port)
}
```

---

### MCP Commands

🆕 **NEW IN 2025:** Model Context Protocol integration

#### `get_tools(server_name: string)`

Get available tools from MCP server

**Parameters:**
- `server_name`: MCP server name

**Returns:** `Tool[]`

```typescript
const tools = await invoke<Tool[]>('get_tools', {
  server_name: 'my-mcp-server'
})
```

---

#### `call_tool(server_name: string, tool_name: string, arguments: any)`

Call an MCP tool

**Parameters:**
- `server_name`: MCP server name
- `tool_name`: Tool name
- `arguments`: Tool arguments

**Returns:** `any` (tool result)

```typescript
const result = await invoke('call_tool', {
  server_name: 'filesystem',
  tool_name: 'read_file',
  arguments: { path: '/path/to/file.txt' }
})
```

---

#### `get_mcp_configs()`

Get MCP server configurations

**Returns:** `McpConfig[]`

```typescript
const configs = await invoke<McpConfig[]>('get_mcp_configs')
```

---

#### `save_mcp_configs(configs: McpConfig[])`

Save MCP configurations

**Parameters:**
- `configs`: MCP configurations

**Returns:** `Result<(), string>`

```typescript
await invoke('save_mcp_configs', {
  configs: [
    {
      name: 'my-server',
      command: 'node',
      args: ['server.js'],
      env: {}
    }
  ]
})
```

---

### Download Commands

#### `download_files(url: string, destination: string)`

Download file from URL

**Parameters:**
- `url`: Download URL
- `destination`: Save path

**Returns:** `Result<(), string>`

**Events:** Emits `download_progress` events

```typescript
// Start download
invoke('download_files', {
  url: 'https://example.com/model.gguf',
  destination: '/home/user/jan/models/model.gguf'
})

// Listen for progress
listen<{ progress: number }>('download_progress', (event) => {
  console.log(`Downloaded: ${event.payload.progress}%`)
})
```

---

#### `cancel_download_task(task_id: string)`

Cancel ongoing download

**Parameters:**
- `task_id`: Download task ID

**Returns:** `Result<(), string>`

```typescript
await invoke('cancel_download_task', { task_id: 'download-123' })
```

---

## Services

**Location:** [web-app/src/services/](../../web-app/src/services/)

Services provide platform-specific implementations (Tauri, Web, Default)

### Service Pattern

```typescript
// Service interface
export interface MyService {
  doSomething(): Promise<string>
}

// Tauri implementation
export const tauriMyService: MyService = {
  async doSomething() {
    return await invoke('do_something')
  }
}

// Web implementation
export const webMyService: MyService = {
  async doSomething() {
    // Web-specific logic
    return 'web result'
  }
}

// Default (fallback)
export const myService: MyService = isTauri()
  ? tauriMyService
  : webMyService
```

### Core Service

**Location:** [services/core/types.ts](../../web-app/src/services/core/types.ts)

```typescript
interface CoreService {
  getAppConfigurations(): Promise<AppConfiguration>
  updateAppConfiguration(config: AppConfiguration): Promise<void>
  getDataFolder(): Promise<string>
  changeDataFolder(path: string): Promise<void>
}
```

**Usage:**
```typescript
import { coreService } from '@/services/core'

const config = await coreService.getAppConfigurations()
```

---

### Thread Service

**Location:** [services/threads/types.ts](../../web-app/src/services/threads/types.ts)

```typescript
interface ThreadService {
  listThreads(): Promise<Thread[]>
  createThread(thread: Thread): Promise<Thread>
  updateThread(thread: Thread): Promise<void>
  deleteThread(id: string): Promise<void>
  getMessages(threadId: string): Promise<Message[]>
  addMessage(threadId: string, message: Message): Promise<void>
}
```

**Usage:**
```typescript
import { threadService } from '@/services/threads'

const threads = await threadService.listThreads()
```

---

### Path Service

**Location:** [services/path/types.ts](../../web-app/src/services/path/types.ts)

```typescript
interface PathService {
  join(...paths: string[]): string
  basename(path: string): string
  dirname(path: string): string
  resolve(...paths: string[]): string
}
```

**Usage:**
```typescript
import { pathService } from '@/services/path'

const fullPath = pathService.join('/home', 'user', 'jan')
```

---

### RAG Service

**Location:** [services/rag/types.ts](../../web-app/src/services/rag/types.ts)

```typescript
interface RAGService {
  addDocument(doc: Document): Promise<void>
  search(query: string, topK?: number): Promise<SearchResult[]>
  deleteDocument(id: string): Promise<void>
}
```

**Usage:**
```typescript
import { ragService } from '@/services/rag'

const results = await ragService.search('my query', 5)
```

---

## Events

**Pattern:** Backend emits events, Frontend listens

### Listening to Events

```typescript
import { listen } from '@tauri-apps/api/event'

useEffect(() => {
  const unlisten = listen<PayloadType>('event_name', (event) => {
    console.log('Event:', event.payload)
  })

  return () => {
    unlisten.then((fn) => fn())
  }
}, [])
```

### Common Events

#### `download_progress`

**Payload:**
```typescript
{
  task_id: string
  progress: number // 0.0 to 1.0
  downloaded: number // bytes
  total: number // bytes
}
```

**Usage:**
```typescript
listen<DownloadProgress>('download_progress', (event) => {
  setProgress(event.payload.progress * 100)
})
```

---

#### `download_complete`

**Payload:**
```typescript
{
  task_id: string
  path: string
}
```

---

#### `download_error`

**Payload:**
```typescript
{
  task_id: string
  error: string
}
```

---

#### `model_loaded`

**Payload:**
```typescript
{
  model_id: string
}
```

---

#### `inference_started`

**Payload:**
```typescript
{
  thread_id: string
  message_id: string
}
```

---

#### `inference_token`

**Payload:**
```typescript
{
  token: string
  message_id: string
}
```

**Usage:**
```typescript
// Stream LLM response
listen<{ token: string }>('inference_token', (event) => {
  appendToMessage(event.payload.token)
})
```

---

## Zustand Stores

**Pattern:** State management for frontend

### useThreads Store

**Location:** [stores/threads.ts](../../web-app/src/stores/threads.ts) (if exists)

```typescript
interface ThreadsState {
  threads: Thread[]
  activeThreadId: string | null
}

interface ThreadsActions {
  setThreads: (threads: Thread[]) => void
  addThread: (thread: Thread) => void
  removeThread: (id: string) => void
  setActiveThread: (id: string) => void
}

const useThreads = create<ThreadsState & ThreadsActions>((set) => ({
  threads: [],
  activeThreadId: null,

  setThreads: (threads) => set({ threads }),

  addThread: (thread) =>
    set((state) => ({ threads: [...state.threads, thread] })),

  removeThread: (id) =>
    set((state) => ({
      threads: state.threads.filter((t) => t.id !== id),
    })),

  setActiveThread: (id) => set({ activeThreadId: id }),
}))
```

**Usage:**
```typescript
function ThreadList() {
  const threads = useThreads((state) => state.threads)
  const addThread = useThreads((state) => state.addThread)

  return (
    <div>
      {threads.map(thread => <div key={thread.id}>{thread.title}</div>)}
      <button onClick={() => addThread(newThread)}>Add</button>
    </div>
  )
}
```

---

## Core Types

**Location:** [core/src/types/](../../core/src/types/)

### Thread

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

**Source:** [thread/threadEntity.ts](../../core/src/types/thread/threadEntity.ts)

---

### Message

```typescript
export interface Message {
  id: string
  thread_id: string
  role: 'user' | 'assistant' | 'system'
  content: string
  status?: 'queued' | 'in_progress' | 'completed' | 'failed'
  createdAt: number
  updatedAt: number
}
```

---

### Model

```typescript
export interface Model {
  id: string
  name: string
  version: string
  format: 'gguf' | 'onnx'
  parameters: ModelParameters
  engine?: 'llamacpp' | 'onnx'
}

export interface ModelParameters {
  temperature?: number
  top_p?: number
  max_tokens?: number
  stream?: boolean
  // ... more
}
```

---

### AppConfiguration

```typescript
export interface AppConfiguration {
  data_folder: string
  theme?: 'light' | 'dark'
  language?: string
  // ... more settings
}
```

**Source:** [config/appConfigEntity.ts](../../core/src/types/config/appConfigEntity.ts)

---

## Extension API

**Creating an extension:**

```typescript
// my-extension/src/index.ts
import { Extension } from '@janhq/core'

export default class MyExtension implements Extension {
  // Called when extension loads
  async onLoad() {
    console.log('Extension loaded')
    // Register providers, services, etc.
  }

  // Called when extension unloads
  async onUnload() {
    console.log('Extension unloaded')
    // Cleanup
  }
}
```

**Package structure:**
```
my-extension/
├── package.json
├── src/
│   └── index.ts
└── dist/
    └── index.js
```

**package.json:**
```json
{
  "name": "@janhq/my-extension",
  "version": "1.0.0",
  "main": "dist/index.js",
  "dependencies": {
    "@janhq/core": "workspace:*"
  }
}
```

---

## Utility Functions

### isTauri()

Check if running in Tauri

```typescript
import { isTauri } from '@/utils/platform'

if (isTauri()) {
  // Use Tauri APIs
} else {
  // Use web fallback
}
```

---

### generateId()

Generate unique ID

```typescript
const id = crypto.randomUUID()
// "550e8400-e29b-41d4-a716-446655440000"
```

---

## Usage Examples

### Complete Flow: Send Message

```typescript
import { invoke } from '@tauri-apps/api/core'
import { listen } from '@tauri-apps/api/event'
import { useThreads } from '@/stores/threads'

async function sendMessage(threadId: string, content: string) {
  // 1. Create message
  const message: Message = {
    id: crypto.randomUUID(),
    thread_id: threadId,
    role: 'user',
    content,
    createdAt: Date.now(),
    updatedAt: Date.now(),
  }

  // 2. Save message
  await invoke('create_message', {
    thread_id: threadId,
    message,
  })

  // 3. Listen for response
  const unlisten = await listen<{ token: string }>(
    'inference_token',
    (event) => {
      // Append token to assistant message
      appendToken(event.payload.token)
    }
  )

  // 4. Start inference (hypothetical command)
  await invoke('start_inference', { thread_id: threadId })

  // 5. Cleanup listener when done
  return unlisten
}
```

---

### Complete Flow: Download Model

```typescript
async function downloadModel(url: string, modelId: string) {
  const dataFolder = await invoke<string>('get_jan_data_folder_path')
  const destination = `${dataFolder}/models/${modelId}/model.gguf`

  // Listen for progress
  const progressUnlisten = await listen<{ progress: number }>(
    'download_progress',
    (event) => {
      setProgress(event.payload.progress * 100)
    }
  )

  // Listen for completion
  const completeUnlisten = await listen<{ path: string }>(
    'download_complete',
    (event) => {
      console.log('Downloaded to:', event.payload.path)
      setDownloaded(true)
    }
  )

  // Start download
  try {
    await invoke('download_files', { url, destination })
  } catch (error) {
    console.error('Download failed:', error)
  } finally {
    progressUnlisten()
    completeUnlisten()
  }
}
```

---

## Error Handling

All Rust commands return `Result<T, String>`. Handle errors:

```typescript
try {
  await invoke('my_command', { params })
} catch (error) {
  // error is a string from Rust
  console.error('Command failed:', error)
  toast.error(`Error: ${error}`)
}
```

---

## Best Practices

1. **Type safety:** Always type invoke results
   ```typescript
   const result = await invoke<MyType>('command')
   ```

2. **Error handling:** Wrap invoke in try-catch
   ```typescript
   try {
     await invoke('command')
   } catch (e) {
     handleError(e)
   }
   ```

3. **Cleanup listeners:** Always unlisten in cleanup
   ```typescript
   useEffect(() => {
     const unlisten = listen('event', handler)
     return () => { unlisten.then(fn => fn()) }
   }, [])
   ```

4. **Use services:** Prefer service layer over direct invoke
   ```typescript
   // Good
   await threadService.deleteThread(id)

   // Less good
   await invoke('delete_thread', { id })
   ```

5. **Validate inputs:** Check data before sending to backend
   ```typescript
   if (!isValidThreadId(id)) {
     throw new Error('Invalid thread ID')
   }
   await invoke('delete_thread', { id })
   ```

---

## API Versioning

⚠️ **Note:** Jan AI doesn't currently version its internal APIs

**Future consideration:** Semantic versioning for extensions

---

## Next Steps

- **Security:** [SECURITY_GUIDE.md](./SECURITY_GUIDE.md)
- **Testing:** [TESTING_GUIDE.md](./TESTING_GUIDE.md)
- **How-To:** [HOW_TO_GUIDE.md](./HOW_TO_GUIDE.md)

**Questions?** Return to the [Learning Hub](./README.md)
