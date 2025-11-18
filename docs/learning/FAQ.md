# Frequently Asked Questions (FAQ)

**Common questions from developers joining Jan AI**

**Last Updated:** November 2025

---

## Getting Started

### Q: What are the minimum requirements to develop Jan AI?

**A:**
- Node.js ≥ 20.0.0
- Yarn ≥ 1.22.0 (uses 4.5.3 via corepack)
- Rust (for Tauri)
- Make ≥ 3.81
- 8GB+ RAM recommended

See [Getting Started](./GETTING_STARTED.md) for details.

---

### Q: How long does the first build take?

**A:** 5-15 minutes depending on your machine. Subsequent builds are much faster (< 1 minute for incremental changes).

---

### Q: Can I contribute without knowing Rust?

**A:** Yes! Most features are in the React frontend. You can contribute UI improvements, components, routes, and services without touching Rust.

---

## Architecture

### Q: Why use Tauri instead of Electron?

**A:**
- **Smaller** binaries (10-20x smaller)
- **Faster** startup and lower memory usage
- **More secure** (no Node.js in renderer process)
- **Cross-platform** including mobile (iOS/Android)

---

### Q: How does the frontend communicate with Rust?

**A:** Through Tauri's IPC (Inter-Process Communication):
- **Commands**: Frontend → Backend (request/response)
- **Events**: Backend → Frontend (pub/sub, streaming)

See [Architecture Overview](./ARCHITECTURE_OVERVIEW.md).

---

### Q: What's the service layer pattern?

**A:** Services provide a unified interface with multiple implementations:
- `tauri.ts` - Desktop implementation (calls Tauri commands)
- `web.ts` - Browser implementation (calls HTTP APIs)
- `default.ts` - Fallback implementation

This allows the same code to work on desktop and web.

---

## Development

### Q: How do I add a new route?

**A:** Create a file in `web-app/src/routes/`:
```
web-app/src/routes/my-page.tsx → /my-page
web-app/src/routes/settings/my-setting.tsx → /settings/my-setting
```

TanStack Router auto-generates the route tree.

---

### Q: How do I add a Tauri command?

**A:**
1. Add command in Rust: `src-tauri/src/core/*/commands.rs`
2. Register in `src-tauri/src/lib.rs` (`generate_handler![]`)
3. Call from frontend: `invoke('command_name', params)`

See [How-To Guide](./HOW_TO_GUIDE.md).

---

### Q: Why doesn't my code change appear?

**A:**
- **Frontend changes**: Should hot-reload instantly
- **Core/extension changes**: Rebuild with `yarn build:core` or `yarn build:extensions`
- **Rust changes**: Restart `yarn dev`
- **Config changes**: Always require restart

---

### Q: How do I debug Rust code?

**A:**
- Add `println!()` statements
- Check terminal output (Tauri console)
- Use `dbg!()` macro
- See [Debugging Guide](./DEBUGGING_GUIDE.md)

---

## Technology Stack

### Q: Why Zustand instead of Redux?

**A:**
- **Simpler**: No reducers, actions, or dispatch
- **Smaller**: < 1KB vs Redux's larger bundle
- **Faster**: Less boilerplate = faster development
- **Sufficient**: Our state management needs don't require Redux complexity

---

### Q: Is the project using the latest React?

**A:** Yes! React 19.0.0 (stable released December 2024). See [Tech Stack Research](./TECH_STACK_RESEARCH.md).

---

### Q: Why TanStack Router instead of React Router?

**A:**
- **Type safety**: Routes are fully typed (autocomplete!)
- **File-based**: Simpler organization like Next.js
- **Better DX**: Built-in DevTools
- **Modern**: Designed for modern React

---

## Extensions

### Q: What's the difference between native and web extensions?

**A:**
- **Native** (`extensions/`): Full Tauri access (filesystem, hardware)
- **Web** (`extensions-web/`): Browser-only, no native APIs

---

### Q: How do I create a new extension?

**A:** See [Integration Guide](./INTEGRATION_GUIDE.md) for step-by-step instructions.

---

## Testing

### Q: How do I run tests?

**A:**
```bash
yarn test              # Run all tests
yarn test:watch        # Watch mode
yarn test:coverage     # With coverage
```

---

### Q: How do I test Tauri commands?

**A:** Mock the `invoke` function in your tests. See [Testing Guide](./TESTING_GUIDE.md).

---

## Troubleshooting

### Q: "Error: Cannot find module '@janhq/core'"

**A:**
```bash
yarn build:core
yarn dev
```

The core package needs to be built first.

---

### Q: Build fails with Rust errors

**A:**
```bash
cd src-tauri
cargo clean
cd ..
make dev
```

---

### Q: Port 5173 already in use

**A:** Another Vite dev server is running. Kill it:
```bash
# macOS/Linux
lsof -i :5173
kill -9 <PID>

# Windows
netstat -ano | findstr :5173
taskkill /PID <PID> /F
```

---

### Q: Changes not hot-reloading

**A:**
1. Hard refresh (Ctrl+Shift+R)
2. Restart dev server
3. Clear cache: `rm -rf web-app/node_modules/.vite`

---

## Contributing

### Q: How do I find a good first issue?

**A:** Check GitHub issues with the `good-first-issue` label:
https://github.com/janhq/jan/labels/good-first-issue

---

### Q: What's the PR process?

**A:**
1. Fork the repository
2. Create a branch
3. Make changes and test
4. Run `yarn lint`
5. Create PR to `dev` branch
6. Wait for review

See [First Contributions](./FIRST_CONTRIBUTIONS.md).

---

### Q: Do I need to sign a CLA?

**A:** Check [CONTRIBUTING.md](../../CONTRIBUTING.md) for current requirements.

---

## Specific Features

### Q: How does model downloading work?

**A:**
1. User selects model in Hub
2. Frontend calls download service
3. Tauri download manager handles HTTP download
4. Progress emitted via events
5. Model stored in local directory

See [Code Tours](./CODE_TOURS.md) for detailed walkthrough.

---

### Q: How does chat streaming work?

**A:**
1. User sends message
2. llama.cpp plugin processes
3. Emits `message_chunk` events
4. Frontend appends chunks in real-time
5. `message_complete` event signals end

---

### Q: What is MCP?

**A:** Model Context Protocol - allows AI models to interact with external tools/data sources. Jan uses it for file access, database queries, and custom integrations.

Configuration: `Settings → MCP Servers`

---

## Platform-Specific

### Q: Can I develop on Windows?

**A:** Yes! Install:
- Node.js
- Rust (with Microsoft C++ Build Tools)
- Make (via Chocolatey or WSL)

See [Getting Started](./GETTING_STARTED.md#prerequisites).

---

### Q: Does Jan work on mobile?

**A:** Yes! Tauri 2 supports iOS and Android. See:
```bash
make dev-ios       # iOS
make dev-android   # Android
```

---

### Q: Can I build for Linux?

**A:** Yes! Both AppImage and deb packages:
```bash
make build      # Uses build-utils/buildAppImage.sh
```

---

## Performance

### Q: Why is the first build so slow?

**A:** Rust compilation is slow on first build. Subsequent builds use incremental compilation and are much faster.

---

### Q: How do I improve build speed?

**A:**
- Use `yarn dev:web-app` for frontend-only development
- Enable Rust's LLD linker (faster)
- Increase RAM allocated to Rust compiler

---

## Documentation

### Q: Are these docs complete?

**A:** These docs were generated November 18, 2025. Some sections may have uncertainty markers (⚠️ 🔍 ❓) where verification is needed. Contributions welcome!

---

### Q: I found an error in the docs. How do I report it?

**A:**
1. Submit a PR to fix it
2. Open a GitHub issue
3. Ask in Discord `#📚|documentation`

---

### Q: Where's the API reference?

**A:**
- **Core types**: [core/src/types/](../../core/src/types/)
- **Tauri commands**: [src-tauri/src/core/](../../src-tauri/src/core/)
- **Online docs**: https://jan.ai/api-reference

---

## Community

### Q: Where can I ask questions?

**A:**
- **Discord**: https://discord.gg/FTk2MvZwJH (`#🆘|jan-help`)
- **GitHub Discussions**: https://github.com/janhq/jan/discussions
- **GitHub Issues**: For bugs only

---

### Q: How do I stay updated?

**A:**
- **Changelog**: https://jan.ai/changelog
- **Discord announcements**
- **GitHub releases**

---

## Still Have Questions?

**Not answered here?**
1. Search [GitHub Issues](https://github.com/janhq/jan/issues)
2. Ask in [Discord](https://discord.gg/FTk2MvZwJH)
3. Check [GitHub Discussions](https://github.com/janhq/jan/discussions)
4. Review the [Learning Hub](./README.md) for more guides

**Found this helpful?** Consider contributing more Q&As!
