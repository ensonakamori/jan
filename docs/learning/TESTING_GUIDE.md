# Testing Guide

**Comprehensive testing strategies for Jan AI**

**Documented:** November 18, 2025 | **Framework:** Vitest 3.x

---

## Testing Stack

✅ **CURRENT (Nov 2025):**
- **Vitest 3.x** - Unit testing framework
- **@testing-library/react** - React component testing
- **@testing-library/user-event** - User interaction simulation
- **jsdom** - DOM environment for tests

⚠️ **Note:** Vitest 4.0 available (Oct 2025), consider upgrading

---

## Running Tests

```bash
# All tests
yarn test

# Watch mode (auto-rerun)
yarn test:watch

# With coverage
yarn test:coverage

# UI mode (visual runner)
yarn test:ui

# Specific file
yarn test Button.test.tsx

# Pattern matching
yarn test --grep "Button"
```

---

## Test File Organization

### Naming Convention

```
Component.tsx → Component.test.tsx
useHook.ts → useHook.test.ts
service.ts → service.test.ts
```

### Location Options

**Option A: Co-located**
```
components/
  ├── Button/
  │   ├── Button.tsx
  │   └── Button.test.tsx
```

**Option B: __tests__ directory**
```
components/
  ├── Button/
  │   ├── Button.tsx
  │   └── __tests__/
  │       └── Button.test.tsx
```

---

## Component Testing

### Basic Component Test

```typescript
import { render, screen } from '@testing-library/react'
import { Button } from './Button'

describe('Button', () => {
  it('renders children correctly', () => {
    render(<Button>Click me</Button>)

    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('calls onClick when clicked', async () => {
    const handleClick = vi.fn()
    const { user } = render(<Button onClick={handleClick}>Click</Button>)

    await user.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('applies disabled state', () => {
    render(<Button disabled>Click</Button>)

    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### Testing Component with Props

```typescript
interface CardProps {
  title: string
  description?: string
  onClose?: () => void
}

describe('Card', () => {
  const defaultProps: CardProps = {
    title: 'Test Card',
  }

  it('renders title', () => {
    render(<Card {...defaultProps} />)
    expect(screen.getByText('Test Card')).toBeInTheDocument()
  })

  it('renders description when provided', () => {
    render(<Card {...defaultProps} description="Test description" />)
    expect(screen.getByText('Test description')).toBeInTheDocument()
  })

  it('calls onClose when close button clicked', async () => {
    const handleClose = vi.fn()
    const { user } = render(<Card {...defaultProps} onClose={handleClose} />)

    await user.click(screen.getByRole('button', { name: /close/i }))

    expect(handleClose).toHaveBeenCalled()
  })
})
```

---

## Testing Hooks

### Custom Hook Testing

```typescript
import { renderHook, act } from '@testing-library/react'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter())

    expect(result.current.count).toBe(0)
  })

  it('increments count', () => {
    const { result } = renderHook(() => useCounter())

    act(() => {
      result.current.increment()
    })

    expect(result.current.count).toBe(1)
  })

  it('accepts initial value', () => {
    const { result } = renderHook(() => useCounter(10))

    expect(result.current.count).toBe(10)
  })
})
```

### Testing Hooks with Dependencies

```typescript
describe('useDebounce', () => {
  beforeEach(() => {
    vi.useFakeTimers()
  })

  afterEach(() => {
    vi.useRealTimers()
  })

  it('debounces value changes', () => {
    const { result, rerender } = renderHook(
      ({ value }) => useDebounce(value, 300),
      { initialProps: { value: 'initial' } }
    )

    expect(result.current).toBe('initial')

    // Change value
    rerender({ value: 'updated' })

    // Not yet updated
    expect(result.current).toBe('initial')

    // Fast-forward time
    act(() => {
      vi.advanceTimersByTime(300)
    })

    // Now updated
    expect(result.current).toBe('updated')
  })
})
```

---

## Testing with Zustand Stores

### Mocking Store

```typescript
import { useThreads } from '@/services/threads/store'

vi.mock('@/services/threads/store')

describe('ThreadList', () => {
  it('displays threads from store', () => {
    const mockThreads = [
      { id: '1', title: 'Thread 1' },
      { id: '2', title: 'Thread 2' },
    ]

    vi.mocked(useThreads).mockReturnValue({
      threads: mockThreads,
      addThread: vi.fn(),
    })

    render(<ThreadList />)

    expect(screen.getByText('Thread 1')).toBeInTheDocument()
    expect(screen.getByText('Thread 2')).toBeInTheDocument()
  })
})
```

### Testing Store Actions

```typescript
import { renderHook, act } from '@testing-library/react'
import { useThreads } from './store'

describe('useThreads store', () => {
  it('adds thread', () => {
    const { result } = renderHook(() => useThreads())
    const newThread = { id: '1', title: 'New Thread', messages: [] }

    act(() => {
      result.current.addThread(newThread)
    })

    expect(result.current.threads).toContainEqual(newThread)
  })

  it('removes thread', () => {
    const { result } = renderHook(() => useThreads())

    act(() => {
      result.current.addThread({ id: '1', title: 'Thread 1', messages: [] })
      result.current.removeThread('1')
    })

    expect(result.current.threads).toHaveLength(0)
  })
})
```

---

## Testing Tauri Commands

### Mocking invoke()

```typescript
import { invoke } from '@tauri-apps/api/core'

vi.mock('@tauri-apps/api/core', () => ({
  invoke: vi.fn(),
}))

describe('appService', () => {
  it('fetches configuration', async () => {
    const mockConfig = { theme: 'dark', language: 'en' }

    vi.mocked(invoke).mockResolvedValue(mockConfig)

    const config = await appService.getConfiguration()

    expect(invoke).toHaveBeenCalledWith('get_app_configurations')
    expect(config).toEqual(mockConfig)
  })

  it('handles errors', async () => {
    vi.mocked(invoke).mockRejectedValue(new Error('Command failed'))

    await expect(appService.getConfiguration()).rejects.toThrow(
      'Command failed'
    )
  })
})
```

---

## Testing Async Operations

### Testing Promises

```typescript
describe('fetchData', () => {
  it('fetches data successfully', async () => {
    const mockData = { id: 1, name: 'Test' }

    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => mockData,
    })

    const data = await fetchData('/api/test')

    expect(data).toEqual(mockData)
  })

  it('handles fetch errors', async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: false,
      statusText: 'Not Found',
    })

    await expect(fetchData('/api/test')).rejects.toThrow('Not Found')
  })
})
```

### Testing Loading States

```typescript
describe('DataLoader', () => {
  it('shows loading state', () => {
    render(<DataLoader />)
    expect(screen.getByText('Loading...')).toBeInTheDocument()
  })

  it('shows data after loading', async () => {
    render(<DataLoader />)

    await waitFor(() => {
      expect(screen.getByText('Data loaded')).toBeInTheDocument()
    })
  })
})
```

---

## Testing User Interactions

### Click Events

```typescript
it('toggles state on click', async () => {
  const { user } = render(<Toggle />)

  const button = screen.getByRole('button')
  expect(button).toHaveAttribute('aria-pressed', 'false')

  await user.click(button)

  expect(button).toHaveAttribute('aria-pressed', 'true')
})
```

### Form Input

```typescript
it('updates input value', async () => {
  const { user } = render(<SearchInput />)

  const input = screen.getByRole('textbox')

  await user.type(input, 'search query')

  expect(input).toHaveValue('search query')
})
```

### Keyboard Navigation

```typescript
it('navigates with keyboard', async () => {
  const { user } = render(<Menu />)

  await user.keyboard('{ArrowDown}')
  expect(screen.getByRole('menuitem', { name: 'Item 1' })).toHaveFocus()

  await user.keyboard('{ArrowDown}')
  expect(screen.getByRole('menuitem', { name: 'Item 2' })).toHaveFocus()

  await user.keyboard('{Enter}')
  // Assert selection
})
```

---

## Testing Rust Code

### Cargo Test

```bash
cd src-tauri
cargo test
```

### Unit Test Example

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_parse_config() {
        let json = r#"{"theme": "dark"}"#;
        let config: AppConfig = serde_json::from_str(json).unwrap();

        assert_eq!(config.theme, "dark");
    }

    #[test]
    fn test_validate_path() {
        assert!(validate_path("/valid/path").is_ok());
        assert!(validate_path("../invalid").is_err());
    }

    #[tokio::test]
    async fn test_async_operation() {
        let result = async_operation().await;
        assert!(result.is_ok());
    }
}
```

---

## Snapshot Testing

```typescript
import { render } from '@testing-library/react'

describe('Button snapshot', () => {
  it('matches snapshot', () => {
    const { container } = render(<Button>Click me</Button>)

    expect(container.firstChild).toMatchSnapshot()
  })
})
```

**Update snapshots:**
```bash
yarn test -u
```

---

## Coverage Requirements

### Viewing Coverage

```bash
yarn test:coverage
# Opens coverage/index.html
```

### Coverage Thresholds

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      lines: 80,
      functions: 80,
      branches: 80,
      statements: 80,
    },
  },
})
```

**What to cover:**
- ✅ All business logic
- ✅ Edge cases and error paths
- ✅ Critical user flows
- ⚠️ Simple presentational components (optional)

---

## Testing Best Practices

### ✅ Do

```typescript
// Test behavior, not implementation
it('displays error message when validation fails', () => {
  // Good: Tests what user sees
})

// Use meaningful test descriptions
it('calls onSubmit with form data when submit button is clicked', () => {
  // Clear and specific
})

// Arrange, Act, Assert pattern
it('increments counter', () => {
  // Arrange
  render(<Counter initialValue={0} />)

  // Act
  fireEvent.click(screen.getByRole('button', { name: /increment/i }))

  // Assert
  expect(screen.getByText('1')).toBeInTheDocument()
})
```

### ❌ Don't

```typescript
// Don't test implementation details
it('sets state.count to 1', () => {
  // Bad: Testing internal state
})

// Don't use vague descriptions
it('works correctly', () => {
  // What does "works" mean?
})

// Don't test multiple things
it('does everything', () => {
  // Test one behavior per test
})
```

---

## Mocking Strategies

### Mock Functions

```typescript
const mockFn = vi.fn()
mockFn.mockReturnValue('mocked')
mockFn.mockResolvedValue('async mocked')
mockFn.mockRejectedValue(new Error('error'))

expect(mockFn).toHaveBeenCalled()
expect(mockFn).toHaveBeenCalledWith('arg')
expect(mockFn).toHaveBeenCalledTimes(2)
```

### Mock Modules

```typescript
vi.mock('@/services/api', () => ({
  apiService: {
    fetch: vi.fn().mockResolvedValue({ data: 'mocked' }),
  },
}))
```

### Partial Mocks

```typescript
vi.mock('@/lib/utils', async () => {
  const actual = await vi.importActual('@/lib/utils')
  return {
    ...actual,
    specificFunction: vi.fn(),
  }
})
```

---

## Integration Testing

```typescript
describe('Thread Creation Flow', () => {
  it('creates thread end-to-end', async () => {
    const { user } = render(<App />)

    // Navigate to new thread
    await user.click(screen.getByRole('button', { name: /new thread/i }))

    // Enter message
    const input = screen.getByRole('textbox')
    await user.type(input, 'Hello AI')

    // Send message
    await user.click(screen.getByRole('button', { name: /send/i }))

    // Verify message appears
    await waitFor(() => {
      expect(screen.getByText('Hello AI')).toBeInTheDocument()
    })
  })
})
```

---

## Continuous Integration

### GitHub Actions

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: yarn install
      - run: yarn test:coverage
      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

---

## Next Steps

- **Debugging:** [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)
- **Workflow:** [DEVELOPMENT_WORKFLOW.md](./DEVELOPMENT_WORKFLOW.md)
- **Contributing:** [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)

**Questions?** Return to the [Learning Hub](./README.md)
