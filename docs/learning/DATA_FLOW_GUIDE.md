# Data Flow Guide

**How data moves through the Jan AI system**

**Documented:** November 18, 2025

---

## Core Communication Patterns

### Pattern 1: Commands (Frontend → Backend)

**Use**: One-time requests/responses

```typescript
// Frontend calls Tauri command
const result = await invoke('get_app_configurations')
```

```rust
// Backend handles command
#[tauri::command]
async fn get_app_configurations() -> Result<Config, String> {
    // Process and return
}
```

---

### Pattern 2: Events (Backend → Frontend)

**Use**: Real-time updates, streaming

```rust
// Backend emits event
app.emit("download_progress", ProgressPayload {
    id,
    progress: 45.0
})?;
```

```typescript
// Frontend listens
await listen('download_progress', (event) => {
    console.log(event.payload.progress)
})
```

---

## Example Flows

### Flow 1: Sending a Chat Message

```
1. User types message in ChatInput component
   [web-app/src/routes/threads/$threadId.tsx]

2. Component calls threadService.sendMessage()
   [web-app/src/services/threads/]

3. Service invokes Tauri command
   invoke('send_message', { threadId, content })

4. Rust receives command
   [src-tauri/src/core/threads/commands.rs]

5. Calls llama.cpp plugin
   [src-tauri/plugins/tauri-plugin-llamacpp/]

6. LLM generates response (streaming)

7. Emits events: "message_chunk", "message_complete"

8. Frontend event listener receives chunks
   [web-app/src/services/events/]

9. Updates Zustand store
   [web-app/src/services/threads/store.ts]

10. React re-renders with new message
```

---

### Flow 2: Downloading a Model

```
1. User clicks Download in Hub
   [web-app/src/routes/hub/$modelId.tsx]

2. Calls downloadService.downloadModel()

3. Invokes 'download_model' command

4. Rust download manager starts
   [src-tauri/src/core/downloads/]

5. Emits progress events every 100ms
   emit('download_progress', { id, progress, speed })

6. Frontend updates progress bar in real-time

7. On complete: emit('download_complete')

8. Frontend marks model as downloaded
```

---

### Flow 3: Changing a Setting

```
1. User toggles setting in Settings page
   [web-app/src/routes/settings/general.tsx]

2. Calls settingsService.updateSetting()

3. Invokes 'update_app_configuration'

4. Rust updates config file
   [src-tauri/src/core/app/commands.rs]

5. Returns success

6. Frontend updates Zustand store

7. UI reflects new setting immediately
```

---

## State Management Flow

### Zustand Store Updates

```typescript
// 1. Action triggered
store.addThread(newThread)

// 2. Store updated (synchronous)
set((state) => ({
    threads: [...state.threads, newThread]
}))

// 3. React components re-render
// Any component using useThreads() gets new state
```

---

## Service Layer Pattern

```
Component
  ↓
Service (abstraction)
  ├─ Tauri implementation (desktop)
  ├─ Web implementation (browser)
  └─ Default implementation (fallback)
```

Example:
```typescript
// Service picks implementation based on platform
export const appService = isTauri()
    ? tauriAppService
    : webAppService
```

---

## Event System

### Frontend Event Listeners

```typescript
// Set up listener
const unlisten = await listen('model_loaded', (event) => {
    console.log('Model ready:', event.payload)
})

// Clean up
unlisten()
```

### Backend Event Emission

```rust
app.emit("model_loaded", ModelPayload { id, name })?;
```

---

## Extension Loading Flow

```
1. App starts
2. Extension service scans directories
3. Loads extension metadata
4. Calls extension.onLoad()
5. Extension registers capabilities
6. Extension ready
```

---

## Next Steps

- **Deep Dives**: [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) | [Backend Architecture](./BACKEND_ARCHITECTURE.md)
- **Code Tours**: [Guided walkthroughs](./CODE_TOURS.md)

**Questions?** Return to the [Learning Hub](./README.md)
