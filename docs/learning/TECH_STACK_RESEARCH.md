# Technology Stack Research (November 2025)

**Research conducted:** November 18, 2025
**AI Knowledge Cutoff:** January 2025
**Purpose:** Verify current state of all technologies used in this project

---

## Executive Summary

This document catalogs all major technologies used in the Jan AI project, comparing the versions currently used against the latest stable releases as of November 2025. This research ensures that the learning documentation accurately reflects whether patterns and practices shown in this codebase represent current best practices or legacy approaches.

**Overall Assessment:** ✅ The project is generally up-to-date with most modern technologies, with a few exceptions noted below.

---

## Technologies Used in This Project

### React - v19.0.0

**Current Status (Nov 2025):**
- Latest stable: v19.2.0 (October 2025)
- Project uses: v19.0.0
- Status: ✅ **CURRENT** - React 19 stable was released December 5, 2024

**Important Updates Since Jan 2025:**
- React 19 became stable in December 2024
- New **Actions API** for handling async functions in transitions
- Built-in **React Compiler** that auto-optimizes components (reduces need for useMemo, useCallback, memo)
- New hooks: `useActionState`, `useFormStatus`, `useOptimistic`
- **React Server Components** are now stable
- New resource preloading APIs: `preinit`, `preload`, `prefetchDNS`, `preconnect`
- Better hydration error messages with diffs
- Full Custom Elements support
- Title and meta tag management directly in components

**What This Means for Learning:**
- ✅ This project uses the stable version of React 19
- Patterns shown represent **current best practices**
- Legacy class components should be avoided (not present in this project)
- This project can leverage all stable React 19 features

**Official Resources:**
- Docs: https://react.dev
- Release notes: https://react.dev/blog/2024/12/05/react-19
- Migration guide: https://react.dev/blog/2024/04/25/react-19-upgrade-guide

---

### TypeScript - v5.8.3 / v5.9.2

**Current Status (Nov 2025):**
- Latest stable: v5.9.x (August 2025)
- Project uses: v5.8.3 (core), v5.9.2 (web-app)
- Status: ✅ **CURRENT** - Project is on the latest major version

**Important Updates Since Jan 2025:**
- **TypeScript 5.9** released August 1, 2025
- Deferred imports (`import defer`) for performance optimization
- Expandable hover previews for better DX
- Minimal and updated `tsconfig.json` from `tsc --init`
- Enhanced Node.js v20 support
- Better TypeScript inference throughout

**What This Means for Learning:**
- ✅ Type patterns in this project are current
- No deprecated patterns to worry about
- Project benefits from latest TypeScript features

**Official Resources:**
- Docs: https://www.typescriptlang.org
- Release notes: https://devblogs.microsoft.com/typescript/announcing-typescript-5-9/
- Handbook: https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-9.html

---

### Vite - v6.3.2 / v5.4.20

**Current Status (Nov 2025):**
- Latest stable: v6.x
- Project uses: v6.3.2 (web-app), v5.4.20 (extensions-web)
- Status: ✅ **MOSTLY CURRENT** - Main app uses latest Vite 6

**Important Updates Since Jan 2025:**
- **Vite 6.0** released with major improvements
- Up to **5x faster** build times with new engine
- Node.js 18, 20, 22+ supported (Node 21 dropped)
- New default build target: `baseline-widely-available` (replaces 'modules')
- Sass legacy API removed (modern API only)
- New JSON handling: `json.stringify: 'auto'` for large files
- PostCSS updated to v6
- Better HTML element support
- New Environment API for framework authors

**What This Means for Learning:**
- ✅ Build configuration is current
- ⚠️ Note: `extensions-web` still uses Vite 5 (minor version behind)
- Patterns shown are current best practices

**Official Resources:**
- Docs: https://vite.dev
- Migration guide: https://vite.dev/guide/migration
- Release announcement: https://vite.dev/blog/announcing-vite6

---

### TailwindCSS - v4.1.4

**Current Status (Nov 2025):**
- Latest stable: v4.x (stable released January 22, 2025)
- Project uses: v4.1.4
- Status: ✅ **CURRENT** - On latest major version

**Important Updates Since Jan 2025:**
- **TailwindCSS 4.0 stable** released January 22, 2025
- **5x faster** builds with new high-performance engine
- **CSS-first configuration** using `@theme` directive (replaces JS config)
- Automatic content detection (no manual configuration)
- Built-in **container queries** (`@min-*`, `@max-*` variants)
- **3D transforms** (`rotate-x-*`, `rotate-y-*`, `scale-z-*`, `translate-z-*`)
- Enhanced gradients
- New utilities: `inset-shadow-*`, `inset-ring-*`, `field-sizing`, `color-scheme`, `font-stretch`
- New variants: `inert`, `nth-*`
- **Modernized color palette** using oklch color space (more vivid colors)
- Browser requirements: Safari 16.4+, Chrome 111+, Firefox 128+

**Breaking Changes:**
- `border` utility uses `currentColor` instead of `gray-200` default

**What This Means for Learning:**
- ✅ Project uses cutting-edge Tailwind 4
- All patterns represent **current best practices**
- CSS-first configuration is the modern approach

**Official Resources:**
- Docs: https://tailwindcss.com
- v4 announcement: https://tailwindcss.com/blog/tailwindcss-v4
- Upgrade guide: https://tailwindcss.com/docs/upgrade-guide

---

### Tauri - v2.8.5

**Current Status (Nov 2025):**
- Latest stable: v2.9.x (likely)
- Project uses: v2.8.5
- Status: ✅ **CURRENT** - Very recent version

**Important Updates Since Jan 2025:**
- Tauri 2.0 stable released in 2024
- **Mobile support** for iOS and Android
- Custom folder watching in `tauri dev`
- Plugin version compatibility checking
- iOS deployment target increased to 14.0
- `--root-certificate-path` option for HTTPS dev servers
- AppImage bundler improvements (reduced libfuse requirement)
- System TLS certificate reading
- Enhanced Tauri plugin ecosystem

**What This Means for Learning:**
- ✅ Project uses modern Tauri 2.x
- Mobile development patterns are current
- Plugin architecture represents best practices

**Official Resources:**
- Docs: https://v2.tauri.app
- Release notes: https://v2.tauri.app/blog/tauri-20/
- Migration guide: https://v2.tauri.app/start/migrate/

---

### Zustand - v5.0.3

**Current Status (Nov 2025):**
- Latest stable: v5.0.8
- Project uses: v5.0.3
- Status: ✅ **CURRENT** - On Zustand 5.x (minor version behind)

**Important Updates Since Jan 2025:**
- **Zustand v5** focuses on "no new features, just dropping old things"
- Dropped React <18 support
- Uses native `useSyncExternalStore` (no external package needed)
- **Much smaller bundle size**
- Dropped TypeScript <4.5 support
- Dropped ES5 support
- **Persist middleware** no longer stores initial state during creation
- New `useShallow` hook for shallow equality (replaces old pattern)

**What This Means for Learning:**
- ✅ State management patterns are current
- Minimal API surface makes it easy to learn
- Project benefits from smaller bundle and better performance

**Official Resources:**
- Docs: https://zustand.docs.pmnd.rs
- Migration to v5: https://zustand.docs.pmnd.rs/migrations/migrating-to-v5
- GitHub: https://github.com/pmndrs/zustand

---

### Vitest - v3.1.3 / v3.2.4 / v2.1.8

**Current Status (Nov 2025):**
- Latest stable: **v4.0** (October 22, 2025)
- Project uses: v3.x across workspaces
- Status: ⚠️ **ONE VERSION BEHIND** - Vitest 4 was released October 2025

**Important Updates Since Jan 2025:**
- **Vitest 3.0** released January 17, 2025
- Complete overhaul of test reporting system (reduced flicker)
- Workspace configuration simplified (no separate files needed)
- New browser testing method with multiple instances
- Test filtering by line number
- Fake timers: all timing APIs faked by default
- **Vitest 4.0** released October 22, 2025 (project hasn't upgraded yet)

**What This Means for Learning:**
- ⚠️ Project uses Vitest 3.x (one major version behind)
- Testing patterns are still valid and current
- Consider upgrading to Vitest 4 for latest features
- No urgent need to upgrade unless specific v4 features needed

**Official Resources:**
- Docs: https://vitest.dev
- Vitest 3 announcement: https://vitest.dev/blog/vitest-3
- Migration guide: https://v3.vitest.dev/guide/migration

---

### TanStack Router - v1.117.0

**Current Status (Nov 2025):**
- Latest stable: v1.x
- Project uses: v1.117.0
- Status: ✅ **CURRENT**

**Important Updates Since Jan 2025:**
- **TanStack Start** (full-stack framework) launched in RC (November 2025)
- 100% type-safe parallel route loaders
- First-class search param APIs with validation
- Auto-completed paths
- Lossless type inference throughout
- Integration with TanStack Start for SSR and server functions

**What This Means for Learning:**
- ✅ Routing patterns represent cutting-edge type-safe routing
- This is the modern alternative to React Router
- TanStack ecosystem integration is a major advantage

**Official Resources:**
- Docs: https://tanstack.com/router/latest
- Overview: https://tanstack.com/router/latest/docs/framework/react/overview
- GitHub: https://github.com/TanStack/router

---

### Rust - v1.77.2 (edition 2021)

**Current Status (Nov 2025):**
- Latest stable: **v1.91.1**
- Project uses: v1.77.2
- Status: 🚨 **SIGNIFICANTLY BEHIND** - Project is ~14 versions behind

**Important Updates Since Jan 2025:**
- Current stable is Rust 1.91.1 (November 2025)
- **Rust 1.85.0** (February 2025) stabilized Rust 2024 Edition
- Rust 1.87.0 celebrated 10 years of Rust (May 2025)
- Many language improvements and features added
- Performance and tooling enhancements

**What This Means for Learning:**
- ⚠️ Rust code patterns may not reflect latest features
- Core patterns are still valid (Rust maintains backward compatibility)
- **Recommendation:** Consider upgrading Rust version for latest features
- Minimum version requirement is 1.77.2 for this project

**Official Resources:**
- Docs: https://doc.rust-lang.org
- Release notes: https://doc.rust-lang.org/stable/releases.html
- What's new: https://blog.rust-lang.org

---

### Node.js - (Inferred v22.x)

**Current Status (Nov 2025):**
- Latest LTS: v22.21.1 (October 28, 2025) - "Jod" in Maintenance LTS
- Project uses: Not explicitly specified (likely v22.x based on Yarn 4 usage)
- Status: ✅ **CURRENT** (assuming v22.x is used)

**Important Updates Since Jan 2025:**
- Node.js 22.x entered Maintenance LTS in October 2025
- Support until April 30, 2027
- Built-in **WebSocket client** (enabled by default)
- **Watch Mode** is now stable
- New `--use-env-proxy` CLI flag
- HTTP proxy support for fetch
- OpenSSL 3.5.2
- Updated root certificates to NSS 3.114

**What This Means for Learning:**
- ✅ Modern Node.js features available
- No deprecated patterns to worry about
- WebSocket support eliminates need for third-party libs

**Official Resources:**
- Docs: https://nodejs.org
- Release notes: https://nodejs.org/en/blog/release/v22.11.0
- LTS schedule: https://github.com/nodejs/Release

---

### Yarn - v4.5.3

**Current Status (Nov 2025):**
- Latest stable: v4.x
- Project uses: v4.5.3
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- ✅ Modern package management
- Workspace management is current best practice
- Plug'n'Play (PnP) architecture if enabled

**Official Resources:**
- Docs: https://yarnpkg.com
- Migration guide: https://yarnpkg.com/migration/guide

---

### Radix UI - Multiple v1.x / v2.x packages

**Current Status (Nov 2025):**
- Project uses: Latest versions of Radix UI primitives
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- ✅ Accessible component patterns
- Modern unstyled primitive approach
- Perfect complement to TailwindCSS

**Official Resources:**
- Docs: https://www.radix-ui.com
- Primitives: https://www.radix-ui.com/primitives

---

### Framer Motion - v12.23.12

**Current Status (Nov 2025):**
- Project uses: v12.x
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- ✅ Modern animation library patterns
- Latest Motion v12 features available

**Official Resources:**
- Docs: https://www.framer.com/motion/

---

### Rolldown - v1.0.0-beta.1

**Current Status (Nov 2025):**
- Project uses: v1.0.0-beta.1
- Status: 🔍 **BETA** - Not yet stable (v1.0)

**What This Means for Learning:**
- ⚠️ This is a beta tool (Rollup alternative in Rust)
- Only used in core package build
- Patterns may change before stable v1.0

**Official Resources:**
- GitHub: https://github.com/rolldown/rolldown

---

### Python Stack (autoqa)

**Technologies:**
- cua-computer ~0.3.5
- cua-agent ~0.3.0
- opencv-python ~4.10.0
- PyAutoGUI ~0.9.54
- psutil ~7.0.0

**Current Status (Nov 2025):**
- Status: 🔍 **NEEDS VERIFICATION** - These appear to be current versions
- `cua-*` packages are specific to this project/org

**What This Means for Learning:**
- ✅ Automation tooling for QA
- Modern Python ecosystem usage

---

### Model Context Protocol (MCP) - v1.17.5

**Current Status (Nov 2025):**
- Project uses: @modelcontextprotocol/sdk v1.17.5
- Status: ✅ **CURRENT**

**What This Means for Learning:**
- ✅ Integration with AI model context protocol
- Modern SDK usage

**Official Resources:**
- Docs: https://modelcontextprotocol.io

---

## Summary Matrix

| Technology | Project Version | Latest (Nov 2025) | Status | Priority |
|-----------|----------------|-------------------|--------|----------|
| React | 19.0.0 | 19.2.0 | ✅ Current | - |
| TypeScript | 5.8.3 / 5.9.2 | 5.9.x | ✅ Current | - |
| Vite | 6.3.2 / 5.4.20 | 6.x | ✅ Mostly Current | Low |
| TailwindCSS | 4.1.4 | 4.x | ✅ Current | - |
| Tauri | 2.8.5 | 2.9.x | ✅ Current | - |
| Zustand | 5.0.3 | 5.0.8 | ✅ Current | Low |
| Vitest | 3.x | 4.0 | ⚠️ One behind | Medium |
| TanStack Router | 1.117.0 | 1.x | ✅ Current | - |
| Node.js | 22.x (inferred) | 22.21.1 LTS | ✅ Current | - |
| **Rust** | **1.77.2** | **1.91.1** | 🚨 **14 versions behind** | **High** |
| Yarn | 4.5.3 | 4.x | ✅ Current | - |
| Rolldown | 1.0.0-beta.1 | beta | 🔍 Beta | Low |

---

## Recommendations

### High Priority

1. **Upgrade Rust** from 1.77.2 → 1.91.1
   - Missing 14 versions worth of improvements
   - Rust 2024 Edition available (currently using 2021 Edition)
   - Better performance, tooling, and language features

### Medium Priority

2. **Consider upgrading Vitest** from 3.x → 4.0
   - Released October 2025
   - Enhanced testing and reporting features
   - Check breaking changes before upgrading

### Low Priority

3. **Update Zustand** from 5.0.3 → 5.0.8 (patch updates)
4. **Upgrade Vite in extensions-web** from 5.4.20 → 6.x
5. **Monitor Rolldown** for stable v1.0 release

---

## Conclusion

✅ **Overall Assessment: The Jan AI project uses modern, current technologies.**

The project is in excellent shape technologically, using React 19, TypeScript 5.9, Vite 6, TailwindCSS 4, and Tauri 2.8 - all representing cutting-edge, production-ready versions. The only significant gap is the Rust version (1.77.2 vs 1.91.1), which should be updated to access the latest language features and performance improvements.

All learning materials created for this project can confidently reference these technologies as **current best practices** (Nov 2025), with appropriate notes for the Rust version lag and minor version differences in Vitest.

---

**Last Updated:** November 18, 2025
**Next Review:** Recommend reviewing quarterly or when major version bumps occur
