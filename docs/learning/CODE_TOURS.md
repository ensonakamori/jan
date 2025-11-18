# Code Tours

**Guided walkthroughs through actual code flows**

**Documented:** November 18, 2025

---

## Tour 1: Sending a Chat Message

**Follow a message from UI input to LLM response**

### Entry Point: User Types Message

**File:** [web-app/src/routes/threads/$threadId.tsx](../../web-app/src/routes/threads/$threadId.tsx)

```typescript
// User types in chat input component
<ChatInput onSend={handleSendMessage} />
```

### Step 1: Component Calls Service

```typescript
const handleSendMessage = async (content: string) => {
  await threadService.sendMessage(threadId, content)
}
```

### Step 2: Service Layer

**File:** [web-app/src/services/threads/tauri.ts](../../web-app/src/services/threads/tauri.ts)

```typescript
async sendMessage(threadId: string, content: string) {
  return await invoke('send_thread_message', {
    threadId,
    message: { role: 'user', content }
  })
}
```

### Step 3: Tauri IPC

Message crosses IPC boundary → Rust backend

### Step 4: Rust Command Handler

**File:** [src-tauri/src/core/threads/commands.rs](../../src-tauri/src/core/threads/commands.rs)

```rust
#[tauri::command]
async fn send_thread_message(
    thread_id: String,
    message: Message,
    app: AppHandle
) -> Result<(), String> {
    // 1. Save message to storage
    save_message(&thread_id, &message)?;

    // 2. Call LLM plugin
    let response = llama_plugin::generate(message).await?;

    // 3. Emit streaming events
    for chunk in response.chunks() {
        app.emit("message_chunk", chunk)?;
    }

    app.emit("message_complete", ())?;
    Ok(())
}
```

### Step 5: Event Listeners

**File:** [web-app/src/services/events/tauri.ts](../../web-app/src/services/events/tauri.ts)

```typescript
await listen('message_chunk', (event) => {
  const chunk = event.payload
  // Update store with chunk
  useMessages.getState().appendChunk(chunk)
})
```

### Step 6: Store Updates

**File:** [web-app/src/services/threads/store.ts](../../web-app/src/services/threads/store.ts)

```typescript
appendChunk: (chunk) => set((state) => ({
  messages: state.messages.map((msg) =>
    msg.id === chunk.messageId
      ? { ...msg, content: msg.content + chunk.text }
      : msg
  )
}))
```

### Step 7: React Re-renders

Component using `useThreads` automatically re-renders with new content

🎯 **Key Takeaway:** One-way data flow with event-driven updates

---

## Tour 2: Downloading a Model

**Complete download flow from Hub to local storage**

### Entry Point: Hub Page

**File:** [web-app/src/routes/hub/$modelId.tsx](../../web-app/src/routes/hub/$modelId.tsx)

```typescript
const handleDownload = () => {
  downloadService.downloadModel(modelId, downloadUrl)
}
```

### Step 1: Download Service

**File:** [web-app/src/services/downloads/tauri.ts](../../web-app/src/services/downloads/tauri.ts)

```typescript
async downloadModel(id: string, url: string) {
  return await invoke('start_model_download', { id, url })
}
```

### Step 2: Rust Download Manager

**File:** [src-tauri/src/core/downloads/commands.rs](../../src-tauri/src/core/downloads/commands.rs)

```rust
#[tauri::command]
async fn start_model_download(
    id: String,
    url: String,
    state: State<'_, AppState>,
    app: AppHandle
) -> Result<(), String> {
    let download_manager = state.download_manager.lock().await;

    download_manager.start(id.clone(), url, |progress| {
        // Emit progress event
        app.emit("download_progress", DownloadProgress {
            id: id.clone(),
            progress: progress.percent,
            speed: progress.speed,
            eta: progress.eta,
        }).ok();
    }).await?;

    Ok(())
}
```

### Step 3: Progress Updates

**File:** [web-app/src/services/downloads/](../../web-app/src/services/downloads/)

```typescript
// Listen for progress events
useEffect(() => {
  listen('download_progress', (event) => {
    const { id, progress, speed } = event.payload
    useDownloads.getState().updateProgress(id, progress, speed)
  })
}, [])
```

### Step 4: UI Updates

```typescript
function DownloadProgress({ downloadId }) {
  const download = useDownloads((state) =>
    state.downloads.find((d) => d.id === downloadId)
  )

  return (
    <ProgressBar
      value={download.progress}
      label={`${download.speed} MB/s`}
    />
  )
}
```

🎯 **Key Takeaway:** Long-running operations use events for real-time updates

---

## Tour 3: Settings Persistence

**How settings are saved and loaded**

### Loading Settings on App Start

**File:** [web-app/src/providers/DataProvider.tsx](../../web-app/src/providers/DataProvider.tsx)

```typescript
useEffect(() => {
  const loadData = async () => {
    const config = await appService.getConfigurations()
    // Update stores with config
    useSettings.getState().setConfig(config)
  }
  loadData()
}, [])
```

### Tauri Command

**File:** [src-tauri/src/core/app/commands.rs](../../src-tauri/src/core/app/commands.rs)

```rust
#[tauri::command]
pub fn get_app_configurations<R: Runtime>(
    app_handle: tauri::AppHandle<R>
) -> AppConfiguration {
    let config_file = get_configuration_file_path(app_handle);

    if !config_file.exists() {
        // Create default config
        return AppConfiguration::default();
    }

    // Read and parse JSON
    let content = fs::read_to_string(&config_file).unwrap();
    serde_json::from_str(&content).unwrap()
}
```

### Updating Settings

**File:** [web-app/src/routes/settings/general.tsx](../../web-app/src/routes/settings/general.tsx)

```typescript
const handleSave = async () => {
  await appService.updateConfiguration(newConfig)
  toast.success('Settings saved')
}
```

**File:** [src-tauri/src/core/app/commands.rs](../../src-tauri/src/core/app/commands.rs)

```rust
#[tauri::command]
pub fn update_app_configuration<R: Runtime>(
    app_handle: tauri::AppHandle<R>,
    configuration: AppConfiguration,
) -> Result<(), String> {
    let config_file = get_configuration_file_path(app_handle);

    // Write JSON to file
    fs::write(
        config_file,
        serde_json::to_string(&configuration)?
    )?;

    Ok(())
}
```

🎯 **Key Takeaway:** Settings persist as JSON files in user data directory

---

## Tour 4: Extension Loading

**How extensions are discovered and initialized**

### App Startup

**File:** [web-app/src/providers/ExtensionProvider.tsx](../../web-app/src/providers/ExtensionProvider.tsx)

```typescript
useEffect(() => {
  const loadExtensions = async () => {
    const extensionPaths = await invoke('get_jan_extensions_path')
    const extensions = await discoverExtensions(extensionPaths)

    for (const ext of extensions) {
      await ext.onLoad()
    }

    setLoadedExtensions(extensions)
  }

  loadExtensions()
}, [])
```

### Rust Extension Manager

**File:** [src-tauri/src/core/extensions/commands.rs](../../src-tauri/src/core/extensions/commands.rs)

```rust
#[tauri::command]
pub fn get_active_extensions() -> Result<Vec<Extension>, String> {
    let extension_dir = get_extensions_directory();

    let mut extensions = Vec::new();

    for entry in fs::read_dir(extension_dir)? {
        let path = entry?.path();
        if path.is_dir() {
            let manifest = load_manifest(&path)?;
            extensions.push(Extension {
                id: manifest.id,
                name: manifest.name,
                version: manifest.version,
            });
        }
    }

    Ok(extensions)
}
```

### Extension Entry Point

**File:** [extensions/*/src/index.ts](../../extensions/)

```typescript
export default class MyExtension implements Extension {
  async onLoad() {
    console.log('Extension loaded')
    // Register capabilities
    this.registerModels()
    this.registerProviders()
  }

  async onUnload() {
    console.log('Extension unloading')
    // Cleanup
  }
}
```

🎯 **Key Takeaway:** Extensions are loaded dynamically from directories

---

## Tour 5: Theme Switching

**Dark/Light mode implementation**

### Theme Provider

**File:** [web-app/src/providers/ThemeProvider.tsx](../../web-app/src/providers/ThemeProvider.tsx)

```typescript
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('dark')

  useEffect(() => {
    // Apply to document
    document.documentElement.classList.toggle('dark', theme === 'dark')
  }, [theme])

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  )
}
```

### Using Theme

```typescript
function ThemeToggle() {
  const { theme, setTheme } = useTheme()

  return (
    <button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
      {theme === 'dark' ? '☀️' : '🌙'}
    </button>
  )
}
```

### TailwindCSS Dark Mode

```css
/* Automatically applies with dark: prefix */
.bg-white dark:bg-slate-900
.text-black dark:text-white
```

🎯 **Key Takeaway:** Theme is a React context with TailwindCSS dark mode

---

## Next Steps

**Practice with:**
- [Exercises](./EXERCISES.md) - Hands-on practice
- [How-To Guide](./HOW_TO_GUIDE.md) - Step-by-step tasks
- [First Contributions](./FIRST_CONTRIBUTIONS.md) - Make your first PR

**Understand architecture:**
- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md)
- [Backend Architecture](./BACKEND_ARCHITECTURE.md)

**Questions?** Return to the [Learning Hub](./README.md)
