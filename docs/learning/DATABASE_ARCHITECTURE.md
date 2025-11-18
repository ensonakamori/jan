# Database Architecture

**Local data persistence and storage**

**Documented:** November 18, 2025

---

## Overview

Jan AI uses **file-based storage** for data persistence. No traditional database server is used.

🧠 **Mental Model:**
```
Database = Organized JSON files on disk
Tables = Directories
Rows = Individual JSON files
```

---

## Storage Location

### Data Folder

**Default Paths:**
- **macOS:** `~/jan`
- **Windows:** `%USERPROFILE%\jan`
- **Linux:** `~/jan`

**Configurable:** Users can change via Settings → General → Data Folder

**Structure:**
```
~/jan/
├── models/              # Downloaded models
│   ├── llama-2-7b/
│   │   ├── model.gguf
│   │   └── metadata.json
│   └── mistral-7b/
├── threads/             # Chat threads
│   ├── thread-1.json
│   ├── thread-2.json
│   └── thread-3.json
├── settings/            # User settings
│   └── settings.json
├── extensions/          # Installed extensions
└── logs/                # Application logs
```

---

## Data Models

### Thread (Chat) Schema

**File:** `~/jan/threads/{threadId}.json`

```json
{
  "id": "thread-abc123",
  "title": "My Chat",
  "assistantId": "assistant-1",
  "createdAt": 1700000000000,
  "updatedAt": 1700000000000,
  "metadata": {},
  "messages": [
    {
      "id": "msg-1",
      "role": "user",
      "content": "Hello",
      "createdAt": 1700000000000
    },
    {
      "id": "msg-2",
      "role": "assistant",
      "content": "Hi there!",
      "createdAt": 1700000001000
    }
  ]
}
```

**TypeScript Interface:** [core/src/types/thread.ts](../../core/src/types/thread.ts)

---

### Model Metadata

**File:** `~/jan/models/{modelId}/metadata.json`

```json
{
  "id": "llama-2-7b",
  "name": "Llama 2 7B",
  "version": "1.0",
  "format": "gguf",
  "size": 3800000000,
  "parameters": {
    "context_length": 4096,
    "temperature": 0.7
  },
  "downloadedAt": 1700000000000
}
```

---

### Settings

**File:** `~/jan/settings/settings.json`

```json
{
  "theme": "dark",
  "language": "en",
  "dataFolder": "~/jan",
  "apiServer": {
    "enabled": true,
    "port": 1337
  }
}
```

---

## File Operations

### Read Thread

**Rust:**
```rust
#[tauri::command]
pub fn get_thread(thread_id: String) -> Result<Thread, String> {
    let path = get_thread_path(&thread_id);
    let content = fs::read_to_string(path)
        .map_err(|e| e.to_string())?;
    let thread: Thread = serde_json::from_str(&content)
        .map_err(|e| e.to_string())?;
    Ok(thread)
}
```

### Write Thread

**Rust:**
```rust
#[tauri::command]
pub fn save_thread(thread: Thread) -> Result<(), String> {
    let path = get_thread_path(&thread.id);
    let json = serde_json::to_string_pretty(&thread)
        .map_err(|e| e.to_string())?;
    fs::write(path, json)
        .map_err(|e| e.to_string())
}
```

---

## Vector Database (RAG)

**Purpose:** Semantic search for RAG (Retrieval-Augmented Generation)

**Implementation:** Tauri plugin: `tauri-plugin-vector-db`

**Storage:** Embedded vector database (not cloud)

**Operations:**
- Add documents
- Semantic search
- Delete documents

**Usage:**
```rust
use tauri_plugin_vector_db::VectorDb;

// Add document
vector_db.add_document(doc_id, embedding, metadata)?;

// Search
let results = vector_db.search(query_embedding, top_k)?;
```

---

## Caching Strategy

### Model Cache

**Location:** `~/jan/models/{modelId}/`

**What's cached:**
- Downloaded model files (`.gguf`)
- Model metadata
- Tokenizer files

**Eviction:** Manual only (user deletes via UI)

### Message Cache

**Location:** In-memory during session

**Persistence:** Written to thread files on change

---

## Data Migration

⚠️ **UNCLEAR:** Migration strategy for schema changes

**To investigate:**
1. Check for migration files in `src-tauri/src/core/`
2. Look for version numbers in data files
3. Check changelog for migration notes

---

## Backup & Restore

### Manual Backup

**User action:** Copy entire `~/jan/` folder

### Programmatic Backup

⚠️ **TODO:** Not yet implemented

**Proposed approach:**
```rust
#[tauri::command]
pub async fn backup_data(dest: PathBuf) -> Result<(), String> {
    let data_folder = get_data_folder();
    copy_dir_recursive(&data_folder, &dest)?;
    Ok(())
}
```

---

## Performance Considerations

### File I/O

**Current:** Synchronous file operations

**Impact:** May block on large threads (1000+ messages)

**Potential optimization:** Async file I/O with Tokio

### JSON Parsing

**Current:** Parse entire thread file on load

**Impact:** Slow for very large threads

**Potential optimization:**
- Streaming JSON parser
- Pagination (load messages on scroll)
- SQLite for large threads

---

## Concurrency

### File Locking

⚠️ **UNCLEAR:** Current file locking strategy

**Potential issues:**
- Multiple app instances
- Simultaneous writes

**To investigate:** Check for file locking in Rust code

---

## Next Steps

- **Integration Guide:** [INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)
- **API Reference:** [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)
- **Schema Details:** [DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)

**Questions?** Return to the [Learning Hub](./README.md)
