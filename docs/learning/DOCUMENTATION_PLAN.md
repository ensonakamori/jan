# Documentation Plan for Jan AI Project

**Created:** November 18, 2025
**Purpose:** Complete learning path for mid-level developers joining the project

---

## Project Overview

**Jan AI** is an open-source ChatGPT replacement that enables users to run local LLMs with full privacy and control. The project is built as a modern desktop application using:

- **Frontend**: React 19 + TypeScript + TanStack Router + TailwindCSS 4
- **Desktop**: Tauri 2 (Rust) with custom plugins
- **Architecture**: Monorepo with Yarn workspaces
- **Key Features**: Local AI models, cloud integration, MCP support, OpenAI-compatible API

---

## Repository Structure Analysis

### Monorepo Workspaces

```
jan/
├── core/                   # @janhq/core - Core library and types
├── web-app/                # React frontend application
├── extensions-web/         # Web-specific extensions
├── extensions/             # Native extensions (llama.cpp, RAG, etc.)
├── src-tauri/              # Rust/Tauri backend
├── autoqa/                 # Python automation/testing
└── docs/                   # Project documentation (Astro-based)
```

### Key Architectural Components

1. **Frontend (web-app/)**
   - TanStack Router for type-safe routing
   - Zustand for state management
   - Service layer pattern
   - Container components + UI components (Radix UI)
   - i18next for internationalization

2. **Backend (src-tauri/)**
   - Tauri 2.8 desktop framework
   - Custom Rust plugins (llamacpp, vector-db, RAG, hardware)
   - MCP integration (rmcp crate)
   - HTTP server (OpenAI-compatible API)

3. **Extension System**
   - Native extensions (TypeScript): assistant, conversational, download, llamacpp, RAG, vector-db
   - Web extensions: conversational-web, jan-provider-web, mcp-web

4. **Routes/Features**
   - Threads (chat interface)
   - Hub (model management)
   - Projects
   - Settings (assistants, providers, extensions, hardware, etc.)
   - Local API server
   - System monitor

---

## Documentation Strategy

### Target Audience

Mid-level developer who:
- Knows JavaScript/TypeScript and React
- May be new to: Tauri, Rust, Zustand, TanStack Router, MCP
- Wants to contribute meaningfully within 2-3 weeks
- Learns through connections to familiar concepts

### Pedagogical Approach

Every document includes:
- 🧠 Mental Models - Simplified conceptual frameworks
- 🌉 Bridge from React/JS/TS - Analogies to familiar concepts
- 💡 Aha Moments - Key insights
- 🎯 Remember This - Mnemonics
- ⚠️ Common Pitfalls - What to avoid
- 🔗 Code Examples - Links to actual code with line numbers
- ✅/⚠️/🔍/🚨 Currency Markers - Tech stack status indicators

---

## Documentation Roadmap

### PHASE 2: Central Navigation Hub
**File:** `docs/learning/README.md`

Central starting point linking to all other documents with:
- Quick start path for day 1
- Week-by-week learning progression
- Technology overview with links to research
- Architecture at-a-glance
- Common tasks quick reference

### PHASE 3: Foundation Documents (Core Understanding)

#### 1. `GETTING_STARTED.md`
- Prerequisites and system setup
- Installation and first build
- Running the app locally
- Making first code change
- Development workflow basics
- Troubleshooting common issues
- **Links to**: Core package structure, web-app entry points

#### 2. `ARCHITECTURE_OVERVIEW.md`
- High-level system architecture (frontend + backend + extensions)
- Data flow: User action → React → Tauri → Rust → LLM → Response
- Monorepo structure and workspace relationships
- Extension system overview
- MCP integration architecture
- API server architecture
- **Diagrams**: System architecture, data flow, component relationships
- **Links to**: Core modules, Tauri main process, key routes

#### 3. `PROJECT_STRUCTURE.md`
- Detailed walkthrough of every major directory
- Purpose of each workspace (core, web-app, extensions, src-tauri)
- File naming conventions
- Where to find what (models, services, components, routes, etc.)
- Build artifacts and outputs
- Configuration files
- **Links to**: Every major directory with descriptions

#### 4. `TECH_STACK_GUIDE.md`
- Detailed explanation of each technology and its role
- Why each technology was chosen
- How technologies work together
- Comparison to alternatives (why Zustand not Redux, why TanStack Router not React Router, etc.)
- Version-specific features being used
- **Links to**: Tech stack research, official docs

#### 5. `DATA_FLOW_GUIDE.md`
- End-to-end request lifecycle examples:
  - User sends message → Thread state update → Tauri command → LLM inference → Streaming response
  - Model download → Progress tracking → Storage
  - Settings change → Persist → Reload
- State management flow (Zustand stores)
- Event system (Tauri events, RXJS)
- API request/response patterns
- **Diagrams**: Sequence diagrams for major flows
- **Links to**: Actual handler code, store definitions, event emitters

### PHASE 4: Deep-Dive Documents (Technology-Specific)

#### 6. `FRONTEND_ARCHITECTURE.md`
- React 19 features used in this project
- TanStack Router setup and patterns
- Route structure and navigation
- Component architecture (containers vs presentational)
- UI component system (Radix UI + TailwindCSS)
- State management with Zustand (stores breakdown)
- Service layer pattern
- Hooks architecture
- Form handling and validation
- Internationalization (i18next setup)
- **Links to**: Route files, stores, services, component examples

#### 7. `BACKEND_ARCHITECTURE.md`
- Tauri architecture overview (frontend ↔ backend communication)
- Rust entry point and main process
- Custom Tauri plugins (llamacpp, vector-db, RAG, hardware)
- Command handlers and IPC
- Event emission from Rust to frontend
- File system operations
- HTTP server implementation (OpenAI-compatible API)
- MCP integration (rmcp crate)
- **Rust concepts for JS/TS devs**: ownership, borrowing, async, traits
- **Links to**: Tauri commands, plugin source, event handlers

#### 8. `DATABASE_ARCHITECTURE.md` (if applicable)
- Local storage mechanisms
- Model storage and caching
- Thread/message persistence
- Settings storage
- Vector database (if used in RAG)
- **Links to**: Storage implementations, schemas

#### 9. `INTEGRATION_GUIDE.md`
- Extension system architecture
- Native extensions (how they work)
- Web extensions (how they work)
- Creating a new extension
- MCP (Model Context Protocol) integration
- Provider system (OpenAI, Anthropic, Groq, etc.)
- Local model integration (llama.cpp)
- **Links to**: Extension examples, provider implementations

### PHASE 5: Practical Guides (Hands-On)

#### 10. `PATTERNS_AND_CONVENTIONS.md`
- Code style and formatting (ESLint, Prettier)
- TypeScript patterns used
- React patterns (hooks, components, rendering)
- Naming conventions
- File organization patterns
- State management patterns
- Error handling patterns
- Async patterns (Promises, async/await, RXJS)
- Testing patterns
- **Good vs Bad examples** with explanations
- **Currency markers** for outdated vs current patterns
- **Links to**: Style configs, example implementations

#### 11. `HOW_TO_GUIDE.md`
Common tasks with step-by-step instructions:
- Add a new route
- Create a new Zustand store
- Add a Tauri command
- Create a new component
- Add a new setting
- Implement a new provider
- Add a new extension
- Handle form submission
- Add internationalization
- Implement real-time updates
- **Links to**: Relevant code examples for each task

#### 12. `CODE_TOURS.md`
Guided walkthroughs through actual code:
- Tour 1: "Sending a Message" - Follow a chat message from UI to LLM and back
- Tour 2: "Downloading a Model" - Model download flow
- Tour 3: "Settings Persistence" - How settings are saved and loaded
- Tour 4: "Extension Loading" - How extensions are discovered and loaded
- Tour 5: "Provider Integration" - How cloud providers work
- **Links to**: Every file involved with line numbers

#### 13. `DEVELOPMENT_WORKFLOW.md`
- Git workflow and branch strategy
- PR process and reviews
- Local development setup
- Hot reload and debugging
- Build process (development vs production)
- Testing workflow
- Linting and formatting
- Commit message conventions
- Release process
- **Links to**: GitHub workflows, make targets

### PHASE 6: Quality & Reference Docs

#### 14. `TESTING_GUIDE.md`
- Testing philosophy
- Vitest setup and configuration
- Unit testing (components, hooks, services)
- Integration testing
- E2E testing (if applicable)
- Test file structure
- Mocking strategies (Tauri APIs, external services)
- Testing Rust code
- Code coverage
- **Links to**: Test examples, test utilities, mocks

#### 15. `DEBUGGING_GUIDE.md`
- React DevTools usage
- Browser debugging
- Tauri debugging (frontend + backend)
- Rust debugging (if applicable)
- Common debugging scenarios
- Performance profiling
- Network debugging
- Log analysis
- **Links to**: Debug configurations, logging utilities

#### 16. `SECURITY_GUIDE.md`
- Security best practices
- Input validation
- XSS prevention
- Command injection prevention (Tauri)
- Secure storage
- API key management
- Privacy considerations (local AI data)
- Dependency security
- **Links to**: Security-related code examples

#### 17. `API_DOCUMENTATION.md`
- Core API reference (@janhq/core)
- Tauri command reference
- Service layer APIs
- Zustand store APIs
- OpenAI-compatible API server endpoints
- Extension APIs
- **Links to**: Type definitions, implementations

#### 18. `DATABASE_SCHEMA.md`
- Local storage schema
- Model storage structure
- Thread/message schema
- Settings schema
- Cache structures
- **Links to**: Schema definitions, migration files (if any)

### PHASE 7: Learning Exercises

#### 19. `EXERCISES.md`
Hands-on exercises with solutions:
- Exercise 1: Add a new settings toggle
- Exercise 2: Create a simple custom component
- Exercise 3: Add a new Tauri command
- Exercise 4: Implement a new Zustand store
- Exercise 5: Add a new route
- Exercise 6: Create a basic extension
- Each with: goal, steps, hints, solution link
- **Links to**: Exercise solutions in separate branch/folder

#### 20. `FIRST_CONTRIBUTIONS.md`
- Good first issues to tackle
- Contribution guidelines
- How to find issues matching your skill level
- Step-by-step first PR guide
- Code review process
- Community resources
- **Links to**: CONTRIBUTING.md, good first issues

### PHASE 8: Finalization

#### 21. Accuracy Review
- Review all documents for unverified claims
- Ensure uncertainty markers present
- Verify all code links work
- Check currency markers
- Validate assumptions

#### 22. Final Touches
- Update central README with all links
- Create FAQ.md
- Final polish

---

## Uncertainty Handling Strategy

Throughout documentation:
- ✅ **VERIFIED** - Confirmed from code/research
- ✅ **CURRENT (Nov 2025)** - Up-to-date pattern
- ⚠️ **UNCLEAR** - Not fully understood
- ⚠️ **OUTDATED PATTERN** - Works but newer approaches exist
- 🔍 **NEEDS VERIFICATION** - Requires deeper investigation
- ❓ **ASSUMPTION** - Inferred, not observed
- 🚧 **TODO** - Placeholder
- 🚨 **DEPRECATED** - Should not be used
- 🆕 **NEW IN 2025** - Recently introduced

When uncertain:
1. State it clearly
2. Provide investigation paths
3. Link to relevant code for developer to explore
4. Never fabricate explanations

---

## Success Metrics

A developer should be able to:
- [ ] Set up and run the project locally (Day 1)
- [ ] Understand the overall architecture (Day 2-3)
- [ ] Navigate the codebase confidently (Week 1)
- [ ] Make their first meaningful PR (Week 2)
- [ ] Understand all major subsystems (Week 3)
- [ ] Contribute to any part of the codebase (Week 4+)

---

## Next Steps

1. ✅ Complete tech stack research (DONE)
2. ✅ Create this plan (DONE)
3. ⏭️ Create central learning hub README.md
4. ⏭️ Create all foundation documents (Phase 3)
5. ⏭️ Create all deep-dive documents (Phase 4)
6. ⏭️ Create all practical guides (Phase 5)
7. ⏭️ Create all quality & reference docs (Phase 6)
8. ⏭️ Create learning exercises (Phase 7)
9. ⏭️ Perform accuracy review and finalize (Phase 8)

---

**Estimated Total Documentation:** 20+ comprehensive markdown files with extensive code links, diagrams, and pedagogical elements.
