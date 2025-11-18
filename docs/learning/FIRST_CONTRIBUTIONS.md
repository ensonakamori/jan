# First Contributions

**Your guide to making your first pull request**

**Documented:** November 18, 2025

---

## Before You Start

**Prerequisites:**
- ✅ Development environment set up ([Getting Started](./GETTING_STARTED.md))
- ✅ App running locally (`make dev`)
- ✅ Basic understanding of the architecture ([Architecture Overview](./ARCHITECTURE_OVERVIEW.md))

---

## Finding Your First Issue

### Good First Issues

Visit: https://github.com/janhq/jan/labels/good-first-issue

**Look for:**
- Documentation improvements
- UI/UX enhancements
- Bug fixes with clear reproduction
- Small feature additions

### Issue Types by Difficulty

| Difficulty | Examples |
|------------|----------|
| 🟢 **Beginner** | Fix typos, update docs, add translations |
| 🟡 **Intermediate** | Add UI components, fix bugs, improve UX |
| 🔴 **Advanced** | New features, Rust changes, architecture changes |

**Recommendation:** Start with 🟢 or 🟡 issues

---

## Your First PR: Adding a Translation

**Perfect first contribution!** Low risk, high value.

### Step 1: Pick a Language

Check existing translations: [web-app/src/locales/](../../web-app/src/locales/)

Missing your language? Great! If it exists, look for incomplete translations.

### Step 2: Fork the Repository

1. Visit https://github.com/janhq/jan
2. Click "Fork" button
3. Clone your fork:
   ```bash
   git clone https://github.com/YOUR-USERNAME/jan.git
   cd jan
   ```

### Step 3: Create a Branch

```bash
git checkout -b add-french-translations
```

**Naming:** Use descriptive branch names (feat/*, fix/*, docs/*)

### Step 4: Make Changes

```json
// web-app/src/locales/fr/translation.json
{
  "welcome": {
    "title": "Bienvenue sur Jan AI",
    "description": "Exécutez des LLM localement"
  },
  "settings": {
    "general": "Général",
    "appearance": "Apparence",
    "extensions": "Extensions"
  }
}
```

### Step 5: Test Your Changes

```bash
yarn dev
# Change language in settings to French
# Verify all strings display correctly
```

### Step 6: Commit

```bash
git add web-app/src/locales/fr/
git commit -m "feat(i18n): add French translations

- Added welcome section
- Added settings section
- Translated 25 keys"
```

### Step 7: Push

```bash
git push origin add-french-translations
```

### Step 8: Create Pull Request

1. Visit https://github.com/janhq/jan/pulls
2. Click "New Pull Request"
3. Select your branch
4. Fill in PR template:

```markdown
## Description
Added French translations for welcome and settings sections

## Changes
- Created `web-app/src/locales/fr/translation.json`
- Translated 25 keys
- Tested in app by switching language

## Screenshots
[Screenshot showing French UI]

## Checklist
- [x] Tested locally
- [x] All strings display correctly
- [x] No breaking changes
```

### Step 9: Wait for Review

Maintainers will review your PR. They may:
- Approve and merge ✅
- Request changes 📝
- Ask questions ❓

**Be responsive!** Reply to feedback promptly.

---

## Your Second PR: Fixing a Bug

### Step 1: Reproduce the Bug

1. Read the issue description
2. Try to reproduce locally
3. Understand the expected vs actual behavior

### Step 2: Find the Code

Use search to find relevant files:
```bash
# Search for text/function name
grep -r "searchTerm" web-app/src/

# Find component
find . -name "ComponentName.tsx"
```

### Step 3: Fix and Test

```typescript
// Before (bug)
if (value = 10) { // Assignment instead of comparison
  doSomething()
}

// After (fixed)
if (value === 10) { // Correct comparison
  doSomething()
}
```

**Test thoroughly:**
- Run the app
- Try the feature
- Test edge cases
- Run tests: `yarn test`

### Step 4: Create PR

```markdown
## Description
Fixed comparison bug in settings validation

## Issue
Fixes #1234

## Changes
- Changed assignment (=) to comparison (===) in settings.ts:45
- Added test case for validation

## Testing
- Tested with valid values ✅
- Tested with invalid values ✅
- All existing tests pass ✅
```

---

## Your Third PR: Adding a Feature

### Step 1: Discuss First

For larger features:
1. Comment on the issue
2. Describe your approach
3. Wait for maintainer feedback

**Why?** Ensures your solution aligns with project direction.

### Step 2: Break Down the Work

**Example:** Adding a "Export Chat" button

**Tasks:**
1. Add button to UI
2. Create export service
3. Add Tauri command (if needed)
4. Add tests
5. Add to i18n

### Step 3: Implement

Follow the [How-To Guide](./HOW_TO_GUIDE.md) for specific tasks.

### Step 4: Write Tests

```typescript
describe('Export Chat', () => {
  it('exports chat as JSON', async () => {
    const result = await exportService.exportAsJSON(threadId)
    expect(result).toContain('"messages":')
  })

  it('exports chat as Markdown', async () => {
    const result = await exportService.exportAsMarkdown(threadId)
    expect(result).toContain('# Chat Export')
  })
})
```

### Step 5: Update Documentation

If you added a feature, document it:
- Update relevant docs
- Add to FAQ if needed
- Update README if user-facing

---

## Code Review Best Practices

### Responding to Feedback

**Good responses:**
✅ "Good catch! Fixed in abc123"
✅ "I chose approach X because Y. Thoughts?"
✅ "Can you clarify what you mean by Z?"

**Avoid:**
❌ Arguing without explanation
❌ Ignoring feedback
❌ Being defensive

### Making Changes

```bash
# Make requested changes
git add .
git commit -m "fix: address review feedback"
git push origin your-branch
```

**PR automatically updates!**

---

## Common Mistakes to Avoid

### ❌ Large, Unfocused PRs

**Bad:** Change 20 files, add 3 features, fix 5 bugs

**Good:** One PR per feature/fix

### ❌ Not Testing

Always test your changes locally!

### ❌ Ignoring CI Failures

If tests fail in CI, fix them before requesting review.

### ❌ Poor Commit Messages

**Bad:** "fix stuff"

**Good:** "fix: correct validation in settings form"

### ❌ Not Following Code Style

Run `yarn lint` before committing

---

## PR Checklist

Before submitting, verify:

- [ ] Code runs locally without errors
- [ ] Tests pass (`yarn test`)
- [ ] Lint passes (`yarn lint`)
- [ ] Changes are focused and minimal
- [ ] Commit messages are descriptive
- [ ] PR description explains the change
- [ ] Screenshots included (for UI changes)
- [ ] Documentation updated (if needed)
- [ ] Issue reference included (Fixes #123)

---

## Getting Help

**Stuck?**

1. **Read the docs** - Check [Learning Hub](./README.md)
2. **Search issues** - Someone may have asked before
3. **Ask in Discord** - `#💻|contributing` channel
4. **Comment on issue** - Tag maintainers

**Don't be shy!** Everyone was a first-time contributor once.

---

## After Your PR is Merged

🎉 **Congratulations!** You're now a Jan AI contributor!

**Next steps:**
1. Add "Contributor" to your LinkedIn/resume
2. Find another issue
3. Help review others' PRs
4. Mentor new contributors

---

## Issue Labels Guide

| Label | Meaning |
|-------|---------|
| `good-first-issue` | Perfect for beginners |
| `help-wanted` | Maintainers need help |
| `bug` | Something broken |
| `enhancement` | New feature request |
| `documentation` | Docs needed |
| `wontfix` | Won't be implemented |
| `duplicate` | Already exists |

---

## Contribution Types

**Not just code!**

- 📝 **Documentation** - Improve guides, fix typos
- 🌍 **Translations** - Add/improve i18n
- 🐛 **Bug reports** - Detailed reproduction steps
- 💡 **Feature ideas** - Well-thought-out proposals
- 👀 **Code review** - Review others' PRs
- ❓ **Help others** - Answer questions in Discord

---

## Recognition

Contributors are recognized in:
- GitHub contributors list
- Release notes (for significant contributions)
- Project README (top contributors)

---

## Resources

- **Contributing Guide:** [CONTRIBUTING.md](../../CONTRIBUTING.md)
- **Code of Conduct:** Be respectful and professional
- **Discord:** https://discord.gg/FTk2MvZwJH
- **GitHub Issues:** https://github.com/janhq/jan/issues

---

**Ready to contribute?** Find an issue and get started!

**Questions?** Return to the [Learning Hub](./README.md) or ask in Discord.
