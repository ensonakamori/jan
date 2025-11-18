# Debugging Guide

**Tools and techniques for debugging Jan AI**

**Documented:** November 18, 2025

---

## Frontend Debugging

### Browser DevTools

**Opening DevTools in Jan:**
- **macOS:** `Cmd + Option + I`
- **Windows/Linux:** `Ctrl + Shift + I`

**Key Panels:**
- **Console** - View logs, errors, warnings
- **Sources** - Set breakpoints, step through code
- **Network** - Monitor API calls (if applicable)
- **Performance** - Profile slow operations
- **React DevTools** - Inspect component tree

### Console Debugging

```typescript
// Basic logging
console.log('Value:', value)
console.error('Error:', error)
console.warn('Warning:', warning)
console.table(data) // Display arrays/objects as table

// Grouped logging
console.group('Operation')
console.log('Step 1')
console.log('Step 2')
console.groupEnd()

// Timing
console.time('operation')
expensiveOperation()
console.timeEnd('operation') // "operation: 123.45ms"

// Conditional logging
console.assert(value > 0, 'Value must be positive')

// Stack trace
console.trace('Call stack')
```

### Breakpoint Debugging

```typescript
function processData(data: Data) {
  debugger // Execution pauses here when DevTools open

  const result = transform(data)
  return result
}
```

**DevTools Breakpoints:**
1. Open Sources tab
2. Find file
3. Click line number to set breakpoint
4. Reload or trigger action
5. Execution pauses at breakpoint

**Controls:**
- **Continue (F8)** - Resume execution
- **Step Over (F10)** - Execute line, don't enter functions
- **Step Into (F11)** - Enter function calls
- **Step Out (Shift+F11)** - Exit current function

---

## React DevTools

### Installing

**Browser Extension:**
- Chrome: Install from Chrome Web Store
- Firefox: Install from Firefox Add-ons

**Features:**
- Inspect component tree
- View props and state
- Edit props/state in real-time
- Profile component renders
- Highlight re-renders

### Using React DevTools

```typescript
// In components, you'll see:
// - Component name
// - Props
// - Hooks (useState, useEffect, etc.)
// - Context values
```

**Profiler:**
1. Click "Profiler" tab
2. Click record
3. Interact with app
4. Stop recording
5. Analyze render times

---

## Debugging State (Zustand)

### Zustand DevTools

```typescript
import { create } from 'zustand'
import { devtools } from 'zustand/middleware'

export const useThreads = create(
  devtools(
    (set) => ({
      threads: [],
      addThread: (thread) =>
        set((state) => ({ threads: [...state.threads, thread] })),
    }),
    { name: 'ThreadsStore' }
  )
)
```

**View in Redux DevTools:**
- Install Redux DevTools extension
- Open DevTools → Redux tab
- See all Zustand state changes

### Manual State Logging

```typescript
// Log state changes
const useThreads = create((set) => ({
  threads: [],
  addThread: (thread) =>
    set((state) => {
      console.log('Adding thread:', thread)
      console.log('Current state:', state)
      const newState = { threads: [...state.threads, thread] }
      console.log('New state:', newState)
      return newState
    }),
}))
```

---

## Backend Debugging (Rust/Tauri)

### Print Debugging

```rust
// println! - Simple output
println!("Value: {}", value);
println!("Debug: {:?}", complex_value); // Debug format
println!("Pretty: {:#?}", complex_value); // Pretty print

// dbg! - Debug macro (shows value and location)
let result = dbg!(some_expression);
```

### Logging with log Crate

```rust
use log::{trace, debug, info, warn, error};

#[tauri::command]
fn my_command(input: String) -> Result<String, String> {
    info!("Command called with input: {}", input);

    if input.is_empty() {
        warn!("Empty input received");
        return Err("Input cannot be empty".to_string());
    }

    debug!("Processing input...");
    let result = process(input);

    info!("Command completed successfully");
    Ok(result)
}
```

**Set log level:**
```bash
# In terminal before running
export RUST_LOG=debug
yarn dev
```

**Levels (lowest to highest):**
- `trace` - Very detailed
- `debug` - Debugging info
- `info` - General info
- `warn` - Warnings
- `error` - Errors only

### VS Code Rust Debugging

**Install:**
- CodeLLDB extension

**Configuration:** `.vscode/launch.json`
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Debug Tauri",
      "cargo": {
        "args": ["build", "--manifest-path=src-tauri/Cargo.toml"]
      },
      "cwd": "${workspaceFolder}"
    }
  ]
}
```

---

## Common Debugging Scenarios

### State Not Updating

```typescript
// Problem: State doesn't update
const [count, setCount] = useState(0)

const increment = () => {
  setCount(count + 1) // ❌ Uses stale value in closure
}

// Solution: Use functional update
const increment = () => {
  setCount((prev) => prev + 1) // ✅
}
```

### Component Not Re-rendering

```typescript
// Problem: Zustand selector not causing re-render
const threads = useThreads() // ❌ Entire store

// Solution: Select only what you need
const threads = useThreads((state) => state.threads) // ✅
```

### Infinite Re-renders

```typescript
// Problem: useEffect runs infinitely
useEffect(() => {
  fetchData()
}, [data]) // ❌ data changes, triggers fetch, updates data...

// Solution: Use correct dependencies
useEffect(() => {
  fetchData()
}, []) // ✅ Run once

// OR use ref for stable reference
const dataRef = useRef(data)
useEffect(() => {
  fetchData(dataRef.current)
}, [])
```

### Tauri Command Not Found

```typescript
// Error: "Command not found: my_command"

// Check:
// 1. Command defined in Rust?
#[tauri::command]
fn my_command() -> Result<String, String> { ... }

// 2. Registered in lib.rs?
.invoke_handler(tauri::generate_handler![
    my_command, // ← Must be here
])

// 3. Correct name in frontend?
await invoke('my_command') // ← Matches Rust function name
```

---

## Network Debugging

### Monitoring Tauri IPC

```typescript
// Wrap invoke to log all calls
const originalInvoke = invoke

globalThis.invoke = async (cmd: string, args?: any) => {
  console.log('→ Tauri Command:', cmd, args)
  try {
    const result = await originalInvoke(cmd, args)
    console.log('← Tauri Result:', result)
    return result
  } catch (error) {
    console.error('← Tauri Error:', error)
    throw error
  }
}
```

### HTTP Requests (if using fetch)

```typescript
// Intercept fetch
const originalFetch = window.fetch

window.fetch = async (...args) => {
  console.log('→ Fetch:', args[0])
  const response = await originalFetch(...args)
  console.log('← Response:', response.status)
  return response
}
```

---

## Performance Debugging

### React Performance

```typescript
import { Profiler } from 'react'

function onRenderCallback(
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime
) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`)
}

<Profiler id="ThreadList" onRender={onRenderCallback}>
  <ThreadList />
</Profiler>
```

### Find Slow Renders

```typescript
// Add to components
useEffect(() => {
  const start = performance.now()
  return () => {
    const duration = performance.now() - start
    if (duration > 16) {
      // Slower than 60fps
      console.warn(`Slow render: ${duration}ms`)
    }
  }
})
```

### Rust Performance

```rust
use std::time::Instant;

#[tauri::command]
async fn slow_operation() -> Result<String, String> {
    let start = Instant::now();

    // Your code
    expensive_operation();

    let duration = start.elapsed();
    info!("Operation took: {:?}", duration);

    Ok("Done".to_string())
}
```

---

## Memory Debugging

### Memory Leaks

**Finding leaks:**
1. Open DevTools → Performance
2. Take heap snapshot
3. Interact with app
4. Take another snapshot
5. Compare to find leaks

**Common causes:**
```typescript
// ❌ Event listener not cleaned up
useEffect(() => {
  window.addEventListener('resize', handleResize)
  // Missing cleanup!
})

// ✅ Cleanup properly
useEffect(() => {
  window.addEventListener('resize', handleResize)
  return () => {
    window.removeEventListener('resize', handleResize)
  }
}, [])

// ❌ Timer not cleared
useEffect(() => {
  setInterval(() => {
    console.log('tick')
  }, 1000)
  // Missing cleanup!
})

// ✅ Clear timer
useEffect(() => {
  const timer = setInterval(() => {
    console.log('tick')
  }, 1000)
  return () => {
    clearInterval(timer)
  }
}, [])
```

---

## Error Tracking

### Error Boundaries

```typescript
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null }

  static getDerivedStateFromError(error) {
    return { hasError: true, error }
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo)
    // Send to error tracking service
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />
    }
    return this.props.children
  }
}
```

### Global Error Handler

```typescript
window.addEventListener('error', (event) => {
  console.error('Global error:', event.error)
})

window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason)
})
```

---

## Source Maps

**Enable in production:**
```typescript
// vite.config.ts
export default defineConfig({
  build: {
    sourcemap: true, // Generate source maps
  },
})
```

**Result:** Readable stack traces in production

---

## Debugging Tools

### Useful Chrome DevTools Snippets

**Snippet 1: Find all Zustand stores**
```javascript
Object.keys(window).filter((key) => key.startsWith('use'))
```

**Snippet 2: Log all Tauri commands**
```javascript
const orig = window.__TAURI_INVOKE__
window.__TAURI_INVOKE__ = (...args) => {
  console.log('Tauri:', args)
  return orig(...args)
}
```

---

## Debugging Checklist

When something doesn't work:

- [ ] Check browser console for errors
- [ ] Check terminal for Rust errors
- [ ] Verify Tauri command is registered
- [ ] Check component is receiving props
- [ ] Verify state is updating (Zustand DevTools)
- [ ] Check network tab (if applicable)
- [ ] Add console.logs at key points
- [ ] Use debugger/breakpoints
- [ ] Check for TypeScript errors
- [ ] Restart dev server
- [ ] Clear cache and rebuild

---

## Next Steps

- **Testing:** [TESTING_GUIDE.md](./TESTING_GUIDE.md)
- **Workflow:** [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md)
- **Troubleshooting:** [FAQ](./FAQ.md)

**Questions?** Return to the [Learning Hub](./README.md)
