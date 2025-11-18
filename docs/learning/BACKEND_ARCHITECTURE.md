# Backend Architecture

**Deep dive into the Rust/Tauri backend**

**Documented:** November 18, 2025 | **Stack:** Rust 1.77.2, Tauri 2.8.5, Tokio

---

## Overview

Jan AI's backend is built with Rust and Tauri 2, providing:
- Native performance for LLM inference
- Secure IPC between frontend and backend
- Cross-platform support (Desktop + Mobile)
- Custom plugins for specialized functionality

⚠️ **Note:** Project uses Rust 1.77.2, but latest is 1.91.1 (Nov 2025)

---

## Rust for JavaScript Developers

### Key Concepts

| Rust Concept | JS/TS Equivalent | Explanation |
|--------------|------------------|-------------|
| `Result<T, E>` | `Promise<T>` | Success or error type |
| `Option<T>` | `T \| null` | Value or none |
| `&str` | `string` | String reference |
| `String` | `string` | Owned string |
| `Vec<T>` | `Array<T>` | Dynamic array |
| `async fn` | `async function` | Async function |
| `.await` | `await` | Wait for async |

### Ownership (Key Difference)

```rust
// Rust: ownership moves
let s1 = String::from("hello");
let s2 = s1;  // s1 is now invalid!
// println!("{}", s1); // ❌ Error!

// Use references instead
let s3 = &s2; // Borrow s2
println!("{}", s3); // ✅ OK
```

🌉 **Bridge:** Unlike JS where variables are always copied/referenced, Rust has strict ownership rules to prevent memory issues.

---

## Tauri Architecture

### Entry Point

**File:** [src-tauri/src/lib.rs](../../src-tauri/src/lib.rs)

```rust
pub fn run() {
    tauri::Builder::default()
        // Register plugins
        .plugin(tauri_plugin_llamacpp::init())
        .plugin(tauri_plugin_vector_db::init())
        .plugin(tauri_plugin_rag::init())
        // Register commands
        .invoke_handler(tauri::generate_handler![
            core::app::commands::get_app_configurations,
            core::filesystem::commands::read_file_sync,
            // ... all callable commands
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

💡 **Aha Moment:** Only commands listed in `generate_handler![]` can be called from frontend. This is the security whitelist.

---

## Command Pattern

### Anatomy of a Tauri Command

**File:** [src-tauri/src/core/app/commands.rs](../../src-tauri/src/core/app/commands.rs)

```rust
#[tauri::command]
pub fn get_app_configurations<R: Runtime>(
    app_handle: tauri::AppHandle<R>
) -> AppConfiguration {
    // 1. Read config file
    let config_file = get_configuration_file_path(app_handle);

    // 2. Parse JSON
    let content = fs::read_to_string(&config_file)?;
    let config: AppConfiguration = serde_json::from_str(&content)?;

    // 3. Return
    config
}
```

**Called from Frontend:**
```typescript
const config = await invoke('get_app_configurations')
```

### Error Handling Pattern

```rust
#[tauri::command]
pub fn risky_operation() -> Result<String, String> {
    // Use ? operator for error propagation
    let data = read_file(path).map_err(|e| e.to_string())?;
    Ok(data)
}
```

Frontend receives:
- Success: `{ data: "..." }`
- Error: Throws exception with error message

---

## Module Structure

**Location:** [src-tauri/src/core/](../../src-tauri/src/core/)

```
core/
├── mod.rs              # Re-exports all modules
├── state.rs            # Global app state
├── setup.rs            # App initialization
├── app/                # App configuration
│   ├── mod.rs
│   ├── commands.rs     # Tauri commands
│   ├── models.rs       # Data structures
│   └── helpers.rs      # Utility functions
├── downloads/          # Download management
├── extensions/         # Extension loading
├── filesystem/         # File operations
├── mcp/                # MCP integration
├── server/             # HTTP API server
├── system/             # System operations
└── threads/            # Thread management
```

### Module Pattern

Each module follows this structure:
- `mod.rs` - Public API, re-exports
- `commands.rs` - Tauri commands
- `models.rs` - Data structures (with Serde)
- `helpers.rs` - Internal functions

---

## Tauri Plugins

**Location:** [src-tauri/plugins/](../../src-tauri/plugins/)

### Custom Plugins

| Plugin | Purpose |
|--------|---------|
| **tauri-plugin-llamacpp** | LLM inference engine |
| **tauri-plugin-vector-db** | Vector database operations |
| **tauri-plugin-rag** | RAG functionality |
| **tauri-plugin-hardware** | System hardware info |

### Plugin Registration

```rust
// lib.rs
.plugin(tauri_plugin_llamacpp::init())
```

### Plugin Structure

```
tauri-plugin-llamacpp/
├── src/
│   └── lib.rs         # Plugin implementation
├── Cargo.toml         # Dependencies
└── build.rs           # Build script (if needed)
```

---

## Event System

### Emitting Events (Rust → Frontend)

```rust
use tauri::Emitter;

#[tauri::command]
async fn download_model(app: AppHandle, url: String) -> Result<(), String> {
    // Start download
    loop {
        let progress = calculate_progress();

        // Emit progress event
        app.emit("download_progress", ProgressPayload {
            progress,
            speed: 1024,
        })?;

        if complete { break; }
    }

    // Emit completion
    app.emit("download_complete", ())?;
    Ok(())
}
```

Frontend listens:
```typescript
await listen('download_progress', (event) => {
  console.log('Progress:', event.payload.progress)
})
```

---

## Async/Await in Rust

```rust
use tokio; // Async runtime

#[tauri::command]
async fn fetch_data(url: String) -> Result<String, String> {
    // Tokio's async HTTP client
    let response = reqwest::get(&url)
        .await
        .map_err(|e| e.to_string())?;

    let body = response.text()
        .await
        .map_err(|e| e.to_string())?;

    Ok(body)
}
```

🌉 **Bridge:** Very similar to JavaScript async/await!

---

## State Management

### App State

**File:** [src-tauri/src/core/state.rs](../../src-tauri/src/core/state.rs)

```rust
pub struct AppState {
    pub download_manager: Arc<Mutex<DownloadManager>>,
    pub mcp_settings: Arc<Mutex<McpSettings>>,
    // ... other shared state
}
```

### Using State in Commands

```rust
#[tauri::command]
async fn get_downloads(state: State<'_, AppState>) -> Result<Vec<Download>, String> {
    let manager = state.download_manager.lock().await;
    Ok(manager.get_all())
}
```

💡 **Aha Moment:** `State` is like dependency injection - Tauri provides it automatically!

---

## HTTP API Server

**Location:** [src-tauri/src/core/server/](../../src-tauri/src/core/server/)

**Purpose:** OpenAI-compatible API at `localhost:1337`

```rust
use hyper::{Server, service::make_service_fn};

async fn start_server() {
    let make_svc = make_service_fn(|_conn| async {
        Ok::<_, Infallible>(service_fn(handle_request))
    });

    let addr = ([127, 0, 0, 1], 1337).into();
    let server = Server::bind(&addr).serve(make_svc);

    server.await.unwrap();
}

async fn handle_request(req: Request<Body>) -> Result<Response<Body>, Error> {
    match req.uri().path() {
        "/v1/chat/completions" => handle_chat(req).await,
        "/v1/models" => handle_models(req).await,
        _ => Ok(Response::builder().status(404).body(Body::empty()).unwrap())
    }
}
```

---

## File System Operations

**File:** [src-tauri/src/core/filesystem/commands.rs](../../src-tauri/src/core/filesystem/commands.rs)

```rust
#[tauri::command]
pub fn read_file_sync(path: String) -> Result<String, String> {
    fs::read_to_string(path).map_err(|e| e.to_string())
}

#[tauri::command]
pub fn write_file_sync(path: String, data: String) -> Result<(), String> {
    fs::write(path, data).map_err(|e| e.to_string())
}
```

⚠️ **Security:** Tauri validates paths to prevent directory traversal attacks

---

## MCP Integration

**Location:** [src-tauri/src/core/mcp/](../../src-tauri/src/core/mcp/)

**Crate:** `rmcp = "0.8.5"` (Rust MCP client)

```rust
use rmcp::Client;

#[tauri::command]
async fn call_mcp_tool(
    server_id: String,
    tool_name: String,
    args: Value
) -> Result<Value, String> {
    let client = get_mcp_client(&server_id)?;
    let result = client.call_tool(&tool_name, args).await?;
    Ok(result)
}
```

---

## Error Handling Best Practices

### Result Pattern

```rust
// ✅ Good: Propagate errors with ?
fn load_config() -> Result<Config, String> {
    let file = fs::read_to_string("config.json")?;
    let config = serde_json::from_str(&file)?;
    Ok(config)
}

// ❌ Bad: Unwrap (panics on error!)
fn bad_load_config() -> Config {
    let file = fs::read_to_string("config.json").unwrap(); // Crash!
    serde_json::from_str(&file).unwrap()
}
```

### Logging

```rust
use log::{info, error, warn};

#[tauri::command]
fn do_something() -> Result<(), String> {
    info!("Starting operation");

    match risky_operation() {
        Ok(result) => {
            info!("Success: {:?}", result);
            Ok(())
        }
        Err(e) => {
            error!("Failed: {}", e);
            Err(e.to_string())
        }
    }
}
```

---

## Dependencies (Cargo.toml)

**Key Crates:**

```toml
[dependencies]
tauri = "2.8.5"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
tokio = { version = "1", features = ["full"] }
reqwest = "0.11"
log = "0.4"
rmcp = "0.8.5"  # MCP client
```

🌉 **Bridge:** Cargo.toml is like package.json

---

## Next Steps

- **Extension Development:** [Integration Guide](./INTEGRATION_GUIDE.md)
- **API Reference:** [API Documentation](./API_DOCUMENTATION.md)
- **Security:** [Security Guide](./SECURITY_GUIDE.md)

**Questions?** Return to the [Learning Hub](./README.md)
