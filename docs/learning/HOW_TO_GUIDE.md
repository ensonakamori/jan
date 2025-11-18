# How-To Guide

**Step-by-step instructions for common tasks**

**Documented:** November 18, 2025

---

## Adding a New Route

### Step 1: Create Route File

Create a file in `web-app/src/routes/`:

```bash
# File: web-app/src/routes/my-feature.tsx
```

```typescript
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/my-feature')({
  component: MyFeaturePage,
})

function MyFeaturePage() {
  return (
    <div>
      <h1>My Feature</h1>
    </div>
  )
}
```

### Step 2: Route Tree Auto-Generates

TanStack Router automatically updates `routeTree.gen.ts`.

### Step 3: Navigate to Route

```typescript
import { useNavigate } from '@tanstack/react-router'

const navigate = useNavigate()
navigate({ to: '/my-feature' })
```

**Result:** Visit `http://localhost:5173/my-feature`

---

## Creating a Zustand Store

### Step 1: Define Types

```typescript
// services/myFeature/types.ts
export interface MyFeatureState {
  items: Item[]
  isLoading: boolean
}

export interface MyFeatureActions {
  setItems: (items: Item[]) => void
  addItem: (item: Item) => void
  setLoading: (loading: boolean) => void
}
```

### Step 2: Create Store

```typescript
// services/myFeature/store.ts
import { create } from 'zustand'

export const useMyFeature = create<MyFeatureState & MyFeatureActions>((set) => ({
  items: [],
  isLoading: false,

  setItems: (items) => set({ items }),

  addItem: (item) =>
    set((state) => ({
      items: [...state.items, item],
    })),

  setLoading: (isLoading) => set({ isLoading }),
}))
```

### Step 3: Use in Component

```typescript
function MyComponent() {
  const items = useMyFeature((state) => state.items)
  const addItem = useMyFeature((state) => state.addItem)

  return <div>{/* Use items and addItem */}</div>
}
```

---

## Adding a Tauri Command

### Step 1: Create Rust Command

```rust
// src-tauri/src/core/mymodule/commands.rs
#[tauri::command]
pub fn my_command(param: String) -> Result<String, String> {
    // Your logic here
    Ok(format!("Processed: {}", param))
}
```

### Step 2: Register Command

```rust
// src-tauri/src/lib.rs
.invoke_handler(tauri::generate_handler![
    // ... existing commands
    core::mymodule::commands::my_command,
])
```

### Step 3: Call from Frontend

```typescript
import { invoke } from '@tauri-apps/api/core'

const result = await invoke('my_command', { param: 'test' })
console.log(result) // "Processed: test"
```

---

## Creating a Service

### Step 1: Define Interface

```typescript
// services/myService/types.ts
export interface MyService {
  getData(): Promise<Data>
  saveData(data: Data): Promise<void>
}
```

### Step 2: Tauri Implementation

```typescript
// services/myService/tauri.ts
import { invoke } from '@tauri-apps/api/core'

export const tauriMyService: MyService = {
  async getData() {
    return await invoke('get_data')
  },

  async saveData(data) {
    await invoke('save_data', { data })
  },
}
```

### Step 3: Web Implementation

```typescript
// services/myService/web.ts
export const webMyService: MyService = {
  async getData() {
    const response = await fetch('/api/data')
    return await response.json()
  },

  async saveData(data) {
    await fetch('/api/data', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    })
  },
}
```

### Step 4: Export with Platform Detection

```typescript
// services/myService/index.ts
import { isTauri } from '@/lib/platform'
import { tauriMyService } from './tauri'
import { webMyService } from './web'

export const myService: MyService = isTauri() ? tauriMyService : webMyService
```

---

## Adding Internationalization

### Step 1: Add Translation Keys

```json
// locales/en/translation.json
{
  "myFeature": {
    "title": "My Feature",
    "description": "Feature description",
    "button": "Click me"
  }
}
```

```json
// locales/zh-CN/translation.json
{
  "myFeature": {
    "title": "我的功能",
    "description": "功能描述",
    "button": "点击我"
  }
}
```

### Step 2: Use in Component

```typescript
import { useTranslation } from 'react-i18next'

function MyFeature() {
  const { t } = useTranslation()

  return (
    <div>
      <h1>{t('myFeature.title')}</h1>
      <p>{t('myFeature.description')}</p>
      <button>{t('myFeature.button')}</button>
    </div>
  )
}
```

---

## Creating a Component with TailwindCSS

```typescript
import { cn } from '@/lib/utils'

interface CardProps {
  title: string
  children: React.ReactNode
  variant?: 'default' | 'outlined'
}

export function Card({ title, children, variant = 'default' }: CardProps) {
  return (
    <div
      className={cn(
        'rounded-lg p-6',
        variant === 'default' && 'bg-white shadow-md',
        variant === 'outlined' && 'border border-gray-300'
      )}
    >
      <h3 className="text-xl font-semibold mb-4">{title}</h3>
      <div>{children}</div>
    </div>
  )
}
```

---

## Form Handling

### Simple Form

```typescript
function SettingsForm() {
  const [name, setName] = useState('')
  const [email, setEmail] = useState('')

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()

    try {
      await settingsService.update({ name, email })
      toast.success('Settings saved')
    } catch (error) {
      toast.error('Failed to save settings')
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit">Save</button>
    </form>
  )
}
```

---

## Real-Time Updates with Events

### Backend (Emit Event)

```rust
#[tauri::command]
async fn long_operation(app: AppHandle) -> Result<(), String> {
    for i in 0..100 {
        // Emit progress
        app.emit("operation_progress", i)?;
        tokio::time::sleep(Duration::from_millis(100)).await;
    }

    app.emit("operation_complete", ())?;
    Ok(())
}
```

### Frontend (Listen to Event)

```typescript
import { listen } from '@tauri-apps/api/event'

function ProgressTracker() {
  const [progress, setProgress] = useState(0)

  useEffect(() => {
    const setupListeners = async () => {
      const unlistenProgress = await listen('operation_progress', (event) => {
        setProgress(event.payload as number)
      })

      const unlistenComplete = await listen('operation_complete', () => {
        toast.success('Operation complete!')
      })

      return () => {
        unlistenProgress()
        unlistenComplete()
      }
    }

    setupListeners()
  }, [])

  return <ProgressBar value={progress} />
}
```

---

## Dialog/Modal Pattern

```typescript
import { Dialog, DialogContent } from '@/components/ui/dialog'

function MyFeature() {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <>
      <button onClick={() => setIsOpen(true)}>Open Dialog</button>

      <Dialog open={isOpen} onOpenChange={setIsOpen}>
        <DialogContent>
          <h2>Dialog Title</h2>
          <p>Dialog content</p>
          <button onClick={() => setIsOpen(false)}>Close</button>
        </DialogContent>
      </Dialog>
    </>
  )
}
```

---

## File Operations

### Read File

```rust
#[tauri::command]
pub fn read_user_file(path: String) -> Result<String, String> {
    std::fs::read_to_string(&path).map_err(|e| e.to_string())
}
```

```typescript
const content = await invoke('read_user_file', { path: '/path/to/file.txt' })
```

### Write File

```rust
#[tauri::command]
pub fn write_user_file(path: String, content: String) -> Result<(), String> {
    std::fs::write(&path, content).map_err(|e| e.to_string())
}
```

```typescript
await invoke('write_user_file', { path: '/path/to/file.txt', content: 'Hello' })
```

---

## Environment Variables

### Tauri Config

```json
// tauri.conf.json
{
  "build": {
    "beforeBuildCommand": "yarn build",
    "beforeDevCommand": "yarn dev",
    "devUrl": "http://localhost:5173"
  }
}
```

### Access in Rust

```rust
let env = std::env::var("MY_ENV_VAR").unwrap_or_default();
```

---

## Adding a Provider

### Step 1: Create Context

```typescript
// providers/MyProvider.tsx
import { createContext, useContext, useState } from 'react'

interface MyContextType {
  value: string
  setValue: (value: string) => void
}

const MyContext = createContext<MyContextType | undefined>(undefined)

export function MyProvider({ children }: { children: React.ReactNode }) {
  const [value, setValue] = useState('')

  return (
    <MyContext.Provider value={{ value, setValue }}>
      {children}
    </MyContext.Provider>
  )
}

export function useMyContext() {
  const context = useContext(MyContext)
  if (!context) {
    throw new Error('useMyContext must be used within MyProvider')
  }
  return context
}
```

### Step 2: Add to Root Layout

```typescript
// routes/__root.tsx
<MyProvider>
  <OtherProviders>{/* ... */}</OtherProviders>
</MyProvider>
```

---

## Testing

### Component Test

```typescript
import { render, screen } from '@testing-library/react'
import { MyComponent } from './MyComponent'

describe('MyComponent', () => {
  it('renders correctly', () => {
    render(<MyComponent title="Test" />)
    expect(screen.getByText('Test')).toBeInTheDocument()
  })
})
```

### Service Test

```typescript
import { vi } from 'vitest'
import { invoke } from '@tauri-apps/api/core'

vi.mock('@tauri-apps/api/core')

describe('myService', () => {
  it('calls Tauri command', async () => {
    vi.mocked(invoke).mockResolvedValue({ success: true })

    const result = await myService.getData()
    expect(invoke).toHaveBeenCalledWith('get_data')
    expect(result).toEqual({ success: true })
  })
})
```

---

## Debugging

### Frontend Debugging

```typescript
// Add breakpoints in browser DevTools
// Or use console.log
console.log('Debug:', value)

// Use React DevTools to inspect components
```

### Backend Debugging

```rust
// Add print statements
println!("Debug: {:?}", value);

// Use log crate
use log::info;
info!("Processing: {}", item_id);
```

View logs in terminal where `yarn dev` is running.

---

## Next Steps

- **Code Tours:** [Guided Walkthroughs](./CODE_TOURS.md)
- **Testing Guide:** [Comprehensive Testing](./TESTING_GUIDE.md)
- **Debugging:** [Advanced Debugging](./DEBUGGING_GUIDE.md)

**Questions?** Return to the [Learning Hub](./README.md)
