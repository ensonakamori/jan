# Hands-On Exercises

**Practice building features with step-by-step guidance**

**Documented:** November 18, 2025

---

## Exercise 1: Add a Settings Toggle

**Difficulty:** 🟢 Beginner | **Time:** 30 minutes

### Goal

Add a new boolean setting "Show Timestamps" in General Settings

### Requirements

- Add toggle in Settings UI
- Save to app configuration
- Persist across app restarts

### Steps

1. **Add to Configuration Type**

```typescript
// core/src/types/config.ts (or similar)
export interface AppConfiguration {
  // ... existing fields
  showTimestamps?: boolean
}
```

2. **Update Rust Model**

```rust
// src-tauri/src/core/app/models.rs
#[derive(Serialize, Deserialize)]
pub struct AppConfiguration {
    // ... existing fields
    #[serde(default)]
    pub show_timestamps: bool,
}
```

3. **Add UI Toggle**

```typescript
// web-app/src/routes/settings/general.tsx
import { Switch } from '@/components/ui/switch'

function GeneralSettings() {
  const config = useSettings((state) => state.config)
  const updateConfig = useSettings((state) => state.updateConfig)

  return (
    <div>
      <label>
        Show Timestamps
        <Switch
          checked={config.showTimestamps ?? false}
          onCheckedChange={(checked) =>
            updateConfig({ ...config, showTimestamps: checked })
          }
        />
      </label>
    </div>
  )
}
```

4. **Test**

- Toggle the setting
- Restart the app
- Verify setting persisted

### Solution

Check: `web-app/src/routes/settings/general.tsx` for similar toggles

---

## Exercise 2: Create a Custom Hook

**Difficulty:** 🟡 Intermediate | **Time:** 45 minutes

### Goal

Create a `useLocalStorage` hook that syncs React state with localStorage

### Requirements

- Save state to localStorage
- Load on mount
- TypeScript support
- Handle JSON serialization

### Template

```typescript
// hooks/useLocalStorage.ts
import { useState, useEffect } from 'react'

export function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T) => void] {
  // TODO: Implement

  return [value, setValue]
}
```

### Steps

1. Initialize state from localStorage or initialValue
2. Save to localStorage whenever value changes
3. Handle JSON parse/stringify errors
4. Add TypeScript generics

### Usage Example

```typescript
function MyComponent() {
  const [name, setName] = useLocalStorage('userName', 'Guest')

  return <input value={name} onChange={(e) => setName(e.target.value)} />
}
```

### Solution

<details>
<summary>Click to reveal solution</summary>

```typescript
export function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T) => void] {
  const [value, setValue] = useState<T>(() => {
    try {
      const item = localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch {
      return initialValue
    }
  })

  useEffect(() => {
    try {
      localStorage.setItem(key, JSON.stringify(value))
    } catch (error) {
      console.error('Failed to save to localStorage:', error)
    }
  }, [key, value])

  return [value, setValue]
}
```

</details>

---

## Exercise 3: Add a Tauri Command

**Difficulty:** 🟡 Intermediate | **Time:** 1 hour

### Goal

Create a Tauri command to get system information

### Requirements

- Rust command that returns CPU count and OS type
- Frontend service to call it
- Display in UI

### Steps

1. **Create Rust Command**

```rust
// src-tauri/src/core/system/commands.rs
use std::env;

#[derive(Serialize)]
pub struct SystemInfo {
    cpu_count: usize,
    os_type: String,
}

#[tauri::command]
pub fn get_system_info() -> SystemInfo {
    SystemInfo {
        cpu_count: num_cpus::get(),
        os_type: env::consts::OS.to_string(),
    }
}
```

2. **Register Command**

```rust
// src-tauri/src/lib.rs
.invoke_handler(tauri::generate_handler![
    // ... existing
    core::system::commands::get_system_info,
])
```

3. **Call from Frontend**

```typescript
// Create service or call directly
const sysInfo = await invoke('get_system_info')
console.log(`CPU Count: ${sysInfo.cpu_count}`)
console.log(`OS: ${sysInfo.os_type}`)
```

4. **Display in UI**

Add to System Monitor page or Settings

### Test

Run app and verify system info displays correctly

---

## Exercise 4: Implement a Zustand Store

**Difficulty:** 🟡 Intermediate | **Time:** 1 hour

### Goal

Create a notifications store for in-app notifications

### Requirements

- Add notification
- Remove notification
- Auto-dismiss after timeout
- Notification types: info, success, error

### Template

```typescript
// services/notifications/store.ts
interface Notification {
  id: string
  type: 'info' | 'success' | 'error'
  message: string
  createdAt: number
}

interface NotificationsState {
  notifications: Notification[]
}

interface NotificationsActions {
  addNotification: (
    type: Notification['type'],
    message: string
  ) => void
  removeNotification: (id: string) => void
}

export const useNotifications = create<
  NotificationsState & NotificationsActions
>((set) => ({
  notifications: [],

  // TODO: Implement actions
}))
```

### Steps

1. Implement `addNotification` with unique ID
2. Implement `removeNotification`
3. Add auto-dismiss using `setTimeout`
4. Create UI component to display notifications

### Bonus

Add persistence using localStorage

---

## Exercise 5: Create a New Route

**Difficulty:** 🟢 Beginner | **Time:** 30 minutes

### Goal

Add a "About" page at `/about`

### Requirements

- New route file
- Display app version
- Display credits/contributors
- Link from settings

### Steps

1. **Create Route File**

```typescript
// web-app/src/routes/about.tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/about')({
  component: AboutPage,
})

function AboutPage() {
  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold">About Jan AI</h1>
      {/* TODO: Add content */}
    </div>
  )
}
```

2. **Get App Version**

```typescript
import { getVersion } from '@tauri-apps/api/app'

const [version, setVersion] = useState('')

useEffect(() => {
  getVersion().then(setVersion)
}, [])
```

3. **Add Link**

```typescript
// In settings or menu
<Link to="/about">About</Link>
```

### Test

Navigate to `/about` and verify content displays

---

## Exercise 6: Implement Form Validation

**Difficulty:** 🟡 Intermediate | **Time:** 1 hour

### Goal

Add validation to a settings form

### Requirements

- Validate required fields
- Show error messages
- Prevent submit if invalid
- Clear errors on fix

### Template

```typescript
function SettingsForm() {
  const [name, setName] = useState('')
  const [email, setEmail] = useState('')
  const [errors, setErrors] = useState<Record<string, string>>({})

  const validate = () => {
    const newErrors: Record<string, string> = {}

    // TODO: Add validation

    setErrors(newErrors)
    return Object.keys(newErrors).length === 0
  }

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    if (validate()) {
      // Submit
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* TODO: Add fields with error display */}
    </form>
  )
}
```

### Validation Rules

- Name: Required, min 2 characters
- Email: Required, valid email format

### Solution Hints

- Use regex for email: `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`
- Clear specific error when field changes
- Show errors below input fields

---

## Exercise 7: Real-Time Progress Tracking

**Difficulty:** 🔴 Advanced | **Time:** 2 hours

### Goal

Implement real-time progress tracking for a long operation

### Requirements

- Backend emits progress events
- Frontend displays progress bar
- Shows ETA and speed
- Cancellable operation

### Backend (Rust)

```rust
#[tauri::command]
async fn long_operation(
    app: AppHandle,
    state: State<'_, AppState>
) -> Result<(), String> {
    let total = 100;

    for i in 0..=total {
        // Check if cancelled
        if state.is_cancelled() {
            return Err("Cancelled".to_string());
        }

        // Emit progress
        app.emit("operation_progress", ProgressEvent {
            current: i,
            total,
            percent: (i as f64 / total as f64) * 100.0,
        })?;

        tokio::time::sleep(Duration::from_millis(100)).await;
    }

    Ok(())
}
```

### Frontend (React)

```typescript
function ProgressTracker() {
  const [progress, setProgress] = useState(0)
  const [isRunning, setIsRunning] = useState(false)

  useEffect(() => {
    // Setup listener
    // Start operation
    // Handle completion
    // Cleanup
  }, [])

  return (
    <div>
      <ProgressBar value={progress} />
      <button onClick={cancelOperation}>Cancel</button>
    </div>
  )
}
```

### Steps

1. Create cancellation mechanism in state
2. Emit progress events from Rust
3. Listen to events in React
4. Calculate and display ETA
5. Implement cancel functionality

---

## Challenge Projects

**Once you've completed the exercises, try these mini-projects:**

### Project 1: Theme Customizer

Create a theme customizer where users can:
- Choose primary color
- Set font size
- Save custom themes
- Export/import themes

### Project 2: Keyboard Shortcut Manager

Build a UI for managing keyboard shortcuts:
- Display all shortcuts
- Allow users to customize
- Detect conflicts
- Save preferences

### Project 3: Export/Import Settings

Implement settings backup/restore:
- Export all settings to JSON
- Import from file
- Validate imported settings
- Merge or replace options

---

## Debugging Exercises

### Debug Exercise 1: Find the Bug

**Scenario:** User reports settings not saving

**Code:**
```typescript
const saveSettings = async (settings) => {
  const result = await invoke('save_settings', settings)
  if (result) {
    toast.success('Saved!')
  }
}
```

**Question:** What's wrong?

<details>
<summary>Answer</summary>

Missing destructuring: `{ settings }` instead of `settings`

```typescript
const result = await invoke('save_settings', { settings })
```

</details>

### Debug Exercise 2: Race Condition

**Scenario:** Sometimes duplicate messages appear

**Code:**
```typescript
const sendMessage = async (content: string) => {
  const message = { id: Date.now(), content }
  addMessageToStore(message)
  await invoke('send_message', { message })
}
```

**Question:** What could cause duplicates?

<details>
<summary>Answer</summary>

Date.now() can return same value if called quickly. Use crypto.randomUUID() instead:

```typescript
const message = { id: crypto.randomUUID(), content }
```

</details>

---

## Next Steps

**Completed exercises?**

- **Contribute:** [First Contributions](./FIRST_CONTRIBUTIONS.md)
- **Build more:** Tackle real GitHub issues
- **Help others:** Answer questions in Discord
- **Deepen knowledge:** Read [Advanced Patterns](./PATTERNS_AND_CONVENTIONS.md)

---

**Questions?** Return to the [Learning Hub](./README.md)
