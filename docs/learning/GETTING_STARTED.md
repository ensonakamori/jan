# Getting Started with Jan AI Development

**Your first day on the project** - Set up your development environment and run the app locally.

**Documented:** November 18, 2025 | **For:** New developers joining the project

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Your First Build](#your-first-build)
- [Running the App](#running-the-app)
- [Making Your First Code Change](#making-your-first-code-change)
- [Development Workflow](#development-workflow)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

---

## Prerequisites

### Required Software

Before you begin, ensure you have these installed:

#### 1. **Node.js** (≥ v20.0.0)

✅ **CURRENT (Nov 2025):** The project uses Node.js 22.x LTS

```bash
# Check your version
node --version  # Should be v20.0.0 or higher

# Download from:
# https://nodejs.org (get the LTS version)
```

🧠 **Mental Model:** Node.js is the JavaScript runtime that powers the build tools and development server.

#### 2. **Yarn** (≥ 1.22.0, uses 4.5.3 via Corepack)

✅ **CURRENT (Nov 2025):** The project uses Yarn 4.5.3

```bash
# Enable Corepack (comes with Node.js 16.10+)
corepack enable
corepack prepare yarn@4.5.3 --activate

# Verify
yarn --version  # Should show 4.5.3
```

🧠 **Mental Model:** Yarn is the package manager that handles dependencies across all workspaces (core, web-app, extensions, etc.).

🌉 **Bridge from npm:** If you're used to npm, Yarn is similar but with workspace support built-in. Think of it as npm with better monorepo handling.

#### 3. **Make** (≥ 3.81)

```bash
# Check your version
make --version

# Installation:
# - macOS: Comes with Xcode Command Line Tools
# - Linux: Usually pre-installed (sudo apt install build-essential)
# - Windows: Install via Chocolatey (choco install make) or use WSL
```

🧠 **Mental Model:** Make orchestrates complex build commands. `make dev` runs multiple yarn commands in the right order.

#### 4. **Rust** (for Tauri)

⚠️ **NOTE:** Project uses Rust 1.77.2, but latest is 1.91.1 (Nov 2025). Either version works.

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Verify
rustc --version  # Should be 1.77.2 or higher
cargo --version
```

🌉 **Bridge from JavaScript:** Rust compiles to native binaries (like C++ but safer). Tauri uses it for the desktop app shell and native integrations.

**Platform-specific Rust requirements:**

- **macOS**: Install Xcode Command Line Tools
  ```bash
  xcode-select --install
  ```

- **Linux**: Install additional dependencies
  ```bash
  sudo apt-get update
  sudo apt-get install libwebkit2gtk-4.1-dev \
    build-essential \
    curl \
    wget \
    file \
    libxdo-dev \
    libssl-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev
  ```

- **Windows**: Install Microsoft C++ Build Tools
  - Download from: https://visualstudio.microsoft.com/visual-cpp-build-tools/
  - Select "Desktop development with C++"

### Optional (Recommended)

- **Git** (version control) - Already installed if you cloned the repo
- **VS Code** with extensions:
  - Rust Analyzer
  - Tauri
  - ESLint
  - Prettier
  - Tailwind CSS IntelliSense

---

## Installation

### Step 1: Clone the Repository

```bash
git clone https://github.com/janhq/jan.git
cd jan
```

🎯 **Remember This:** You're in a **monorepo**. One repo contains multiple packages (core, web-app, extensions, src-tauri).

### Step 2: Configure Yarn

```bash
# Enable corepack and set yarn version
make config-yarn
```

This command:
- Enables corepack (Node.js package manager manager)
- Sets Yarn version to 4.5.3
- Configures Yarn settings

### Step 3: Install Dependencies & Build

**Option A: Quick Start (Recommended)**

```bash
make dev
```

This single command:
1. Runs `make install-and-build`:
   - Installs all workspace dependencies (`yarn install`)
   - Builds Tauri plugin APIs (`yarn build:tauri:plugin:api`)
   - Builds core package (`yarn build:core`)
   - Builds all extensions (`yarn build:extensions`)
2. Downloads binary dependencies (`yarn download:bin`)
3. Starts the development server (`yarn dev`)

💡 **Aha Moment:** `make dev` is your one-stop command for "get everything running." Bookmark it!

**Option B: Manual Step-by-Step**

If you want to understand each step:

```bash
# 1. Install dependencies in all workspaces
yarn install

# 2. Build Tauri plugin APIs
yarn build:tauri:plugin:api

# 3. Build the core package (@janhq/core)
yarn build:core

# 4. Build all extensions
yarn build:extensions

# 5. Download binary dependencies (llama.cpp, etc.)
yarn download:bin

# 6. Start development server
yarn dev
```

🧠 **Mental Model:** The build order matters because:
- `core` provides types used by `web-app` and `extensions`
- `extensions` are loaded by the main app
- `tauri plugins` need to be built before the Tauri app

⏱️ **Expected Time:** First build takes 5-15 minutes depending on your machine.

---

## Your First Build

### What Happens During Build?

```
┌─────────────────────────────────────────────────────┐
│         make dev (or make install-and-build)        │
├─────────────────────────────────────────────────────┤
│                                                       │
│  1. yarn install                                     │
│     ├─ Install dependencies for root                │
│     ├─ Install dependencies for core/               │
│     ├─ Install dependencies for web-app/            │
│     ├─ Install dependencies for extensions-web/     │
│     └─ Install dependencies for extensions/*        │
│                                                       │
│  2. yarn build:tauri:plugin:api                     │
│     └─ Build Tauri plugin TypeScript bindings       │
│                                                       │
│  3. yarn build:core                                  │
│     ├─ Compile TypeScript → JavaScript              │
│     ├─ Bundle with Rolldown                         │
│     └─ Generate .tgz package                        │
│                                                       │
│  4. yarn build:extensions                           │
│     └─ Build each extension in extensions/          │
│                                                       │
│  5. yarn download:bin (if running make dev)         │
│     └─ Download platform-specific binaries          │
│                                                       │
│  6. yarn dev (if running make dev)                  │
│     ├─ Start Vite dev server (frontend)            │
│     └─ Start Tauri in development mode (backend)    │
│                                                       │
└─────────────────────────────────────────────────────┘
```

### Build Artifacts

After building, you'll see:

```
jan/
├── core/dist/              # Compiled core package
├── web-app/dist/           # Compiled frontend (production only)
├── extensions/*/dist/      # Compiled extensions
├── src-tauri/target/       # Rust compiled binaries
└── node_modules/           # Dependencies (in each workspace)
```

---

## Running the App

### Development Mode

```bash
# Quick start (from project root)
make dev

# Or manually:
yarn dev
```

**What you'll see:**
1. Terminal shows Vite dev server starting (frontend)
2. Rust compilation output (backend)
3. App window opens automatically

**Development features:**
- ✅ **Hot reload**: Frontend changes reload instantly
- ✅ **Fast refresh**: React components update without losing state
- ✅ **Auto-reload**: Some Rust changes trigger automatic rebuild
- ✅ **DevTools**: Browser DevTools available in the app

🎯 **Remember This:** Keep the terminal open while developing. It shows errors and logs.

### Web-Only Mode (Faster for Frontend Development)

If you're only working on the UI and don't need Tauri:

```bash
make dev-web-app
# or
yarn dev:web-app
```

This starts only the Vite dev server at `http://localhost:5173`.

⚠️ **Common Pitfall:** Web-only mode doesn't have access to Tauri commands (like file system access). Use for UI-only work.

### Production Build

```bash
make build
# or
yarn build
```

Creates platform-specific installers in `src-tauri/target/release/bundle/`.

---

## Making Your First Code Change

Let's make a simple change to verify your setup works!

### Exercise: Change the App Title

**Step 1:** Find the file

```bash
# Open in your editor
# File: src-tauri/tauri.conf.json
```

**Step 2:** Make the change

Find line with `"title"` under `windows`:

```json
{
  "app": {
    "windows": [
      {
        "title": "Jan",  // Change this
        ...
      }
    ]
  }
}
```

Change to:

```json
"title": "Jan - My Development Build",
```

**Step 3:** See the change

If `yarn dev` is running, the app should auto-reload. If not, restart:

```bash
# Stop the dev server (Ctrl+C)
# Restart
yarn dev
```

✅ **Quick Check:** Do you see the new title in the app window?

🎯 **Remember This:** Configuration changes (JSON files) usually require a full restart. Code changes often hot-reload.

### Exercise 2: Change Frontend Text

**Step 1:** Open a React component

```bash
# File: web-app/src/routes/index.tsx
```

**Step 2:** Find any text in the component and change it

Example: Change a button label, heading, or description

**Step 3:** Save the file

Watch the app hot-reload instantly! ⚡

💡 **Aha Moment:** Frontend changes are instant. Rust changes take longer to compile.

---

## Development Workflow

### Daily Development Cycle

```bash
# Morning: Pull latest changes
git pull origin dev

# Install any new dependencies
yarn install

# Rebuild if needed (after pulling changes)
yarn build:core
yarn build:extensions

# Start development
yarn dev

# Make changes, test, commit
# ...

# Evening: Push your work
git push origin your-branch
```

### Useful Commands

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `make dev` | Full dev setup + start | First time each day |
| `yarn dev` | Start dev server | After initial build |
| `yarn dev:web-app` | Frontend only | UI work without Tauri |
| `yarn build:core` | Rebuild core | After pulling core changes |
| `yarn build:extensions` | Rebuild extensions | After pulling extension changes |
| `yarn test` | Run tests | Before committing |
| `yarn lint` | Check code style | Before committing |
| `make clean` | Delete everything | When things are broken |

### File Watching

Vite watches these for hot reload:
- ✅ `web-app/src/**/*.tsx`
- ✅ `web-app/src/**/*.ts`
- ✅ `web-app/src/**/*.css`

Tauri watches these for auto-rebuild:
- ⚠️ `src-tauri/src/**/*.rs` (sometimes requires manual restart)

❌ Not watched (manual rebuild needed):
- `core/src/**/*` - Run `yarn build:core`
- `extensions/*/src/**/*` - Run `yarn build:extensions`

🎯 **Remember This:** If something doesn't hot-reload, try restarting `yarn dev`.

---

## Troubleshooting

### Issue: `make: command not found`

**Solution:**
- **macOS**: Install Xcode Command Line Tools: `xcode-select --install`
- **Linux**: `sudo apt install build-essential`
- **Windows**: Install via Chocolatey or use WSL

### Issue: `yarn: command not found`

**Solution:**
```bash
corepack enable
corepack prepare yarn@4.5.3 --activate
```

### Issue: Rust compilation errors

**Solution:**
```bash
# Update Rust
rustup update

# Clean Rust build cache
cd src-tauri
cargo clean
cd ..

# Rebuild
make dev
```

### Issue: "Module not found" errors

**Solution:**
```bash
# Rebuild core package
yarn build:core

# Rebuild extensions
yarn build:extensions

# If still failing, clean install
make clean
make dev
```

### Issue: Port already in use

**Solution:**
```bash
# Find process using port 5173 (Vite default)
# macOS/Linux:
lsof -i :5173

# Windows:
netstat -ano | findstr :5173

# Kill the process and restart
```

### Issue: App window doesn't open

**Solution:**
1. Check terminal for errors
2. Look for "Listening on http://localhost:5173"
3. Try opening manually: http://localhost:5173 in browser (web mode)
4. Check if another Jan instance is running

### Issue: Changes not appearing

**Solution:**
1. **Hard refresh** in the app (Ctrl+Shift+R / Cmd+Shift+R)
2. **Restart dev server** (Ctrl+C, then `yarn dev`)
3. **Clear cache**: `make clean && make dev`
4. **Check if file is watched**: See [File Watching](#file-watching) above

### Issue: "Error: Cannot find module '@janhq/core'"

**Solution:**
```bash
# Core package isn't built
yarn build:core

# Restart dev server
yarn dev
```

### Issue: Vite server starts but shows blank page

**Solution:**
```bash
# Clear Vite cache
rm -rf web-app/node_modules/.vite

# Restart
yarn dev
```

### Still Stuck?

1. **Check the full error message** in the terminal
2. **Search GitHub issues**: https://github.com/janhq/jan/issues
3. **Ask in Discord**: `#🆘|jan-help` channel
4. **Read the detailed guides**:
   - [Debugging Guide](./DEBUGGING_GUIDE.md)
   - [Development Workflow](./DEVELOPMENT_WORKFLOW.md)

---

## Next Steps

✅ **You've completed setup!** Here's what to do next:

### Immediate Next Steps (Today)

1. **Explore the app**
   - Open different settings tabs
   - Try the chat interface
   - Look at the model hub

2. **Read the architecture docs**
   - [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) - Understand the big picture
   - [Project Structure](./PROJECT_STRUCTURE.md) - Know where everything lives

### This Week

3. **Understand the tech stack**
   - [Tech Stack Guide](./TECH_STACK_GUIDE.md) - Why each technology?
   - [Data Flow Guide](./DATA_FLOW_GUIDE.md) - How data moves

4. **Dive into code**
   - [Code Tours](./CODE_TOURS.md) - Follow real code flows
   - [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - React/TypeScript deep dive

### Next Week

5. **Start contributing**
   - [How-To Guide](./HOW_TO_GUIDE.md) - Common tasks
   - [First Contributions](./FIRST_CONTRIBUTIONS.md) - Your first PR
   - [Exercises](./EXERCISES.md) - Hands-on practice

---

## Quick Reference

### One-Liners

```bash
# Fresh start (clean everything and rebuild)
make clean && make dev

# Just start dev server (after initial build)
yarn dev

# Run tests
yarn test

# Lint code
yarn lint

# Build for production
make build

# Web app only (no Tauri)
make dev-web-app
```

### Directory Quick Links

- Frontend code: [web-app/src/](../../web-app/src/)
- Core library: [core/src/](../../core/src/)
- Tauri backend: [src-tauri/src/](../../src-tauri/src/)
- Extensions: [extensions/](../../extensions/)
- Documentation: [docs/](../../docs/)

### Important Files

- [package.json](../../package.json) - Root package with workspace configuration
- [Makefile](../../Makefile) - Build commands
- [src-tauri/tauri.conf.json](../../src-tauri/tauri.conf.json) - Tauri app configuration
- [web-app/vite.config.ts](../../web-app/vite.config.ts) - Vite configuration

---

## Additional Resources

- **Main README**: [../../README.md](../../README.md) - Project overview
- **Contributing Guide**: [../../CONTRIBUTING.md](../../CONTRIBUTING.md) - Contribution guidelines
- **Official Docs**: https://jan.ai/docs - User documentation
- **Discord**: https://discord.gg/FTk2MvZwJH - Community help

---

**Congratulations! 🎉** You're now set up and ready to start developing on Jan AI.

**Next:** Read the [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) to understand how everything fits together.

---

**Questions?** Check the [FAQ](./FAQ.md) or ask in Discord `#🆘|jan-help`.
