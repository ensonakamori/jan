# Development Workflow

**Git, testing, PR process, and release workflow**

**Documented:** November 18, 2025

---

## Git Workflow

### Branch Strategy

**Main Branches:**
- `dev` - Development branch (default)
- `main` - Production releases (if exists)

**Feature Branches:**
```bash
feat/add-new-feature
fix/correct-bug
docs/update-readme
refactor/improve-code
test/add-tests
chore/update-dependencies
```

### Daily Workflow

```bash
# Morning: Get latest changes
git checkout dev
git pull origin dev

# Create feature branch
git checkout -b feat/my-feature

# Work on feature
# ... make changes ...

# Commit frequently
git add .
git commit -m "feat: add my feature"

# Push to fork
git push origin feat/my-feature

# Create PR when ready
```

---

## Commit Messages

### Format

```
type(scope): subject

body (optional)

footer (optional)
```

### Types

| Type | Usage |
|------|-------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Code formatting (no logic change) |
| `refactor` | Code restructure (no feature/fix) |
| `perf` | Performance improvement |
| `test` | Adding/updating tests |
| `chore` | Maintenance (deps, build, etc.) |
| `ci` | CI/CD changes |

### Examples

```bash
# Good commits
git commit -m "feat(threads): add message search functionality"
git commit -m "fix(downloads): prevent duplicate downloads"
git commit -m "docs: update architecture overview"
git commit -m "refactor(services): simplify service layer"
git commit -m "test(components): add Button tests"

# With body
git commit -m "feat(settings): add dark mode toggle

- Added toggle in Settings > Appearance
- Theme persists across sessions
- Applies to all components

Fixes #123"
```

---

## Pull Request Process

### Before Creating PR

**Checklist:**
```bash
# 1. Update from dev
git checkout dev
git pull origin dev
git checkout your-branch
git merge dev  # or rebase

# 2. Run tests
yarn test

# 3. Run linter
yarn lint

# 4. Build successfully
yarn build

# 5. Test in app
yarn dev
# Manually verify changes work
```

### Creating PR

1. **Push to fork**
   ```bash
   git push origin feat/my-feature
   ```

2. **Open PR on GitHub**
   - Base: `dev` (not `main`!)
   - Compare: `your-fork:feat/my-feature`

3. **Fill PR template**

```markdown
## Description
Brief description of changes

## Related Issue
Fixes #123
Closes #456

## Type of Change
- [ ] Bug fix
- [x] New feature
- [ ] Breaking change
- [ ] Documentation update

## Changes Made
- Added feature X
- Fixed bug Y
- Updated docs Z

## Screenshots (if UI changes)
[Attach screenshots]

## Testing
- [x] Tested locally
- [x] Added/updated tests
- [x] All tests pass
- [x] Linter passes

## Checklist
- [x] Code follows project style
- [x] Self-reviewed code
- [x] Commented complex code
- [x] Updated documentation
- [x] No new warnings
- [x] Added tests
- [x] Tests pass locally
```

### PR Review Process

**Timeline:**
- Initial review: 1-3 days
- Response to feedback: ASAP
- Final approval: 1-2 days after changes

**Common Feedback:**
- Code style issues → Run `yarn lint --fix`
- Missing tests → Add test coverage
- Unclear code → Add comments
- Breaking changes → Discuss approach

**Responding to Reviews:**
```bash
# Make requested changes
git add .
git commit -m "fix: address review feedback"
git push origin feat/my-feature
# PR updates automatically
```

---

## Testing Workflow

### Running Tests

```bash
# All tests
yarn test

# Watch mode (auto-rerun on change)
yarn test:watch

# With coverage
yarn test:coverage

# Specific file
yarn test path/to/file.test.ts

# UI mode (visual test runner)
yarn test:ui
```

### Writing Tests

**Test file naming:**
```
Button.tsx → Button.test.tsx
useAuth.ts → useAuth.test.ts

OR

__tests__/
  ├── Button.test.tsx
  └── useAuth.test.ts
```

**Test structure:**
```typescript
describe('Feature Name', () => {
  describe('Sub-feature', () => {
    it('should do something', () => {
      // Arrange
      const input = 'test'

      // Act
      const result = doSomething(input)

      // Assert
      expect(result).toBe('expected')
    })
  })
})
```

---

## Code Review Guidelines

### As Author

**Before requesting review:**
- ✅ Self-review your PR
- ✅ Check diff for unintended changes
- ✅ Remove console.logs and debug code
- ✅ Ensure tests pass
- ✅ Update documentation

**During review:**
- ✅ Respond to all comments
- ✅ Ask for clarification if unclear
- ✅ Push fixes promptly
- ✅ Mark conversations as resolved

### As Reviewer

**What to check:**
- Code correctness and logic
- Test coverage
- Performance implications
- Security concerns
- Code style consistency
- Documentation updates

**How to give feedback:**
- Be constructive, not critical
- Explain *why* not just *what*
- Suggest alternatives
- Approve when ready

---

## Pre-commit Hooks

**Husky Setup:** [.husky/](../../.husky/)

**Automatic checks:**
```bash
# On git commit:
- Runs linter
- Runs formatter (Prettier)
- Runs type check

# If checks fail, commit is blocked
```

**Bypassing (NOT recommended):**
```bash
git commit --no-verify  # Skip hooks
```

---

## Continuous Integration (CI)

**GitHub Actions:** [.github/workflows/](../../.github/workflows/)

**Automated checks on PR:**
1. **Lint** - Code style check
2. **Type Check** - TypeScript validation
3. **Tests** - All tests must pass
4. **Build** - Must build successfully

**Status checks:**
- ✅ All green → Ready to merge
- ❌ Any red → Fix before merge

---

## Release Workflow

### Versioning

**Semantic Versioning:** `MAJOR.MINOR.PATCH`

- `MAJOR`: Breaking changes
- `MINOR`: New features (backward compatible)
- `PATCH`: Bug fixes

**Examples:**
- `0.6.0 → 0.6.1` - Bug fix
- `0.6.1 → 0.7.0` - New feature
- `0.7.0 → 1.0.0` - Breaking change

### Creating a Release

⚠️ **Note:** Releases are handled by maintainers

**Process:**
1. Merge all features to `dev`
2. Test thoroughly
3. Update version in `package.json`
4. Create git tag: `git tag v0.7.0`
5. Push tag: `git push origin v0.7.0`
6. GitHub Actions builds release
7. Create GitHub release with changelog

---

## Build Process

### Development Build

```bash
# Quick start
make dev

# Or manually
yarn install
yarn build:core
yarn build:extensions
yarn dev
```

**Output:** Development binaries (not optimized)

### Production Build

```bash
make build

# Or platform-specific
make build:tauri:darwin   # macOS
make build:tauri:win32    # Windows
make build:tauri:linux    # Linux
```

**Output:** `src-tauri/target/release/bundle/`

**Artifacts:**
- **macOS:** `.dmg`, `.app`
- **Windows:** `.exe`, `.msi`
- **Linux:** `.deb`, `.AppImage`

---

## Debugging Workflow

### Frontend Debugging

```bash
# Start dev server
yarn dev

# Open DevTools in app
# macOS: Cmd+Option+I
# Windows/Linux: Ctrl+Shift+I
```

**Common debugging:**
```typescript
// Console logging
console.log('Debug:', value)
console.table(data)
console.error('Error:', error)

// Breakpoints in DevTools
debugger  // Add in code

// React DevTools
// Install extension, inspect components
```

### Backend Debugging (Rust)

```rust
// Print debugging
println!("Debug: {:?}", value);
dbg!(value);  // Debug macro

// Logging (use log crate)
use log::{info, debug, warn, error};

info!("Starting operation");
debug!("Value: {:?}", value);
warn!("Potential issue");
error!("Operation failed: {}", err);
```

**View logs:**
```bash
# Terminal where yarn dev runs shows Rust logs
```

---

## Hot Reload Behavior

### What Auto-Reloads

**✅ Auto-reloads:**
- React components (`.tsx`, `.ts`)
- CSS files (`.css`)
- Most frontend changes

**❌ Requires restart:**
- Rust code changes
- Core package changes
- Extension changes
- Tauri config changes
- Environment variables

**Manual rebuild needed:**
```bash
# Core changes
yarn build:core

# Extension changes
yarn build:extensions

# Rust changes
# Restart yarn dev
```

---

## Performance Profiling

### Frontend Performance

```typescript
// React Profiler
import { Profiler } from 'react'

<Profiler id="MyComponent" onRender={onRenderCallback}>
  <MyComponent />
</Profiler>

function onRenderCallback(
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime
) {
  console.log(`${id} took ${actualDuration}ms`)
}
```

**Chrome DevTools:**
- Performance tab → Record → Analyze

### Backend Performance

```rust
// Time operations
use std::time::Instant;

let start = Instant::now();
expensive_operation();
let duration = start.elapsed();
println!("Took: {:?}", duration);
```

---

## Environment Setup

### Environment Variables

**.env file:** (Not committed)
```bash
# Development
VITE_API_URL=http://localhost:1337
VITE_ENABLE_DEBUG=true

# Tauri (Rust)
RUST_LOG=debug
```

**Access in code:**
```typescript
// Frontend
const apiUrl = import.meta.env.VITE_API_URL

// Rust
let log_level = std::env::var("RUST_LOG").unwrap_or_default();
```

---

## Troubleshooting Common Issues

### Build Failures

```bash
# Clean and rebuild
make clean
make dev
```

### Port Already in Use

```bash
# Kill process on port 5173
lsof -ti:5173 | xargs kill -9
```

### Node Modules Issues

```bash
# Clean install
rm -rf node_modules yarn.lock
yarn install
```

### Rust Compilation Errors

```bash
cd src-tauri
cargo clean
cd ..
make dev
```

---

## Productivity Tips

### Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc
alias jdev='cd ~/jan && make dev'
alias jtest='yarn test:watch'
alias jlint='yarn lint --fix'
alias jclean='make clean && make dev'
```

### VS Code Settings

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.preferences.importModuleSpecifier": "non-relative"
}
```

---

## Next Steps

- **Testing Details:** [TESTING_GUIDE.md](./TESTING_GUIDE.md)
- **Debugging Tips:** [DEBUGGING_GUIDE.md](./DEBUGGING_GUIDE.md)
- **Contributing:** [FIRST_CONTRIBUTIONS.md](./FIRST_CONTRIBUTIONS.md)

**Questions?** Return to the [Learning Hub](./README.md)
