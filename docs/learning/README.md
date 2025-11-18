# Jan AI - Developer Learning Path

**Welcome!** 👋 This is your complete guide to becoming productive on the Jan AI project.

**Documented:** November 18, 2025 | **Tech Stack Research:** [View Here](./TECH_STACK_RESEARCH.md)

---

## 🚀 Quick Start (Day 1)

**Brand new to the project? Start here:**

1. **[Getting Started Guide](./GETTING_STARTED.md)** - Set up your development environment
2. **[Architecture Overview](./ARCHITECTURE_OVERVIEW.md)** - Understand the big picture
3. **[Project Structure](./PROJECT_STRUCTURE.md)** - Navigate the codebase

**Quick facts:**
- Jan AI is an **open-source ChatGPT replacement** for running local LLMs
- Built with **React 19 + Tauri 2 + TypeScript + Rust**
- Monorepo architecture with 5 workspaces (core, web-app, extensions, src-tauri, docs)
- Mobile (iOS/Android) + Desktop (Windows/macOS/Linux) support

---

## 📚 What is Jan AI?

Jan AI enables users to:
- **Run local AI models** (Llama, Gemma, Qwen, GPT-oss, etc.) with full privacy
- **Connect to cloud providers** (OpenAI, Anthropic, Groq, Mistral, etc.)
- **Create custom AI assistants** for specific tasks
- **Use an OpenAI-compatible API** server at `localhost:1337`
- **Integrate with MCP** (Model Context Protocol) for agentic capabilities

**Privacy-first**: Everything runs locally when you want it to.

---

## 🎯 Learning Path by Week

### Week 1: Foundation & Environment

**Goal:** Get the app running locally and understand the architecture

| Day | Focus | Resources |
|-----|-------|-----------|
| 1 | Setup & First Build | [Getting Started](./GETTING_STARTED.md) |
| 2 | Architecture Overview | [Architecture](./ARCHITECTURE_OVERVIEW.md) |
| 3 | Code Structure | [Project Structure](./PROJECT_STRUCTURE.md) |
| 4 | Tech Stack Deep Dive | [Tech Stack Guide](./TECH_STACK_GUIDE.md) |
| 5 | Data Flow | [Data Flow Guide](./DATA_FLOW_GUIDE.md) |

**End of Week 1 Checkpoint:** You can run the app, navigate the codebase, and understand how data flows.

### Week 2: Frontend Deep Dive

**Goal:** Master the React + TypeScript + TanStack Router frontend

| Day | Focus | Resources |
|-----|-------|-----------|
| 6-7 | Frontend Architecture | [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) |
| 8 | Patterns & Conventions | [Patterns](./PATTERNS_AND_CONVENTIONS.md) |
| 9 | How-To Guides | [How-To Guide](./HOW_TO_GUIDE.md) |
| 10 | Code Tours | [Code Tours](./CODE_TOURS.md) - Follow actual code flows |

**End of Week 2 Checkpoint:** You can add new routes, components, and Zustand stores.

### Week 3: Backend & Extensions

**Goal:** Understand Tauri backend and extension system

| Day | Focus | Resources |
|-----|-------|-----------|
| 11-12 | Backend Architecture (Rust/Tauri) | [Backend Architecture](./BACKEND_ARCHITECTURE.md) |
| 13 | Extension System | [Integration Guide](./INTEGRATION_GUIDE.md) |
| 14 | Testing | [Testing Guide](./TESTING_GUIDE.md) |
| 15 | First Contribution | [First Contributions](./FIRST_CONTRIBUTIONS.md) |

**End of Week 3 Checkpoint:** You can add Tauri commands and understand the extension system.

### Week 4+: Mastery & Contribution

**Goal:** Contribute meaningfully to any part of the codebase

- Complete hands-on [Exercises](./EXERCISES.md)
- Deep dive into specific areas (Database, API, Security)
- Tackle real GitHub issues
- Review PRs and help others

---

## 🗺️ Documentation Map

### 📖 Foundation (Start Here)

Essential reading for all developers:

| Document | What You'll Learn | Time |
|----------|-------------------|------|
| [Getting Started](./GETTING_STARTED.md) | Setup, installation, first build | 30 min |
| [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) | System architecture, data flow, components | 45 min |
| [Project Structure](./PROJECT_STRUCTURE.md) | Where everything lives, file organization | 30 min |
| [Tech Stack Guide](./TECH_STACK_GUIDE.md) | All technologies and why they're used | 1 hour |
| [Data Flow Guide](./DATA_FLOW_GUIDE.md) | How data moves through the system | 45 min |

### 🔍 Deep Dives (Technology-Specific)

Choose based on what you're working on:

| Document | When You Need It |
|----------|------------------|
| [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) | Working on React components, routes, state |
| [Backend Architecture](./BACKEND_ARCHITECTURE.md) | Working on Tauri commands, Rust code, plugins |
| [Database Architecture](./DATABASE_ARCHITECTURE.md) | Working on data persistence, models |
| [Integration Guide](./INTEGRATION_GUIDE.md) | Creating extensions or integrating providers |

### 🛠️ Practical Guides (How-To)

Task-oriented guides when you need to do something specific:

| Document | Use It For |
|----------|-----------|
| [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md) | Code style, naming, patterns to follow/avoid |
| [How-To Guide](./HOW_TO_GUIDE.md) | Step-by-step instructions for common tasks |
| [Code Tours](./CODE_TOURS.md) | Guided walkthroughs of actual code flows |
| [Development Workflow](./DEVELOPMENT_WORKFLOW.md) | Git, PRs, testing, deployment |

### ⚙️ Quality & Reference

| Document | Use It For |
|----------|-----------|
| [Testing Guide](./TESTING_GUIDE.md) | Writing and running tests |
| [Debugging Guide](./DEBUGGING_GUIDE.md) | Solving problems, profiling |
| [Security Guide](./SECURITY_GUIDE.md) | Security best practices |
| [API Documentation](./API_DOCUMENTATION.md) | API reference for core, Tauri, services |
| [Database Schema](./DATABASE_SCHEMA.md) | Data models and storage |

### 🎓 Learning & Exercises

| Document | Use It For |
|----------|-----------|
| [Exercises](./EXERCISES.md) | Hands-on practice with solutions |
| [First Contributions](./FIRST_CONTRIBUTIONS.md) | Making your first PR |
| [FAQ](./FAQ.md) | Common questions and answers |

---

## 🏗️ Architecture at a Glance

```
┌─────────────────────────────────────────────────────────────┐
│                         Jan AI Desktop App                   │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────┐         ┌────────────────────┐    │
│  │   React Frontend    │◄───────►│  Tauri Backend    │    │
│  │                     │  IPC    │  (Rust)           │    │
│  │ • TanStack Router   │         │                    │    │
│  │ • Zustand (State)   │         │ • Custom Plugins   │    │
│  │ • TailwindCSS 4     │         │ • HTTP Server      │    │
│  │ • Radix UI          │         │ • MCP Integration  │    │
│  └─────────────────────┘         └────────────────────┘    │
│           │                               │                  │
│           │                               │                  │
│  ┌────────▼────────────┐         ┌───────▼──────────┐     │
│  │  Extension System   │         │   LLM Engines    │     │
│  │                     │         │                   │     │
│  │ • Conversational    │         │ • llama.cpp       │     │
│  │ • RAG               │         │ • Cloud Providers │     │
│  │ • Vector DB         │         │ • MCP Servers     │     │
│  │ • Assistants        │         │                   │     │
│  └─────────────────────┘         └───────────────────┘     │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

**Key Components:**
- **Frontend (web-app/)**: React 19 SPA with type-safe routing
- **Backend (src-tauri/)**: Rust-based Tauri app with custom plugins
- **Core (@janhq/core)**: Shared types and interfaces
- **Extensions**: Modular functionality (native + web)
- **LLM Integration**: llama.cpp for local models, APIs for cloud

[**Detailed architecture →**](./ARCHITECTURE_OVERVIEW.md)

---

## 🔧 Technology Stack (November 2025)

**Status:** ✅ **Project uses cutting-edge, current technologies** | [Full Research →](./TECH_STACK_RESEARCH.md)

### Frontend

| Technology | Version | Status | Why It's Used |
|------------|---------|--------|---------------|
| **React** | 19.0.0 | ✅ Current | Modern UI with new Actions API, compiler |
| **TypeScript** | 5.8.3 / 5.9.2 | ✅ Current | Type safety across the stack |
| **Vite** | 6.3.2 | ✅ Current | Lightning-fast builds (5x faster than v5) |
| **TailwindCSS** | 4.1.4 | ✅ Current | Utility-first styling, 5x faster builds |
| **TanStack Router** | 1.117.0 | ✅ Current | 100% type-safe routing |
| **Zustand** | 5.0.3 | ✅ Current | Lightweight state management |
| **Radix UI** | Latest | ✅ Current | Accessible UI primitives |

### Backend

| Technology | Version | Status | Why It's Used |
|------------|---------|--------|---------------|
| **Tauri** | 2.8.5 | ✅ Current | Cross-platform desktop + mobile |
| **Rust** | 1.77.2 | ⚠️ Behind | Native performance, safety |
| **llama.cpp** | Latest | - | Local LLM inference |

### Build & Test

| Technology | Version | Status | Why It's Used |
|------------|---------|--------|---------------|
| **Yarn** | 4.5.3 | ✅ Current | Workspace management |
| **Vitest** | 3.x | ⚠️ One behind | Fast unit testing |
| **Node.js** | 22.x LTS | ✅ Current | Runtime environment |

**Notes:**
- ⚠️ **Rust 1.77.2**: 14 versions behind latest (1.91.1). Consider upgrading.
- ⚠️ **Vitest 3.x**: Vitest 4.0 released October 2025. Non-urgent upgrade.
- ✅ All other dependencies are current (Nov 2025)

[**Full tech stack analysis →**](./TECH_STACK_GUIDE.md)

---

## 🎨 Key Design Patterns

### Frontend Patterns

✅ **Current patterns in this project:**

1. **Service Layer Pattern**: Business logic in `services/` (not in components)
2. **Container/Presentational Split**: Smart containers + dumb UI components
3. **Custom Hooks**: Reusable logic (`useThreads`, `useModels`, etc.)
4. **Zustand Stores**: Single store per domain (threads, models, settings)
5. **Type-Safe Routing**: TanStack Router with auto-generated types

### Backend Patterns (Rust/Tauri)

1. **Plugin Architecture**: Modular functionality via Tauri plugins
2. **Command/Query Separation**: Tauri commands for operations, events for updates
3. **Event-Driven**: Rust emits events, frontend listens
4. **Async/Await**: Tokio for async Rust operations

[**All patterns explained →**](./PATTERNS_AND_CONVENTIONS.md)

---

## 📁 Repository Structure Quick Reference

```
jan/
├── core/                    # @janhq/core - Core library and types
│   └── src/
│       ├── types/           # TypeScript interfaces & types
│       └── browser/         # Browser-specific implementations
│
├── web-app/                 # React frontend (main UI)
│   └── src/
│       ├── routes/          # TanStack Router routes
│       ├── services/        # Business logic layer
│       ├── stores/          # Zustand state stores (in services/)
│       ├── components/      # Reusable UI components
│       ├── containers/      # Smart container components
│       ├── hooks/           # Custom React hooks
│       ├── lib/             # Utilities and helpers
│       └── locales/         # i18n translations (12 languages)
│
├── extensions-web/          # Web-specific extensions
│   └── src/
│       ├── conversational-web/   # Chat functionality
│       ├── jan-provider-web/     # Provider integration
│       └── mcp-web/              # MCP support
│
├── extensions/              # Native extensions (TypeScript)
│   ├── assistant-extension/      # AI assistants
│   ├── conversational-extension/ # Chat core
│   ├── download-extension/       # Model downloads
│   ├── llamacpp-extension/       # llama.cpp integration
│   ├── rag-extension/            # RAG (Retrieval-Augmented Generation)
│   └── vector-db-extension/      # Vector database
│
├── src-tauri/               # Rust/Tauri backend
│   ├── src/
│   │   ├── main.rs          # Tauri entry point
│   │   ├── lib.rs           # Library exports
│   │   └── core/            # Core backend modules
│   └── plugins/             # Custom Tauri plugins
│       ├── tauri-plugin-llamacpp/    # LLM inference
│       ├── tauri-plugin-vector-db/   # Vector DB
│       ├── tauri-plugin-rag/         # RAG functionality
│       └── tauri-plugin-hardware/    # System info
│
├── autoqa/                  # Python automation/testing
│   ├── tests/               # Automated tests
│   └── scripts/             # Test scripts
│
└── docs/                    # Documentation (Astro site)
    └── learning/            # 👈 YOU ARE HERE
        ├── README.md        # This file
        ├── TECH_STACK_RESEARCH.md
        └── ... (all guides)
```

[**Detailed structure →**](./PROJECT_STRUCTURE.md)

---

## 🚦 Common Tasks (Quick Reference)

Need to do something specific? Jump straight to the how-to:

| Task | Guide | Complexity |
|------|-------|------------|
| **Add a new settings page** | [How-To Guide](./HOW_TO_GUIDE.md#add-a-new-settings-page) | 🟢 Easy |
| **Create a new route** | [How-To Guide](./HOW_TO_GUIDE.md#create-a-new-route) | 🟢 Easy |
| **Add a Zustand store** | [How-To Guide](./HOW_TO_GUIDE.md#add-a-zustand-store) | 🟡 Medium |
| **Create a Tauri command** | [How-To Guide](./HOW_TO_GUIDE.md#create-a-tauri-command) | 🟡 Medium |
| **Add a new extension** | [Integration Guide](./INTEGRATION_GUIDE.md#creating-extensions) | 🔴 Advanced |
| **Integrate a new provider** | [Integration Guide](./INTEGRATION_GUIDE.md#provider-integration) | 🔴 Advanced |
| **Add internationalization** | [How-To Guide](./HOW_TO_GUIDE.md#add-internationalization) | 🟢 Easy |
| **Implement real-time updates** | [How-To Guide](./HOW_TO_GUIDE.md#real-time-updates) | 🟡 Medium |

---

## 🐛 Debugging & Troubleshooting

**Something not working?**

1. **Check the logs**
   - Browser console (frontend errors)
   - Tauri console (backend errors)
   - See [Debugging Guide](./DEBUGGING_GUIDE.md)

2. **Common issues:**
   - Build failures → [Getting Started - Troubleshooting](./GETTING_STARTED.md#troubleshooting)
   - Type errors → [Patterns Guide](./PATTERNS_AND_CONVENTIONS.md#typescript-patterns)
   - State not updating → [Frontend Architecture - Zustand](./FRONTEND_ARCHITECTURE.md#state-management)

3. **Still stuck?**
   - Search GitHub issues
   - Ask in Discord: `#🆘|jan-help`
   - Check [FAQ](./FAQ.md)

---

## 🧪 Testing Your Changes

```bash
# Run all tests
yarn test

# Run tests with coverage
yarn test:coverage

# Run tests in watch mode
yarn test:watch

# Lint code
yarn lint
```

[**Complete testing guide →**](./TESTING_GUIDE.md)

---

## 🤝 Contributing

Ready to contribute? Here's how:

1. **Find an issue**
   - Good first issues: [GitHub label: good-first-issue](https://github.com/janhq/jan/labels/good-first-issue)
   - See [First Contributions](./FIRST_CONTRIBUTIONS.md)

2. **Development workflow**
   - Fork → Branch → Code → Test → PR
   - See [Development Workflow](./DEVELOPMENT_WORKFLOW.md)

3. **Code review**
   - PRs reviewed by maintainers
   - Address feedback promptly
   - See [CONTRIBUTING.md](../../CONTRIBUTING.md)

---

## 📖 Additional Resources

### Official Documentation
- [Jan AI Docs](https://jan.ai/docs) - User-facing documentation
- [API Reference](https://jan.ai/api-reference) - API docs
- [Changelog](https://jan.ai/changelog) - Release notes

### Technology Documentation
- [React 19 Docs](https://react.dev) - Official React docs
- [Tauri Docs](https://v2.tauri.app) - Tauri v2 documentation
- [TanStack Router](https://tanstack.com/router/latest) - Router docs
- [Zustand Docs](https://zustand.docs.pmnd.rs) - State management
- [TailwindCSS](https://tailwindcss.com) - Styling framework

### Community
- [Discord](https://discord.gg/FTk2MvZwJH) - Join the community
- [GitHub Issues](https://github.com/janhq/jan/issues) - Bug reports
- [GitHub Discussions](https://github.com/janhq/jan/discussions) - Q&A

---

## 💡 Learning Tips

**For JavaScript/TypeScript developers new to Rust:**
- Rust concepts are explained in JS/TS terms in [Backend Architecture](./BACKEND_ARCHITECTURE.md)
- Start with reading Tauri commands (they're simpler than full Rust programs)
- Focus on understanding patterns, not writing Rust initially

**For React developers new to TanStack Router:**
- It's similar to React Router but 100% type-safe
- Routes are file-based (like Next.js App Router)
- See [Frontend Architecture](./FRONTEND_ARCHITECTURE.md#routing)

**For developers new to Zustand:**
- Much simpler than Redux (no reducers, actions, dispatchers)
- Direct state mutation (with immer-style syntax)
- See [Frontend Architecture](./FRONTEND_ARCHITECTURE.md#state-management)

---

## 🎯 Success Milestones

Track your progress:

- [ ] **Day 1**: Run the app locally
- [ ] **Week 1**: Understand architecture and can navigate codebase
- [ ] **Week 2**: Make first code change (add a component, route, or store)
- [ ] **Week 3**: Submit first PR (documentation, bug fix, or small feature)
- [ ] **Month 1**: Contribute to core functionality
- [ ] **Month 2+**: Mentor new contributors

---

## 📝 Document Conventions Used

Throughout these learning materials, you'll see these markers:

### Currency Indicators
- ✅ **CURRENT (Nov 2025)** - Pattern/tech is up-to-date
- ⚠️ **OUTDATED PATTERN** - Works but newer approaches exist
- 🚨 **DEPRECATED** - Should not be used in new code
- 🆕 **NEW IN 2025** - Recently introduced feature

### Uncertainty Indicators
- ✅ **VERIFIED** - Confirmed from code/research
- ⚠️ **UNCLEAR** - Not fully understood
- 🔍 **NEEDS VERIFICATION** - Requires investigation
- ❓ **ASSUMPTION** - Inferred, not observed
- 🚧 **TODO** - Placeholder needing work

### Learning Elements
- 🧠 **Mental Model** - Conceptual framework
- 🌉 **Bridge from React/JS** - Familiar analogies
- 💡 **Aha Moment** - Key insight
- 🎯 **Remember This** - Mnemonic or phrase
- ⚠️ **Common Pitfall** - What to avoid
- ✅ **Quick Check** - Self-test question

---

## 🙋 Getting Help

**Stuck? Have questions?**

1. **Search this documentation** - Use browser search (Ctrl/Cmd + F)
2. **Check [FAQ](./FAQ.md)** - Common questions answered
3. **Review [Debugging Guide](./DEBUGGING_GUIDE.md)** - Troubleshooting steps
4. **Ask in Discord** - `#🆘|jan-help` channel
5. **GitHub Discussions** - For deeper technical questions

**Found an error in the docs?**
- These docs were generated and may contain inaccuracies
- Please submit a PR to fix any errors you find
- Uncertainty is marked clearly - help us verify unclear sections!

---

## 📅 Documentation Metadata

**Last Updated:** November 18, 2025
**Tech Stack Verified:** November 18, 2025
**Next Review:** Quarterly or when major versions change

**What's New Since Last Update:**
- Initial comprehensive documentation created
- Tech stack research completed for Nov 2025
- All technologies verified as current (except Rust 1.77 → 1.91)

---

## 🚀 Ready to Start?

**New to the project?** Begin with [Getting Started](./GETTING_STARTED.md)

**Know the basics?** Jump to your area of focus:
- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - React/TypeScript work
- [Backend Architecture](./BACKEND_ARCHITECTURE.md) - Tauri/Rust work
- [Integration Guide](./INTEGRATION_GUIDE.md) - Extensions/Providers

**Want hands-on practice?** Try the [Exercises](./EXERCISES.md)

**Ready to contribute?** Check [First Contributions](./FIRST_CONTRIBUTIONS.md)

---

**Happy coding! 🎉**

If you have questions or need help, the Jan AI community is here for you in [Discord](https://discord.gg/FTk2MvZwJH).
