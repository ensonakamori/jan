# Patterns and Conventions

**Code style, patterns, and best practices**

**Documented:** November 18, 2025

---

## Code Style

### TypeScript/React

**ESLint Config:** [web-app/.eslintrc.js](../../web-app/.eslintrc.js)
**Prettier Config:** [.prettierrc](../../.prettierrc)

#### Naming Conventions

```typescript
// ✅ Components: PascalCase
function ButtonComponent() {}
export function UserProfile() {}

// ✅ Hooks: camelCase with "use" prefix
function useAuth() {}
function useLocalStorage() {}

// ✅ Variables/functions: camelCase
const userName = 'John'
function handleClick() {}

// ✅ Constants: UPPER_SNAKE_CASE
const MAX_RETRIES = 3
const API_BASE_URL = 'https://api.example.com'

// ✅ Types/Interfaces: PascalCase
interface UserProfile {}
type ButtonProps = {}

// ✅ Files: Match export name
ButtonComponent.tsx
useAuth.ts
userProfile.types.ts
```

#### Import Order

```typescript
// 1. External libraries
import { useState } from 'react'
import { invoke } from '@tauri-apps/api/core'

// 2. Internal aliases (@/)
import { Button } from '@/components/ui/button'
import { useThreads } from '@/hooks/useThreads'

// 3. Relative imports
import { helper } from './utils'
import styles from './styles.module.css'
```

---

## React Patterns

### Component Structure

```typescript
// ✅ Good: Clear, organized
interface ButtonProps {
  children: React.ReactNode
  onClick?: () => void
  variant?: 'primary' | 'secondary'
  disabled?: boolean
}

export function Button({
  children,
  onClick,
  variant = 'primary',
  disabled = false,
}: ButtonProps) {
  // Hooks first
  const [isLoading, setIsLoading] = useState(false)

  // Event handlers
  const handleClick = () => {
    if (disabled || isLoading) return
    onClick?.()
  }

  // Render
  return (
    <button
      onClick={handleClick}
      className={cn('btn', `btn-${variant}`)}
      disabled={disabled || isLoading}
    >
      {children}
    </button>
  )
}
```

### Hooks Best Practices

```typescript
// ✅ Good: Custom hooks extract logic
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])

  return debouncedValue
}

// ❌ Bad: Logic in component
function SearchComponent() {
  const [debouncedQuery, setDebouncedQuery] = useState('')

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedQuery(query), 300)
    return () => clearTimeout(timer)
  }, [query])
}
```

---

## State Management (Zustand)

### Store Pattern

```typescript
// ✅ Good: Typed, organized
interface ThreadsState {
  threads: Thread[]
  activeId: string | null
}

interface ThreadsActions {
  setThreads: (threads: Thread[]) => void
  addThread: (thread: Thread) => void
  removeThread: (id: string) => void
}

export const useThreads = create<ThreadsState & ThreadsActions>((set) => ({
  threads: [],
  activeId: null,

  setThreads: (threads) => set({ threads }),

  addThread: (thread) =>
    set((state) => ({
      threads: [...state.threads, thread],
    })),

  removeThread: (id) =>
    set((state) => ({
      threads: state.threads.filter((t) => t.id !== id),
    })),
}))
```

### Selector Pattern

```typescript
// ✅ Good: Select only what you need
const threads = useThreads((state) => state.threads)
const activeThread = useThreads((state) =>
  state.threads.find((t) => t.id === state.activeId)
)

// ❌ Bad: Select entire store (causes unnecessary re-renders)
const { threads, activeId, setThreads, addThread } = useThreads()
```

---

## Service Layer Pattern

### Service Structure

```typescript
// types.ts - Interface
export interface AppService {
  getConfig(): Promise<Config>
  updateConfig(config: Config): Promise<void>
}

// tauri.ts - Implementation
export const tauriAppService: AppService = {
  async getConfig() {
    return await invoke('get_app_configurations')
  },

  async updateConfig(config) {
    await invoke('update_app_configuration', { config })
  },
}

// index.ts - Platform selection
import { isTauri } from '@/lib/platform'

export const appService: AppService = isTauri()
  ? tauriAppService
  : webAppService
```

---

## Tauri Command Patterns

### Rust Side

```rust
// ✅ Good: Return Result, use ? operator
#[tauri::command]
pub fn safe_operation(path: String) -> Result<String, String> {
    let content = fs::read_to_string(&path)
        .map_err(|e| e.to_string())?;
    Ok(content)
}

// ❌ Bad: Unwrap (crashes on error!)
#[tauri::command]
pub fn unsafe_operation(path: String) -> String {
    fs::read_to_string(&path).unwrap()
}
```

### TypeScript Side

```typescript
// ✅ Good: Handle errors
async function loadConfig() {
  try {
    const config = await invoke('get_app_configurations')
    return config
  } catch (error) {
    console.error('Failed to load config:', error)
    toast.error('Failed to load configuration')
    return null
  }
}
```

---

## Error Handling

### Frontend

```typescript
// ✅ Good: Try-catch with user feedback
async function saveSettings(settings: Settings) {
  try {
    await settingsService.save(settings)
    toast.success('Settings saved successfully')
  } catch (error) {
    console.error('Failed to save settings:', error)
    toast.error('Failed to save settings. Please try again.')
    throw error // Re-throw if caller needs to handle
  }
}
```

### Backend

```rust
// ✅ Good: Result type with descriptive errors
#[tauri::command]
pub fn load_model(id: String) -> Result<Model, String> {
    let path = get_model_path(&id)
        .ok_or_else(|| format!("Model not found: {}", id))?;

    let model = Model::load(&path)
        .map_err(|e| format!("Failed to load model: {}", e))?;

    Ok(model)
}
```

---

## Async Patterns

### ✅ Current (Nov 2025): React 19 Actions

```typescript
function MessageForm() {
  const [isPending, startTransition] = useTransition()

  const handleSubmit = async (formData: FormData) => {
    startTransition(async () => {
      await sendMessage(formData.get('message') as string)
    })
  }

  return (
    <form action={handleSubmit}>
      <button disabled={isPending}>
        {isPending ? 'Sending...' : 'Send'}
      </button>
    </form>
  )
}
```

### Optimistic Updates

```typescript
const [optimisticMessages, addOptimistic] = useOptimistic(
  messages,
  (state, newMessage: Message) => [...state, newMessage]
)

async function sendMessage(content: string) {
  const tempMessage = { id: crypto.randomUUID(), content, pending: true }

  addOptimistic(tempMessage) // Show immediately

  try {
    const realMessage = await messageService.send(content)
    // Optimistic message automatically replaced when messages updates
  } catch (error) {
    toast.error('Failed to send message')
  }
}
```

---

## Type Safety

### Strict TypeScript

```typescript
// ✅ Good: Explicit types
function processUser(user: User): ProcessedUser {
  return {
    id: user.id,
    displayName: `${user.firstName} ${user.lastName}`,
    isActive: user.status === 'active',
  }
}

// ❌ Bad: Any types
function processUser(user: any): any {
  return {
    id: user.id,
    displayName: user.firstName + ' ' + user.lastName,
  }
}
```

### Type Guards

```typescript
// ✅ Good: Type narrowing
function isThread(value: unknown): value is Thread {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'messages' in value
  )
}

if (isThread(data)) {
  // TypeScript knows data is Thread here
  console.log(data.messages)
}
```

---

## File Organization

### Component Files

```
Button/
├── Button.tsx           # Component
├── Button.test.tsx      # Tests
├── Button.stories.tsx   # Storybook (if used)
└── index.ts             # Re-export
```

### Service Files

```
services/threads/
├── index.ts            # Exports main service
├── types.ts            # Interface
├── tauri.ts            # Tauri implementation
├── web.ts              # Web implementation
├── default.ts          # Fallback
├── store.ts            # Zustand store
└── __tests__/
    └── threads.test.ts
```

---

## Testing Patterns

### Component Tests

```typescript
import { render, screen } from '@testing-library/react'
import { Button } from './Button'

describe('Button', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>Click</Button>)

    fireEvent.click(screen.getByText('Click'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

### Service Tests (Mock Tauri)

```typescript
import { vi } from 'vitest'

vi.mock('@tauri-apps/api/core', () => ({
  invoke: vi.fn(),
}))

describe('appService', () => {
  it('fetches config', async () => {
    const mockConfig = { theme: 'dark' }
    vi.mocked(invoke).mockResolvedValue(mockConfig)

    const config = await appService.getConfig()
    expect(config).toEqual(mockConfig)
    expect(invoke).toHaveBeenCalledWith('get_app_configurations')
  })
})
```

---

## Performance Patterns

### Memoization

```typescript
// ✅ Good: Memoize expensive calculations
const filteredThreads = useMemo(
  () => threads.filter((t) => t.title.includes(searchQuery)),
  [threads, searchQuery]
)

// ✅ Good: Stable callbacks
const handleClick = useCallback(() => {
  doSomething(id)
}, [id])
```

### Code Splitting

```typescript
// ✅ Good: Lazy load heavy components
const HeavyComponent = lazy(() => import('./HeavyComponent'))

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  )
}
```

---

## Security Patterns

### Input Validation

```rust
// ✅ Good: Validate inputs
#[tauri::command]
pub fn read_file(path: String) -> Result<String, String> {
    // Validate path
    if path.contains("..") {
        return Err("Invalid path".to_string());
    }

    // Check file exists
    if !Path::new(&path).exists() {
        return Err("File not found".to_string());
    }

    fs::read_to_string(path).map_err(|e| e.to_string())
}
```

### XSS Prevention

```typescript
// ✅ Good: React auto-escapes
<div>{userInput}</div>

// ⚠️ Dangerous: Only use if you trust the HTML
<div dangerouslySetInnerHTML={{ __html: trustedHTML }} />
```

---

## Common Anti-Patterns to Avoid

### ❌ Prop Drilling

```typescript
// ❌ Bad: Passing props through many components
<Parent>
  <Child user={user}>
    <GrandChild user={user}>
      <GreatGrandChild user={user} />
    </GrandChild>
  </Child>
</Parent>

// ✅ Good: Use context or global state
const user = useAuth((state) => state.user)
```

### ❌ Mixing Concerns

```typescript
// ❌ Bad: Business logic in component
function ThreadList() {
  const [threads, setThreads] = useState([])

  useEffect(() => {
    invoke('get_threads').then(setThreads)
  }, [])

  return <div>{threads.map(renderThread)}</div>
}

// ✅ Good: Use service + store
function ThreadList() {
  const threads = useThreads((state) => state.threads)
  return <div>{threads.map(renderThread)}</div>
}
```

### ❌ Unwrap in Rust

```rust
// ❌ Bad: App crashes on error
let config = fs::read_to_string("config.json").unwrap();

// ✅ Good: Handle error gracefully
let config = fs::read_to_string("config.json")
    .map_err(|e| e.to_string())?;
```

---

## Commit Message Format

```
type(scope): subject

body (optional)

footer (optional)
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting
- `refactor`: Code restructure
- `test`: Tests
- `chore`: Maintenance

**Examples:**
```
feat(threads): add message search functionality

fix(downloads): prevent duplicate downloads

docs: update architecture overview

refactor(services): simplify service layer pattern
```

---

## Next Steps

- **Practical Examples:** [How-To Guide](./HOW_TO_GUIDE.md)
- **Code Tours:** [Guided Walkthroughs](./CODE_TOURS.md)
- **Testing:** [Testing Guide](./TESTING_GUIDE.md)

**Questions?** Return to the [Learning Hub](./README.md)
