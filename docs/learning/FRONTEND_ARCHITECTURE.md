# Frontend Architecture

**Deep dive into the React 19 + TypeScript + TanStack Router frontend**

**Documented:** November 18, 2025 | **Stack:** React 19, TypeScript 5.9, TanStack Router, Zustand, TailwindCSS 4

---

## Table of Contents

- [Overview](#overview)
- [Routing System](#routing-system)
- [State Management](#state-management)
- [Component Architecture](#component-architecture)
- [Service Layer](#service-layer)
- [Providers & Context](#providers--context)
- [Styling System](#styling-system)
- [Internationalization](#internationalization)
- [Performance Patterns](#performance-patterns)

---

## Overview

The Jan AI frontend is a modern React 19 single-page application built with:

✅ **CURRENT (Nov 2025):**
- React 19.0.0 - Latest stable with Actions API
- TypeScript 5.9.2 - Latest with deferred imports
- TanStack Router 1.117.0 - Type-safe file-based routing
- Zustand 5.0.3 - Minimal state management
- TailwindCSS 4.1.4 - Latest with CSS-first config

**Architecture Pattern:**
```
Routes (Pages) → Containers (Smart) → Components (Dumb)
                      ↓
                  Services (Business Logic)
                      ↓
                Tauri Commands / Web APIs
```

---

## Routing System

### TanStack Router (File-Based)

**Location:** [web-app/src/routes/](../../web-app/src/routes/)

**How It Works:**
```
File Path                          → Route Path
routes/index.tsx                   → /
routes/__root.tsx                  → Root layout
routes/threads/$threadId.tsx       → /threads/:threadId
routes/hub/$modelId.tsx            → /hub/:modelId
routes/settings/general.tsx        → /settings/general
```

✅ **CURRENT (Nov 2025):** TanStack Router auto-generates type-safe route tree

**Root Layout:** [routes/__root.tsx](../../web-app/src/routes/__root.tsx)

```typescript
// Root layout wraps all routes
export const Route = createRootRoute({
  component: RootLayout,
  errorComponent: ({ error }) => <GlobalError error={error} />,
})

function RootLayout() {
  return (
    <>
      {/* Provider tree */}
      <ThemeProvider>
        <DataProvider>
          <ExtensionProvider>
            {/* More providers... */}
            <Outlet /> {/* Child routes render here */}
          </ExtensionProvider>
        </DataProvider>
      </ThemeProvider>
    </>
  )
}
```

### Provider Hierarchy

**Order matters!** Providers wrap in this order (outermost to innermost):

1. `TranslationProvider` - i18next setup
2. `ThemeProvider` - Dark/light mode
3. `AuthProvider` - Authentication state
4. `AnalyticProvider` - Analytics tracking
5. `GoogleAnalyticsProvider` - Google Analytics
6. `ExtensionProvider` - Extension system
7. `DataProvider` - Initial data loading
8. `ServiceHubProvider` - Service layer setup
9. `GlobalEventHandler` - Tauri event listeners
10. `KeyboardShortcutsProvider` - Keyboard shortcuts
11. `InterfaceProvider` - UI state (left panel, etc.)
12. `ToasterProvider` - Toast notifications

💡 **Aha Moment:** Each provider sets up global functionality. Child routes inherit all context.

### Type-Safe Navigation

```typescript
import { useNavigate } from '@tanstack/react-router'

const navigate = useNavigate()

// Type-safe navigation (autocomplete & validation!)
navigate({
  to: '/threads/$threadId',
  params: { threadId: '123' },    // TypeScript knows this is required
  search: { filter: 'active' }    // Optional query params
})
```

🎯 **Remember This:** TypeScript will error if you forget required params or use wrong route names!

---

## State Management

### Zustand Stores

**Pattern:** One store per domain, colocated with service

```
web-app/src/services/
├── threads/
│   ├── store.ts        # ← Zustand store
│   ├── types.ts        # Service types
│   └── tauri.ts        # Implementation
├── models/
│   └── store.ts        # ← Model store
└── ...
```

⚠️ **Note:** Stores are in `services/` not a separate `stores/` directory

### Store Pattern

```typescript
// Example: Thread store
import { create } from 'zustand'

interface ThreadsState {
  threads: Thread[]
  activeThreadId: string | null
}

interface ThreadsActions {
  setThreads: (threads: Thread[]) => void
  addThread: (thread: Thread) => void
  setActiveThread: (id: string) => void
}

export const useThreads = create<ThreadsState & ThreadsActions>((set) => ({
  // Initial state
  threads: [],
  activeThreadId: null,

  // Actions
  setThreads: (threads) => set({ threads }),

  addThread: (thread) => set((state) => ({
    threads: [...state.threads, thread]
  })),

  setActiveThread: (id) => set({ activeThreadId: id }),
}))
```

### Using Stores in Components

```typescript
function ThreadList() {
  // Select only what you need
  const threads = useThreads(state => state.threads)
  const addThread = useThreads(state => state.addThread)

  // Component logic...
}
```

✅ **Best Practice:** Use selector functions to avoid unnecessary re-renders

🌉 **Bridge from Redux:**
- No `dispatch`, no action creators
- Direct function calls
- Simpler, less boilerplate
- Same reactivity guarantees

---

## Component Architecture

### Three Component Types

#### 1. Route Components (Pages)

**Location:** `routes/*.tsx`

**Purpose:** Page-level components, handle routing

```typescript
// routes/threads/$threadId.tsx
export const Route = createFileRoute('/threads/$threadId')({
  component: ThreadPage,
})

function ThreadPage() {
  const { threadId } = Route.useParams()
  // Use services, stores, hooks
  return <ThreadContainer threadId={threadId} />
}
```

#### 2. Container Components (Smart)

**Location:** `containers/`

**Purpose:** Business logic, data fetching, state management

```typescript
// containers/ThreadContainer.tsx
function ThreadContainer({ threadId }: Props) {
  const thread = useThreads(state =>
    state.threads.find(t => t.id === threadId)
  )
  const messages = useMessages(threadId)

  const handleSendMessage = async (content: string) => {
    // Business logic here
    await threadService.sendMessage(threadId, content)
  }

  return <ThreadView thread={thread} onSend={handleSendMessage} />
}
```

#### 3. Presentational Components (Dumb)

**Location:** `components/ui/`

**Purpose:** Pure UI, no business logic, reusable

```typescript
// components/ui/button.tsx
interface ButtonProps {
  children: React.ReactNode
  onClick?: () => void
  variant?: 'primary' | 'secondary'
}

export function Button({ children, onClick, variant }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      className={cn('btn', variant === 'primary' && 'btn-primary')}
    >
      {children}
    </button>
  )
}
```

🎯 **Remember This:**
```
Route → Container → Component
(Page)  (Smart)     (Dumb)
```

---

## Service Layer

### Purpose

**Abstraction over Tauri/Web APIs** - same interface, different implementations

### Service Structure

```
services/app/
├── types.ts      # Interface definition
├── tauri.ts      # Tauri implementation
├── web.ts        # Web implementation
└── default.ts    # Fallback implementation
```

### Interface Definition

```typescript
// services/app/types.ts
export interface AppService {
  getConfigurations(): Promise<AppConfiguration>
  updateConfiguration(config: Partial<AppConfiguration>): Promise<void>
  getDataFolder(): Promise<string>
}
```

### Tauri Implementation

```typescript
// services/app/tauri.ts
import { invoke } from '@tauri-apps/api/core'

export const tauriAppService: AppService = {
  async getConfigurations() {
    return await invoke('get_app_configurations')
  },

  async updateConfiguration(config) {
    return await invoke('update_app_configuration', { config })
  },

  async getDataFolder() {
    return await invoke('get_jan_data_folder_path')
  },
}
```

### Platform Selection

```typescript
// services/app/index.ts
import { isTauri } from '@/lib/platform'
import { tauriAppService } from './tauri'
import { webAppService } from './web'

export const appService: AppService = isTauri()
  ? tauriAppService
  : webAppService
```

💡 **Aha Moment:** Components don't know if they're calling Tauri or web APIs - the service layer handles it!

---

## Providers & Context

### Available Providers

| Provider | Purpose | Location |
|----------|---------|----------|
| **ThemeProvider** | Dark/light mode | [providers/ThemeProvider.tsx](../../web-app/src/providers/ThemeProvider.tsx) |
| **DataProvider** | Initial data loading | [providers/DataProvider.tsx](../../web-app/src/providers/DataProvider.tsx) |
| **ExtensionProvider** | Extension system | [providers/ExtensionProvider.tsx](../../web-app/src/providers/ExtensionProvider.tsx) |
| **InterfaceProvider** | UI state (panels, etc.) | [providers/InterfaceProvider.tsx](../../web-app/src/providers/InterfaceProvider.tsx) |
| **KeyboardShortcutsProvider** | Keyboard shortcuts | [providers/KeyboardShortcuts.tsx](../../web-app/src/providers/KeyboardShortcuts.tsx) |
| **AuthProvider** | Authentication | [providers/AuthProvider.tsx](../../web-app/src/providers/AuthProvider.tsx) |
| **GlobalEventHandler** | Tauri events | [providers/GlobalEventHandler.tsx](../../web-app/src/providers/GlobalEventHandler.tsx) |

### Creating a Custom Hook from Context

```typescript
// providers/ThemeProvider.tsx
const ThemeContext = createContext<ThemeContextType | undefined>(undefined)

export function useTheme() {
  const context = useContext(ThemeContext)
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider')
  }
  return context
}
```

---

## Styling System

### TailwindCSS 4 ✅ CURRENT (Nov 2025)

**Configuration:** [web-app/tailwind.config.js](../../web-app/tailwind.config.js)

#### CSS-First Configuration (New in Tailwind 4)

```css
/* global.css */
@theme {
  --color-primary: oklch(0.5 0.2 270);
  --font-sans: 'Inter', sans-serif;
}
```

#### Utility Classes

```tsx
<div className="flex items-center gap-4 p-6 bg-slate-100 dark:bg-slate-900">
  <h1 className="text-2xl font-bold text-slate-900 dark:text-white">
    Hello
  </h1>
</div>
```

#### Component Styling with `cn` Utility

```typescript
import { cn } from '@/lib/utils'

<Button className={cn(
  'base-styles',
  variant === 'primary' && 'primary-styles',
  disabled && 'opacity-50'
)} />
```

**cn Utility:** [lib/utils.ts](../../web-app/src/lib/utils.ts)
```typescript
import { clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

💡 **Why cn?** Merges Tailwind classes correctly (later classes override earlier)

---

## Internationalization

### i18next Setup

**12 Languages Supported:**
- English (en)
- Chinese Simplified (zh-CN)
- Chinese Traditional (zh-TW)
- Japanese (ja)
- Vietnamese (vn)
- German (de-DE)
- French (fr)
- Indonesian (id)
- Polish (pl)
- Portuguese Brazil (pt-BR)
- Russian (ru)
- Czech (cs)

**Translation Files:** [web-app/src/locales/](../../web-app/src/locales/)

### Using Translations

```typescript
import { useTranslation } from 'react-i18next'

function MyComponent() {
  const { t } = useTranslation()

  return (
    <div>
      <h1>{t('welcome.title')}</h1>
      <p>{t('welcome.description', { name: 'User' })}</p>
    </div>
  )
}
```

### Translation File Structure

```json
// locales/en/translation.json
{
  "welcome": {
    "title": "Welcome to Jan AI",
    "description": "Hello, {{name}}!"
  }
}
```

---

## Performance Patterns

### React 19 Features Used

#### 1. Actions API ✅ NEW IN 2025

```typescript
function MessageForm() {
  const [isPending, startTransition] = useTransition()

  const handleSubmit = () => {
    startTransition(async () => {
      await sendMessage(content)
    })
  }

  return <form onSubmit={handleSubmit}>
    <button disabled={isPending}>
      {isPending ? 'Sending...' : 'Send'}
    </button>
  </form>
}
```

#### 2. useOptimistic Hook

```typescript
const [optimisticMessages, addOptimisticMessage] = useOptimistic(
  messages,
  (state, newMessage) => [...state, newMessage]
)

// Show message immediately, update when server confirms
addOptimisticMessage(tempMessage)
```

### Code Splitting

**Automatic with TanStack Router:**
```typescript
// Each route is automatically code-split
// Only loads when navigated to
```

### Memoization

```typescript
// Expensive calculation
const filteredThreads = useMemo(() =>
  threads.filter(t => t.title.includes(searchQuery)),
  [threads, searchQuery]
)

// Callback stability
const handleClick = useCallback(() => {
  doSomething(id)
}, [id])
```

⚠️ **Note:** React 19's compiler may auto-optimize these in the future

---

## Custom Hooks

**Location:** [web-app/src/hooks/](../../web-app/src/hooks/)

### Common Hooks

| Hook | Purpose |
|------|---------|
| `useThreads` | Access thread store |
| `useModels` | Access model store |
| `useAnalytic` | Analytics tracking |
| `useLeftPanel` | Left panel state |
| `useMediaQuery` | Responsive breakpoints |

### Creating a Custom Hook

```typescript
// hooks/useDebounce.ts
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])

  return debouncedValue
}
```

---

## Form Handling

### React Hook Form (if used)

⚠️ **UNCLEAR:** Project may or may not use React Hook Form

Check: [web-app/package.json](../../web-app/package.json)

### Native Form Handling

```typescript
function SettingsForm() {
  const [settings, setSettings] = useState(initialSettings)

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    await settingsService.update(settings)
  }

  return <form onSubmit={handleSubmit}>
    {/* form fields */}
  </form>
}
```

---

## Error Handling

### Error Boundaries

**Root Error Boundary:** [routes/__root.tsx](../../web-app/src/routes/__root.tsx)

```typescript
export const Route = createRootRoute({
  errorComponent: ({ error }) => <GlobalError error={error} />,
})
```

### Try-Catch in Services

```typescript
async function loadData() {
  try {
    const data = await apiService.fetchData()
    return data
  } catch (error) {
    console.error('Failed to load data:', error)
    toast.error('Failed to load data')
    throw error
  }
}
```

---

## Next Steps

- **Backend Deep Dive:** [Backend Architecture](./BACKEND_ARCHITECTURE.md)
- **Practical Examples:** [How-To Guide](./HOW_TO_GUIDE.md)
- **Code Tours:** [Guided Walkthroughs](./CODE_TOURS.md)
- **Patterns:** [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)

**Questions?** Return to the [Learning Hub](./README.md) or check [FAQ](./FAQ.md)
