# Security Guide

**Security best practices for Jan AI development**

**Documented:** November 18, 2025

---

## Security Philosophy

Jan AI runs **locally** on users' machines. Security priorities:

1. **Prevent unauthorized file access** - Protect user data
2. **Validate all inputs** - Commands, paths, user data
3. **Secure IPC boundaries** - Frontend ↔ Backend communication
4. **Dependency safety** - Keep packages updated
5. **No credential leaks** - API keys, tokens

---

## Tauri Security Model

### Capability-Based Access Control

**Configuration:** [src-tauri/tauri.conf.json](../../src-tauri/tauri.conf.json#L41-L47)

```json
{
  "security": {
    "capabilities": [
      "default",
      "log-app-window",
      "logs-window",
      "system-monitor-window"
    ]
  }
}
```

**What are capabilities?**
- Permissions that grant access to specific Tauri features
- Defined in `capabilities/` directory
- Windows can have different capabilities

**Example capability file:**
```json
{
  "identifier": "default",
  "description": "Default permissions",
  "windows": ["main"],
  "permissions": [
    "fs:read",
    "fs:write",
    "dialog:open",
    "shell:execute"
  ]
}
```

**Best practice:** Only grant necessary permissions

---

## Content Security Policy (CSP)

✅ **CURRENT:** Configured in tauri.conf.json

**Configuration:** [src-tauri/tauri.conf.json](../../src-tauri/tauri.conf.json#L48-L57)

```json
{
  "csp": {
    "default-src": "'self' customprotocol: asset:",
    "connect-src": "'self' ipc: https: http:",
    "img-src": "'self' blob: data: https:",
    "style-src": "'unsafe-inline' 'self'",
    "script-src": "'self' asset:"
  }
}
```

### What CSP Prevents

- **XSS attacks** - Blocks inline scripts from untrusted sources
- **Data injection** - Restricts where resources can load from
- **Clickjacking** - Prevents embedding in iframes

### CSP Directives Explained

| Directive | What It Does | Jan's Setting |
|-----------|--------------|---------------|
| `default-src` | Fallback for all resource types | `'self'` + protocols |
| `connect-src` | Where fetch/WebSocket can connect | `'self'` + HTTPS |
| `img-src` | Where images can load from | `'self'` + data URLs |
| `style-src` | CSS sources | `'self'` + inline (⚠️) |
| `script-src` | JavaScript sources | `'self'` only ✅ |

⚠️ **Note:** `'unsafe-inline'` in `style-src` is necessary for TailwindCSS but poses minor risk

---

## Path Validation

### ⚠️ CRITICAL: Prevent Path Traversal

**Vulnerable code:**
```rust
// ❌ DANGEROUS - User can pass "../../../etc/passwd"
#[tauri::command]
fn read_file(path: String) -> Result<String, String> {
    std::fs::read_to_string(path)
        .map_err(|e| e.to_string())
}
```

**Secure code:**
```rust
// ✅ SAFE - Validates path is within allowed directory
#[tauri::command]
fn read_file(app: AppHandle, relative_path: String) -> Result<String, String> {
    let data_folder = get_jan_data_folder_path(app);
    let full_path = data_folder.join(&relative_path);

    // Ensure path is within data_folder
    if !full_path.starts_with(&data_folder) {
        return Err("Invalid path: traversal detected".to_string());
    }

    // Ensure path is canonical (no symlinks escape)
    let canonical = full_path.canonicalize()
        .map_err(|_| "Invalid path")?;

    if !canonical.starts_with(&data_folder) {
        return Err("Invalid path: outside allowed directory".to_string());
    }

    std::fs::read_to_string(canonical)
        .map_err(|e| e.to_string())
}
```

**Real example:** [commands.rs:169-172](../../src-tauri/src/core/app/commands.rs#L169-L172)
```rust
// Check if this is a parent directory to avoid infinite recursion
if new_data_folder_path.starts_with(&current_data_folder) {
    return Err(
        "New data folder cannot be a subdirectory of the current data folder".to_string(),
    );
}
```

### Path Validation Checklist

- [ ] Use `PathBuf` instead of `String` for paths
- [ ] Validate paths with `.starts_with()` or `.is_relative_to()`
- [ ] Canonicalize paths with `.canonicalize()` to resolve symlinks
- [ ] Never trust user-provided absolute paths
- [ ] Use `join()` instead of string concatenation

---

## Input Validation

### TypeScript Validation

```typescript
// ❌ BAD - No validation
async function saveSettings(name: string, email: string) {
  await invoke('save_settings', { name, email })
}

// ✅ GOOD - Validate before sending to backend
async function saveSettings(name: string, email: string) {
  // Length validation
  if (name.trim().length < 2) {
    throw new Error('Name must be at least 2 characters')
  }

  // Email format validation
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!emailRegex.test(email)) {
    throw new Error('Invalid email format')
  }

  // Sanitize HTML to prevent XSS
  const sanitizedName = name.replace(/[<>]/g, '')

  await invoke('save_settings', {
    name: sanitizedName,
    email: email.trim()
  })
}
```

### Rust Validation

```rust
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
pub struct SettingsInput {
    pub name: String,
    pub email: String,
}

#[tauri::command]
fn save_settings(input: SettingsInput) -> Result<(), String> {
    // Length validation
    if input.name.trim().len() < 2 {
        return Err("Name must be at least 2 characters".to_string());
    }

    // Email validation (simple check)
    if !input.email.contains('@') || !input.email.contains('.') {
        return Err("Invalid email format".to_string());
    }

    // Sanitize special characters
    let sanitized_name = input.name
        .chars()
        .filter(|c| c.is_alphanumeric() || c.is_whitespace())
        .collect::<String>();

    // Proceed with saving...
    Ok(())
}
```

### Validation Libraries

**TypeScript:**
```typescript
// Using Zod for runtime validation
import { z } from 'zod'

const SettingsSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
})

async function saveSettings(input: unknown) {
  // Throws if validation fails
  const validated = SettingsSchema.parse(input)

  await invoke('save_settings', validated)
}
```

**Rust:**
```rust
// Using validator crate
use validator::Validate;

#[derive(Deserialize, Validate)]
pub struct SettingsInput {
    #[validate(length(min = 2, max = 100))]
    pub name: String,

    #[validate(email)]
    pub email: String,
}

#[tauri::command]
fn save_settings(input: SettingsInput) -> Result<(), String> {
    input.validate()
        .map_err(|e| format!("Validation error: {}", e))?;

    // Safe to use...
    Ok(())
}
```

---

## XSS Prevention

### Frontend Protection

**React automatically escapes text content:**
```typescript
// ✅ SAFE - React escapes by default
function UserMessage({ content }: { content: string }) {
  return <div>{content}</div>
}

// Even if content = "<script>alert('xss')</script>"
// React renders it as text, not HTML
```

**Dangerous patterns:**
```typescript
// ❌ DANGEROUS - dangerouslySetInnerHTML
function UserMessage({ content }: { content: string }) {
  return <div dangerouslySetInnerHTML={{ __html: content }} />
}

// ❌ DANGEROUS - Setting innerHTML
function renderMessage(content: string) {
  document.getElementById('message')!.innerHTML = content
}
```

**If you MUST render HTML:**
```typescript
import DOMPurify from 'dompurify'

function SafeHTML({ content }: { content: string }) {
  // Sanitize before rendering
  const clean = DOMPurify.sanitize(content, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
    ALLOWED_ATTR: ['href'],
  })

  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

### Markdown Rendering

Jan uses markdown for chat messages. Ensure your markdown library is safe:

```typescript
// Using react-markdown (safe by default)
import ReactMarkdown from 'react-markdown'

function ChatMessage({ content }: { content: string }) {
  return (
    <ReactMarkdown
      // Disallow raw HTML in markdown
      disallowedElements={['script', 'iframe', 'object']}
      unwrapDisallowed
    >
      {content}
    </ReactMarkdown>
  )
}
```

---

## Secure IPC Communication

### Command Pattern

**Commands** = Frontend → Backend

**Security rules:**
1. ✅ **Always validate inputs** in Rust
2. ✅ **Use typed structs** instead of primitives
3. ✅ **Return `Result<T, String>`** to handle errors
4. ❌ **Never trust frontend data**

**Example:**
```rust
// Good command structure
#[derive(Deserialize)]
pub struct DeleteThreadInput {
    thread_id: String,
}

#[tauri::command]
pub fn delete_thread(
    app: AppHandle,
    input: DeleteThreadInput
) -> Result<(), String> {
    // Validate thread_id format (UUIDs only)
    if !is_valid_uuid(&input.thread_id) {
        return Err("Invalid thread ID format".to_string());
    }

    // Validate thread exists and belongs to user
    let data_folder = get_jan_data_folder_path(app);
    let thread_path = data_folder.join("threads").join(&input.thread_id);

    if !thread_path.exists() {
        return Err("Thread not found".to_string());
    }

    // Safe to delete
    std::fs::remove_file(thread_path)
        .map_err(|e| e.to_string())
}
```

### Event Pattern

**Events** = Backend → Frontend

**Security rules:**
1. ✅ **Only emit to intended windows**
2. ✅ **Avoid sending sensitive data**
3. ✅ **Use typed payloads**

**Example:**
```rust
use serde::Serialize;

#[derive(Serialize)]
struct ProgressEvent {
    operation_id: String,
    progress: f64, // 0.0 to 1.0
    // ❌ Don't include: file_path, user_data, tokens
}

#[tauri::command]
async fn download_model(app: AppHandle, model_id: String) -> Result<(), String> {
    // ... download logic ...

    // Emit progress to all listeners
    app.emit("download_progress", ProgressEvent {
        operation_id: model_id.clone(),
        progress: 0.5,
    }).map_err(|e| e.to_string())?;

    Ok(())
}
```

**Frontend listening:**
```typescript
import { listen } from '@tauri-apps/api/event'

useEffect(() => {
  const unlisten = listen<ProgressEvent>('download_progress', (event) => {
    console.log('Progress:', event.payload.progress)
  })

  return () => {
    unlisten.then((fn) => fn())
  }
}, [])
```

---

## Environment Variables

### Secure Handling

```rust
// ❌ BAD - Exposes sensitive env vars to frontend
#[tauri::command]
fn get_env(key: String) -> Option<String> {
    std::env::var(key).ok()
}

// ✅ GOOD - Whitelist allowed env vars
#[tauri::command]
fn get_allowed_env(key: String) -> Result<String, String> {
    match key.as_str() {
        "NODE_ENV" | "APP_VERSION" => {
            std::env::var(key).map_err(|_| "Not found".to_string())
        }
        _ => Err("Access denied".to_string()),
    }
}
```

### .env Files

**Never commit:**
- `.env`
- `.env.local`
- `.env.production`
- API keys, tokens, secrets

**Use .env.example:**
```bash
# .env.example (safe to commit)
OPENAI_API_KEY=your_key_here
ANTHROPIC_API_KEY=your_key_here
```

```bash
# .env (gitignored)
OPENAI_API_KEY=sk-real-key-here
ANTHROPIC_API_KEY=sk-ant-real-key-here
```

**Load in Rust:**
```rust
use std::env;

fn get_api_key() -> Result<String, String> {
    env::var("OPENAI_API_KEY")
        .map_err(|_| "API key not configured".to_string())
}
```

---

## Dependency Security

### Audit Regularly

**Node.js:**
```bash
# Check for vulnerabilities
yarn audit

# Fix auto-fixable issues
yarn audit fix

# Interactive upgrade
yarn upgrade-interactive
```

**Rust:**
```bash
# Install cargo-audit
cargo install cargo-audit

# Check for vulnerabilities
cd src-tauri
cargo audit

# Update dependencies
cargo update
```

### Pinning Versions

**package.json:**
```json
{
  "dependencies": {
    "react": "19.0.0",  // ✅ Exact version
    "zustand": "^5.0.3" // ⚠️ Caret allows minor updates
  }
}
```

**Cargo.toml:**
```toml
[dependencies]
serde = "1.0"       # ⚠️ Allows patch updates
tauri = "=2.8.5"    # ✅ Exact version
```

**Best practice:** Use exact versions for critical deps, allow patches for others

---

## Common Vulnerabilities

### 1. Command Injection

**Vulnerable:**
```rust
use std::process::Command;

// ❌ DANGEROUS - User can inject commands
#[tauri::command]
fn run_command(cmd: String) -> Result<String, String> {
    let output = Command::new("sh")
        .arg("-c")
        .arg(cmd) // User could pass: "ls; rm -rf /"
        .output()
        .map_err(|e| e.to_string())?;

    Ok(String::from_utf8_lossy(&output.stdout).to_string())
}
```

**Secure:**
```rust
// ✅ SAFE - Whitelist allowed commands
#[tauri::command]
fn run_allowed_command(command: String) -> Result<String, String> {
    let allowed = vec!["git status", "git diff"];

    if !allowed.contains(&command.as_str()) {
        return Err("Command not allowed".to_string());
    }

    // Parse and validate each part
    let parts: Vec<&str> = command.split_whitespace().collect();

    let output = Command::new(parts[0])
        .args(&parts[1..])
        .output()
        .map_err(|e| e.to_string())?;

    Ok(String::from_utf8_lossy(&output.stdout).to_string())
}
```

### 2. SQL Injection

Jan doesn't use SQL, but if you add a database:

```rust
// ❌ DANGEROUS - String interpolation
let query = format!("SELECT * FROM users WHERE name = '{}'", user_input);

// ✅ SAFE - Prepared statements
let stmt = conn.prepare("SELECT * FROM users WHERE name = ?1")?;
let rows = stmt.query_map([user_input], |row| {
    // ...
})?;
```

### 3. Insecure Deserialization

```rust
// ❌ DANGEROUS - Deserializing untrusted data without validation
#[tauri::command]
fn load_config(json: String) -> Result<(), String> {
    let config: Config = serde_json::from_str(&json)
        .map_err(|e| e.to_string())?;
    // No validation - config could contain malicious data
    save_config(config);
    Ok(())
}

// ✅ SAFE - Validate after deserializing
#[tauri::command]
fn load_config(json: String) -> Result<(), String> {
    let config: Config = serde_json::from_str(&json)
        .map_err(|e| e.to_string())?;

    // Validate fields
    if config.max_threads > 100 {
        return Err("Invalid config: max_threads too high".to_string());
    }

    if !config.data_folder.is_absolute() {
        return Err("Invalid config: data_folder must be absolute".to_string());
    }

    save_config(config);
    Ok(())
}
```

### 4. Unvalidated Redirects

```typescript
// ❌ DANGEROUS - Open redirect vulnerability
function redirect(url: string) {
  window.location.href = url // Could be https://evil.com
}

// ✅ SAFE - Whitelist allowed domains
function redirect(url: string) {
  const allowed = ['jan.ai', 'docs.jan.ai']
  const urlObj = new URL(url)

  if (!allowed.includes(urlObj.hostname)) {
    throw new Error('Invalid redirect URL')
  }

  window.location.href = url
}
```

---

## API Key Management

### Frontend Storage

**❌ Never store API keys in:**
- LocalStorage
- SessionStorage
- Cookies
- Frontend code

**✅ Store in:**
- Rust backend (encrypted file or OS keychain)

### Keychain Integration

**Tauri plugin:** `tauri-plugin-keychain`

```rust
use tauri_plugin_keychain::Keychain;

#[tauri::command]
fn save_api_key(keychain: State<Keychain>, key: String) -> Result<(), String> {
    keychain.set("openai_api_key", &key)
        .map_err(|e| e.to_string())
}

#[tauri::command]
fn get_api_key(keychain: State<Keychain>) -> Result<String, String> {
    keychain.get("openai_api_key")
        .map_err(|e| e.to_string())
}
```

**Frontend usage:**
```typescript
// Save key (user enters once)
await invoke('save_api_key', { key: userInputKey })

// Retrieve when needed (backend only)
const key = await invoke('get_api_key')
```

---

## File Upload Security

### Validate File Types

```rust
use std::path::Path;

#[tauri::command]
fn upload_file(file_path: String) -> Result<(), String> {
    let path = Path::new(&file_path);

    // Validate extension
    let allowed_extensions = ["json", "txt", "md"];
    let extension = path.extension()
        .and_then(|e| e.to_str())
        .ok_or("No file extension")?;

    if !allowed_extensions.contains(&extension) {
        return Err("Invalid file type".to_string());
    }

    // Validate file size (e.g., max 10MB)
    let metadata = std::fs::metadata(path)
        .map_err(|_| "Cannot read file")?;

    if metadata.len() > 10 * 1024 * 1024 {
        return Err("File too large".to_string());
    }

    // Validate MIME type (read first bytes)
    // ...

    Ok(())
}
```

---

## HTTPS/TLS

### API Server

Jan runs a local API server at `http://localhost:1337`

**When to use HTTPS:**
- ❌ Not needed for `localhost` (already secure)
- ✅ Required if exposing to network

**Enabling HTTPS:**
```rust
// Using rustls for TLS
use axum_server::tls_rustls::RustlsConfig;

let config = RustlsConfig::from_pem_file(
    "cert.pem",
    "key.pem"
).await?;

axum_server::bind_rustls("0.0.0.0:1337", config)
    .serve(app.into_make_service())
    .await?;
```

---

## Security Checklist

**Before merging a PR:**

- [ ] All user inputs validated
- [ ] No hardcoded secrets or API keys
- [ ] File paths validated (no traversal)
- [ ] CSP configured correctly
- [ ] No `dangerouslySetInnerHTML` without sanitization
- [ ] Dependencies audited (`yarn audit`, `cargo audit`)
- [ ] Commands use typed structs, not primitives
- [ ] Error messages don't leak sensitive info
- [ ] Tests include security edge cases
- [ ] No `eval()` or `Function()` constructor
- [ ] HTTPS used for external API calls

---

## Reporting Security Issues

**Found a vulnerability?**

1. **Do NOT create a public issue**
2. Email: security@jan.ai (or check SECURITY.md)
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Impact assessment
   - Suggested fix (if any)

**Response time:** 48 hours acknowledgment

---

## Resources

**Tauri Security:**
- https://tauri.app/v2/security/
- https://tauri.app/v2/reference/config/#security

**OWASP Top 10:**
- https://owasp.org/www-project-top-ten/

**Rust Security:**
- https://rust-lang.github.io/api-guidelines/
- https://anssi-fr.github.io/rust-guide/

**Dependency Auditing:**
- `cargo audit` - https://github.com/rustsec/rustsec
- `yarn audit` - Built into Yarn

---

## Next Steps

- **API Docs:** [API_DOCUMENTATION.md](./API_DOCUMENTATION.md)
- **Testing:** [TESTING_GUIDE.md](./TESTING_GUIDE.md)
- **Contributing:** [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)

**Questions?** Return to the [Learning Hub](./README.md)
