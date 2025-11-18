# Project Structure

**Complete guide to the Jan AI codebase organization**

**Documented:** November 18, 2025 | **For:** Navigating the monorepo

---

## Overview

Jan AI uses a **monorepo** structure managed by **Yarn 4 Workspaces**. One repository contains multiple independent packages that work together.

🧠 **Mental Model:**
```
Monorepo = Multiple npm packages in one Git repository
Each package can depend on others within the same repo
```

---

## Root Directory Structure

```
jan/
├── #claude/                # Claude Code configuration
├── .devcontainer/          # Dev container configuration
├── .github/                # GitHub workflows and CI/CD
├── .husky/                 # Git hooks (pre-commit, etc.)
├── autoqa/                 # Python automation/testing 🐍
├── core/                   # @janhq/core - Core library ⚛️
├── docs/                   # Documentation site (Astro) 📚
├── extensions/             # Native extensions (TypeScript) 🔌
├── extensions-web/         # Web-specific extensions 🌐
├── flatpak/                # Linux Flatpak packaging
├── pre-install/            # Pre-installed extension .tgz files
├── scripts/                # Build and utility scripts
├── src-tauri/              # Rust/Tauri backend 🦀
├── tests/                  # Integration tests
├── web-app/                # React frontend application ⚛️
├── Makefile                # Build automation
├── package.json            # Root workspace configuration
└── yarn.lock               # Dependency lock file
```

---

## Core Workspace (`core/`)

**Purpose**: Shared TypeScript library with types and interfaces

```
core/
├── dist/                   # Compiled output (generated)
├── src/
│   ├── @global/            # Global type declarations
│   ├── browser/            # Browser-specific implementations
│   ├── test/               # Test utilities
│   └── types/              # TypeScript type definitions ⭐
│       ├── model.ts        # Model interfaces
│       ├── thread.ts       # Thread/message types
│       ├── assistant.ts    # Assistant types
│       ├── extension.ts    # Extension interfaces
│       └── ...
├── package.json            # Package dependencies
├── tsconfig.json           # TypeScript configuration
└── rolldown.config.mjs     # Rolldown bundler config
```

**Key Files:**
- [types/model.ts](../../core/src/types/model.ts) - Model definitions
- [types/thread.ts](../../core/src/types/thread.ts) - Chat thread types
- [types/extension.ts](../../core/src/types/extension.ts) - Extension interface

**Build Output**: `core/dist/` (bundled JavaScript)

💡 **Aha Moment:** This package is the "source of truth" for types. All other packages import from `@janhq/core`.

---

## Frontend Workspace (`web-app/`)

**Purpose**: React 19 single-page application

```
web-app/
├── dist/                   # Production build (generated)
├── public/                 # Static assets
├── src/
│   ├── __tests__/          # Root-level tests
│   ├── components/         # Reusable UI components ⭐
│   │   ├── ui/             # Radix UI + shadcn components
│   │   └── ...
│   ├── constants/          # App constants
│   ├── consts/             # Additional constants
│   ├── containers/         # Smart container components ⭐
│   │   ├── analytics/      # Analytics containers
│   │   ├── auth/           # Authentication UI
│   │   ├── dialogs/        # Modal dialogs
│   │   ├── dynamicControllerSetting/
│   │   ├── loaders/        # Loading states
│   │   └── LeftPanel/      # Main left sidebar
│   ├── hooks/              # Custom React hooks ⭐
│   │   ├── useThreads.ts
│   │   ├── useModels.ts
│   │   ├── useAnalytic.ts
│   │   └── ...
│   ├── i18n/               # Internationalization setup
│   ├── lib/                # Utility libraries ⭐
│   │   ├── platform/       # Platform detection (Tauri vs Web)
│   │   ├── shortcuts/      # Keyboard shortcuts
│   │   └── utils.ts        # Helper functions
│   ├── locales/            # Translation files (12 languages) 🌍
│   │   ├── en/             # English
│   │   ├── zh-CN/          # Chinese (Simplified)
│   │   ├── ja/             # Japanese
│   │   └── ...
│   ├── providers/          # React context providers ⭐
│   │   ├── ThemeProvider.tsx
│   │   ├── DataProvider.tsx
│   │   ├── ExtensionProvider.tsx
│   │   └── ...
│   ├── routes/             # TanStack Router pages ⭐⭐⭐
│   │   ├── __root.tsx      # Root layout
│   │   ├── index.tsx       # Home page (/)
│   │   ├── threads/        # Chat routes
│   │   │   └── $threadId.tsx  # /threads/:threadId
│   │   ├── hub/            # Model hub routes
│   │   │   ├── index.tsx   # /hub
│   │   │   └── $modelId.tsx # /hub/:modelId
│   │   ├── project/        # Project routes
│   │   │   ├── index.tsx
│   │   │   └── $projectId.tsx
│   │   ├── settings/       # Settings pages
│   │   │   ├── general.tsx
│   │   │   ├── assistant.tsx
│   │   │   ├── extensions.tsx
│   │   │   ├── hardware.tsx
│   │   │   ├── mcp-servers.tsx
│   │   │   ├── providers/
│   │   │   └── ...
│   │   ├── local-api-server/
│   │   ├── logs.tsx
│   │   └── system-monitor.tsx
│   ├── services/           # Business logic layer ⭐⭐⭐
│   │   ├── analytic/       # Analytics service
│   │   ├── app/            # App configuration
│   │   │   ├── types.ts    # Service interface
│   │   │   ├── tauri.ts    # Tauri implementation
│   │   │   ├── web.ts      # Web implementation
│   │   │   └── default.ts  # Fallback implementation
│   │   ├── assistants/     # Assistant management
│   │   ├── core/           # Core service utilities
│   │   ├── deeplink/       # Deep link handling
│   │   ├── dialog/         # Dialog service
│   │   ├── events/         # Event system (Tauri events)
│   │   ├── hardware/       # Hardware info service
│   │   ├── mcp/            # MCP service
│   │   ├── messages/       # Message handling
│   │   ├── models/         # Model management
│   │   ├── opener/         # File/URL opener
│   │   ├── path/           # Path utilities
│   │   ├── projects/       # Project management
│   │   ├── providers/      # Provider integration
│   │   ├── rag/            # RAG service
│   │   ├── theme/          # Theme service
│   │   ├── threads/        # Thread/chat service
│   │   ├── updater/        # App updater
│   │   ├── uploads/        # File uploads
│   │   └── window/         # Window management
│   ├── styles/             # Global styles
│   ├── test/               # Test utilities and mocks
│   ├── types/              # Frontend-specific types
│   ├── utils/              # Utility functions
│   ├── index.css           # Global CSS
│   ├── main.tsx            # App entry point ⭐
│   ├── routeTree.gen.ts    # Generated route tree (TanStack Router)
│   └── vite-env.d.ts       # Vite type declarations
├── index.html              # HTML entry point
├── package.json            # Frontend dependencies
├── tailwind.config.js      # TailwindCSS configuration
├── tsconfig.json           # TypeScript configuration
├── vite.config.ts          # Vite bundler configuration ⭐
└── vitest.config.ts        # Vitest test configuration
```

**Entry Point Flow:**
```
index.html
  → main.tsx (React root)
    → __root.tsx (Root layout with providers)
      → routes/* (Individual pages)
```

**Key Directories:**
- **routes/**: File-based routing (like Next.js)
- **services/**: Business logic (NOT in components)
- **containers/**: Complex UI components with logic
- **components/ui/**: Dumb, reusable UI components

🎯 **Remember This:**
```
Route → Service → Tauri Command → Rust → Response → Store → Re-render
```

---

## Backend Workspace (`src-tauri/`)

**Purpose**: Rust/Tauri desktop application backend

```
src-tauri/
├── build-utils/            # Build scripts (Linux AppImage, etc.)
├── capabilities/           # Tauri security capabilities
├── gen/                    # Generated files
│   ├── android/            # Android project (generated)
│   └── ios/                # iOS project (generated)
├── icons/                  # App icons
├── plugins/                # Custom Tauri plugins ⭐⭐
│   ├── tauri-plugin-hardware/
│   │   ├── src/
│   │   │   └── lib.rs
│   │   └── Cargo.toml
│   ├── tauri-plugin-llamacpp/   # LLM inference
│   │   ├── src/
│   │   └── Cargo.toml
│   ├── tauri-plugin-rag/        # RAG functionality
│   ├── tauri-plugin-vector-db/  # Vector database
│   └── ...
├── resources/              # Bundled resources
│   ├── pre-install/        # Extension .tgz files
│   └── LICENSE
├── src/                    # Rust source code ⭐⭐⭐
│   ├── core/               # Core backend modules
│   │   ├── app/            # App configuration
│   │   │   ├── commands.rs # Tauri commands for app
│   │   │   ├── models.rs   # Data models
│   │   │   └── mod.rs
│   │   ├── downloads/      # Download management
│   │   ├── extensions/     # Extension loading
│   │   ├── filesystem/     # File operations
│   │   ├── mcp/            # MCP integration
│   │   │   ├── commands.rs
│   │   │   ├── helpers.rs
│   │   │   └── models.rs
│   │   ├── server/         # HTTP API server
│   │   ├── system/         # System operations
│   │   ├── threads/        # Thread management
│   │   ├── mod.rs          # Module exports
│   │   ├── setup.rs        # App setup logic
│   │   └── state.rs        # Global app state
│   ├── lib.rs              # Library entry point ⭐
│   └── main.rs             # Executable entry point
├── static/                 # Static web assets
├── utils/                  # Rust utilities
│   └── src/
│       └── lib.rs
├── Cargo.toml              # Rust dependencies ⭐
├── Cargo.lock              # Dependency lock file
├── build.rs                # Build script
└── tauri.conf.json         # Tauri configuration ⭐
```

**Rust Module Structure:**
```
src-tauri/src/
├── lib.rs                  # Tauri app builder, plugin registration
├── main.rs                 # Desktop executable entry point
└── core/
    ├── mod.rs              # Re-exports all modules
    ├── app/                # Each module follows this pattern:
    │   ├── mod.rs          #   - mod.rs: Public API
    │   ├── commands.rs     #   - commands.rs: Tauri commands
    │   └── models.rs       #   - models.rs: Data structures
    └── ...
```

**Key Files:**
- [src/lib.rs](../../src-tauri/src/lib.rs) - Tauri app configuration, command registration
- [tauri.conf.json](../../src-tauri/tauri.conf.json) - App metadata, window config, security
- [Cargo.toml](../../src-tauri/Cargo.toml) - Rust dependencies

**Build Output**: `target/` (Rust binaries and app bundles)

🌉 **Bridge from Node.js:**
- `src/lib.rs` = Express app setup
- `core/*/commands.rs` = Express route handlers
- `Cargo.toml` = package.json
- `target/` = node_modules + dist

---

## Extensions Workspace (`extensions/`)

**Purpose**: Native TypeScript extensions loaded by the app

```
extensions/
├── assistant-extension/         # AI assistant management
│   ├── dist/                    # Compiled output
│   ├── src/
│   │   └── index.ts             # Extension entry point
│   ├── package.json
│   └── tsconfig.json
├── conversational-extension/    # Chat functionality
│   ├── dist/
│   ├── src/
│   └── package.json
├── download-extension/          # Model download logic
├── llamacpp-extension/          # llama.cpp integration
├── rag-extension/               # RAG (Retrieval-Augmented Generation)
├── vector-db-extension/         # Vector database
└── package.json                 # Workspace root
```

**Extension Build Process:**
```
yarn build:extensions
  → Each extension builds to dist/
    → Packaged as .tgz
      → Copied to pre-install/
        → Loaded by app on startup
```

**Extension Entry Point Example:**
```typescript
// extensions/assistant-extension/src/index.ts
import { Extension } from '@janhq/core'

export default class AssistantExtension implements Extension {
  async onLoad() {
    // Initialize extension
  }

  async onUnload() {
    // Cleanup
  }

  // Extension-specific methods
}
```

---

## Web Extensions Workspace (`extensions-web/`)

**Purpose**: Web-only extensions (no Tauri access)

```
extensions-web/
├── dist/                        # Compiled output
├── src/
│   ├── conversational-web/      # Chat for web version
│   │   ├── index.ts
│   │   └── ...
│   ├── jan-provider-web/        # Provider integration (web)
│   ├── mcp-web/                 # MCP support (web)
│   ├── shared/                  # Shared utilities
│   └── types/                   # Type definitions
├── package.json
└── vite.config.ts
```

**Difference from Native Extensions:**
- Native: Can call Tauri commands (file system, hardware)
- Web: Browser-only, uses Web APIs

---

## Documentation Workspace (`docs/`)

**Purpose**: Astro-based documentation website

```
docs/
├── public/                      # Static assets
├── src/
│   ├── assets/
│   ├── components/              # Astro components
│   ├── helpers/
│   ├── hooks/
│   ├── lib/
│   ├── pages/                   # Documentation pages
│   ├── styles/
│   ├── types/
│   └── utils/
├── static/
├── learning/                    # 👈 YOU ARE HERE
│   ├── README.md                # Learning path hub
│   ├── GETTING_STARTED.md
│   ├── ARCHITECTURE_OVERVIEW.md
│   └── ... (all learning docs)
└── package.json
```

---

## Automation Workspace (`autoqa/`)

**Purpose**: Python-based automation and testing

```
autoqa/
├── scripts/                     # Test scripts
├── tests/                       # Automated tests
│   └── new-user/                # New user onboarding tests
├── requirements.txt             # Python dependencies
└── ...
```

**Technologies:**
- Python 🐍
- cua-computer / cua-agent (automation frameworks)
- opencv-python (screen recording)
- PyAutoGUI (UI automation)

---

## Configuration Files (Root)

| File | Purpose |
|------|---------|
| **package.json** | Root workspace configuration, scripts |
| **Makefile** | Build automation (make dev, make build) |
| **yarn.lock** | Dependency versions lock file |
| **.prettierrc** | Code formatting rules |
| **.prettierignore** | Files to skip formatting |
| **.gitignore** | Files to exclude from Git |
| **.yarnrc.yml** | Yarn configuration |
| **vitest.config.ts** | Root test configuration |
| **tsconfig.json** | Root TypeScript configuration |

---

## Build Artifacts (Generated)

**⚠️ Never commit these directories:**

```
# Frontend
web-app/dist/                    # Vite production build
web-app/node_modules/.vite/      # Vite cache

# Backend
src-tauri/target/                # Rust compiled binaries
src-tauri/gen/                   # Generated mobile projects

# Extensions
extensions/*/dist/               # Compiled extensions
pre-install/*.tgz                # Packaged extensions

# Dependencies
node_modules/                    # npm/yarn dependencies
**/node_modules/                 # Workspace dependencies

# Other
.DS_Store                        # macOS files
*.log                            # Log files
```

---

## Finding Files by Purpose

### "Where do I find...?"

| What | Where |
|------|-------|
| **React components** | `web-app/src/components/` |
| **Route pages** | `web-app/src/routes/` |
| **Business logic** | `web-app/src/services/` |
| **State stores** | `web-app/src/services/*/store.ts` |
| **Custom hooks** | `web-app/src/hooks/` |
| **Type definitions** | `core/src/types/` |
| **Tauri commands** | `src-tauri/src/core/*/commands.rs` |
| **Rust plugins** | `src-tauri/plugins/` |
| **Extensions** | `extensions/` |
| **Tests** | `**/__tests__/` or `*.test.ts` |
| **Translations** | `web-app/src/locales/` |
| **Icons/assets** | `web-app/public/` or `src-tauri/icons/` |
| **Config files** | Root directory |
| **Build scripts** | `scripts/` or `Makefile` |

---

## Naming Conventions

### Files

```
ComponentName.tsx         # React components (PascalCase)
useSomething.ts           # React hooks (camelCase, prefix "use")
something.service.ts      # Services (camelCase + .service)
types.ts                  # Type definitions
constants.ts              # Constants
index.ts                  # Barrel export (re-exports from directory)
*.test.tsx                # Test files
__tests__/                # Test directory
```

### Directories

```
kebab-case                # Preferred (multi-word directories)
camelCase                 # Acceptable (single-concept directories)
PascalCase                # For components matching file name
```

---

## Import Paths

### Aliases (TypeScript)

```typescript
// Defined in tsconfig.json

// web-app imports:
import { Button } from '@/components/ui/button'
import { useThreads } from '@/hooks/useThreads'
import { appService } from '@/services/app'

// Resolves to:
// @ = web-app/src/
```

### Workspace Imports

```typescript
// Any workspace can import core:
import { Model, Thread } from '@janhq/core'

// Web extensions import:
import { ExtensionWeb } from '@jan/extensions-web'
```

---

## Next Steps

Now that you know where everything is:

- **Explore Code**: [Code Tours](./CODE_TOURS.md) - Follow actual code flows
- **Understand Tech**: [Tech Stack Guide](./TECH_STACK_GUIDE.md) - Why each technology
- **See Data Flow**: [Data Flow Guide](./DATA_FLOW_GUIDE.md) - How data moves
- **Start Coding**: [How-To Guide](./HOW_TO_GUIDE.md) - Common tasks

---

**Quick Reference Card:**

```
Frontend Code:    web-app/src/routes/*.tsx
Business Logic:   web-app/src/services/
Types:            core/src/types/
Backend Code:     src-tauri/src/core/
Tauri Commands:   src-tauri/src/core/*/commands.rs
Plugins:          src-tauri/plugins/
Extensions:       extensions/
Tests:            **/__tests__/*.test.tsx
Config:           Root directory
```

**Questions?** Return to the [Learning Hub](./README.md) or check the [FAQ](./FAQ.md).
