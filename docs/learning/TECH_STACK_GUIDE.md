# Technology Stack Guide

**Why each technology was chosen and how they work together**

**Documented:** November 18, 2025 | **Tech Stack Research:** [View Here](./TECH_STACK_RESEARCH.md)

---

## Overview

Jan AI uses cutting-edge technologies as of November 2025. This guide explains why each was chosen and how to use them effectively.

✅ **Status**: Project is up-to-date with modern tech (see [Tech Stack Research](./TECH_STACK_RESEARCH.md))

---

## Frontend Technologies

### React 19.0.0 ✅ CURRENT

**What**: UI library for building user interfaces

**Why Chosen**:
- Industry standard for complex UIs
- New Actions API simplifies async state management
- Built-in compiler auto-optimizes components
- Excellent TypeScript support
- Massive ecosystem

**Key Features Used in Jan**:
- **Actions API**: Handle async operations in forms
- **useOptimistic**: Optimistic UI updates
- **Server Components**: Ready for future SSR (not used yet)
- **Hooks**: Modern functional component pattern

**Learn More**: https://react.dev

---

### TypeScript 5.8.3 / 5.9.2 ✅ CURRENT

**What**: Typed superset of JavaScript

**Why Chosen**:
- Catch errors at compile time, not runtime
- Better IDE support (autocomplete, refactoring)
- Self-documenting code through types
- Required for type-safe routing

**Key Features Used**:
- Strict mode enabled
- Deferred imports (5.9)
- Utility types throughout

**Learn More**: https://www.typescriptlang.org

---

### Vite 6.3.2 ✅ CURRENT

**What**: Next-generation frontend build tool

**Why Chosen Over Webpack**:
- 5x faster builds (benchmarked)
- Instant HMR (Hot Module Replacement)
- Native ESM during development
- Optimized production builds

**Configuration**: [web-app/vite.config.ts](../../web-app/vite.config.ts)

**Learn More**: https://vite.dev

---

### TailwindCSS 4.1.4 ✅ CURRENT

**What**: Utility-first CSS framework

**Why Chosen Over CSS-in-JS**:
- 5x faster builds with new engine
- Predictable class names
- Smaller bundle size
- Excellent DX with IntelliSense
- No runtime overhead

**Key Features (Tailwind 4)**:
- CSS-first configuration (@theme directive)
- Built-in container queries
- 3D transforms
- oklch color space (more vivid colors)

**Configuration**: [web-app/tailwind.config.js](../../web-app/tailwind.config.js)

**Learn More**: https://tailwindcss.com

---

### TanStack Router 1.117.0 ✅ CURRENT

**What**: 100% type-safe React router

**Why Chosen Over React Router**:
- Full type safety (autocomplete for routes!)
- File-based routing (like Next.js)
- Built-in code splitting
- Type-safe search params
- Excellent DevTools

**Example**:
```typescript
// Fully type-safe navigation
navigate({ to: '/threads/$threadId', params: { threadId: '123' } })
// TypeScript knows threadId is required!
```

**Learn More**: https://tanstack.com/router

---

### Zustand 5.0.3 ✅ CURRENT

**What**: Lightweight state management

**Why Chosen Over Redux**:
- Minimal boilerplate (1/10th the code)
- No reducers, no actions, no dispatch
- Direct state updates (with Immer-style syntax)
- Smaller bundle (< 1KB)
- Simple to learn

**Example**:
```typescript
const useThreads = create((set) => ({
  threads: [],
  addThread: (thread) => set((state) => ({
    threads: [...state.threads, thread]
  }))
}))
```

**Learn More**: https://zustand.docs.pmnd.rs

---

### Radix UI (Latest) ✅ CURRENT

**What**: Unstyled, accessible component primitives

**Why Chosen**:
- Accessibility built-in (ARIA, keyboard nav)
- Unstyled = full design control
- Works perfectly with Tailwind
- High-quality components

**Components Used**:
- Dialog, Dropdown, Tooltip, Slider, Switch, etc.

**Learn More**: https://www.radix-ui.com

---

## Backend Technologies

### Tauri 2.8.5 ✅ CURRENT

**What**: Framework for building desktop apps with web technologies

**Why Chosen Over Electron**:
- Smaller binaries (10-20x smaller)
- Lower memory usage (Rust vs Chromium)
- Better security (no Node.js in renderer)
- Cross-platform (Windows, macOS, Linux, iOS, Android)
- Rust performance for heavy operations

**Architecture**:
```
React (Renderer) ←IPC→ Rust (Core)
    ↑                      ↑
  WebView              Native APIs
```

**Learn More**: https://v2.tauri.app

---

### Rust 1.77.2 ⚠️ BEHIND (Latest: 1.91.1)

**What**: Systems programming language

**Why Chosen**:
- Memory safety without garbage collection
- Performance (as fast as C++)
- No runtime overhead
- Excellent for LLM inference (llama.cpp bindings)

**⚠️ Recommendation**: Upgrade to Rust 1.91.1 for latest features

**For JS/TS Developers**:
- Like TypeScript but even stricter
- Ownership instead of garbage collection
- `async/await` works similarly
- Pattern matching is powerful

**Learn More**: https://www.rust-lang.org

---

## Build & Test Technologies

### Yarn 4.5.3 ✅ CURRENT

**What**: Package manager with workspace support

**Why Chosen Over npm**:
- Better monorepo support
- Faster installs
- Plug'n'Play (PnP) architecture
- Modern features

**Workspaces**: Manages all packages in one repo

**Learn More**: https://yarnpkg.com

---

### Vitest 3.x ⚠️ ONE BEHIND (Latest: 4.0)

**What**: Vite-native unit testing framework

**Why Chosen Over Jest**:
- 10x faster test runs
- Vite config reuse
- Native ESM support
- Better watch mode

**⚠️ Note**: Vitest 4 released October 2025, consider upgrading

**Learn More**: https://vitest.dev

---

## Additional Technologies

### Framer Motion 12.x

**What**: Animation library for React

**Why**: Declarative animations, great DX

### i18next

**What**: Internationalization framework

**Why**: 12 languages supported, industry standard

### PostHog

**What**: Product analytics

**Why**: Privacy-focused, self-hostable option

---

## Technology Decisions: Why NOT X?

### Why Not React Router?
→ TanStack Router offers full type safety

### Why Not Redux?
→ Zustand is simpler, smaller, and sufficient for our needs

### Why Not Electron?
→ Tauri is lighter, more secure, and faster

### Why Not Webpack?
→ Vite is significantly faster

### Why Not Sass/Less?
→ TailwindCSS provides better DX and performance

---

## Migration Paths

If technologies are outdated, here's how to upgrade:

### Upgrading Rust (High Priority)
```bash
rustup update
```

### Upgrading Vitest 3 → 4 (Medium Priority)
```bash
yarn upgrade vitest@latest
# Review breaking changes: https://vitest.dev/guide/migration
```

---

## Next Steps

- **Understand Architecture**: [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)
- **Learn Patterns**: [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)
- **Start Coding**: [How-To Guide](./HOW_TO_GUIDE.md)

**Questions?** Return to the [Learning Hub](./README.md)
