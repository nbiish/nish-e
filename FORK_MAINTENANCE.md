# Fork Maintenance Guide

This document explains how to maintain this fork and integrate upstream changes.

## Repository Setup

**Original Repository:** `HandyHQ/handy` (upstream)  
**Your Fork:** `nbiish/nish-e` (origin)  
**Feature Branch:** `feature/llm-post-processing`

## Initial Setup

```bash
# Add upstream remote (if not already added)
git remote add upstream https://github.com/HandyHQ/handy.git

# Verify remotes
git remote -v
```

Expected output:
```
origin    https://github.com/nbiish/nish-e.git (fetch)
origin    https://github.com/nbiish/nish-e.git (push)
upstream  https://github.com/HandyHQ/handy.git (fetch)
upstream  https://github.com/HandyHQ/handy.git (push)
```

## Regular Workflow

### 1. Update Local `main` from Upstream

```bash
# Fetch latest changes from upstream
git fetch upstream

# Switch to main branch
git checkout main

# Merge upstream changes
git merge upstream/main

# Push updated main to your fork
git push origin main
```

### 2. Integrate Updates into Feature Branch

**Option A: Rebase (Recommended for cleaner history)**

```bash
# Switch to feature branch
git checkout feature/llm-post-processing

# Rebase on top of updated main
git rebase main

# If conflicts occur, resolve them, then:
git add <resolved-files>
git rebase --continue

# Force push (since history was rewritten)
git push origin feature/llm-post-processing --force-with-lease
```

**Option B: Merge (Safer, preserves history)**

```bash
# Switch to feature branch
git checkout feature/llm-post-processing

# Merge updated main
git merge main

# Resolve conflicts if any, then commit
git push origin feature/llm-post-processing
```

### 3. Keep Feature Branch Up to Date

```bash
# Quick update (assuming main is current)
git checkout feature/llm-post-processing
git rebase main
```

## Handling Conflicts

When conflicts occur during rebase or merge:

1. **Identify conflicts:**
   ```bash
   git status
   ```

2. **Edit conflicted files** - Look for conflict markers:
   ```
   <<<<<<< HEAD
   Your changes
   =======
   Upstream changes
   >>>>>>> main
   ```

3. **Resolve conflicts:**
   - Keep LLM-specific code
   - Integrate upstream improvements
   - Remove conflict markers

4. **Complete the operation:**
   ```bash
   git add <resolved-files>
   git rebase --continue  # if rebasing
   # or
   git commit             # if merging
   ```

## Development Workflow

### Working on Feature

```bash
# Always work on feature branch
git checkout feature/llm-post-processing

# Make changes
# ... edit files ...

# Commit changes
git add .
git commit -m "feat: add LLM prompt templates"

# Push to your fork
git push origin feature/llm-post-processing
```

### Before Starting Work

```bash
# Update main from upstream
git checkout main
git pull upstream main
git push origin main

# Update feature branch
git checkout feature/llm-post-processing
git rebase main
```

## Best Practices

### 1. Keep Changes Isolated

- Make LLM-specific changes only on `feature/llm-post-processing`
- Don't modify `main` branch directly
- Use feature flags for optional features

### 2. Commit Strategy

```bash
# Use conventional commits
git commit -m "feat: add LLM manager for model inference"
git commit -m "fix: handle LLM model loading errors"
git commit -m "docs: update LLM configuration guide"
```

### 3. Regular Syncing

Update from upstream at least:
- Before starting new work
- Weekly during active development
- Before creating pull requests

### 4. Testing After Updates

After syncing upstream changes:

```bash
# Rebuild to catch breaking changes
bun install
bun run tauri build

# Run in dev mode
bun run tauri dev
```

## Conflict Prevention

### File Organization

Keep LLM feature files separate:
- New files in dedicated directories
- Minimal changes to existing core files
- Use dependency injection where possible

### Key Integration Points

Files likely to need updates when syncing:

**High risk of conflicts:**
- `src-tauri/src/lib.rs` - Manager registration
- `src-tauri/src/settings.rs` - Settings structure
- `src/App.tsx` - Window management

**Medium risk:**
- `src-tauri/Cargo.toml` - Dependencies
- `package.json` - Frontend dependencies
- `src-tauri/resources/default_settings.json` - Default config

**Low risk:**
- New files in `src-tauri/src/managers/llm.rs`
- New files in `src/components/llm-window/`
- `FEATURE_LLM_POST_PROCESSING.md`

## Emergency Procedures

### Abort Rebase

```bash
git rebase --abort
```

### Abort Merge

```bash
git merge --abort
```

### Reset Feature Branch

```bash
# Save current work
git checkout feature/llm-post-processing
git branch feature/llm-post-processing-backup

# Reset to origin
git fetch origin
git reset --hard origin/feature/llm-post-processing
```

### Recover Lost Commits

```bash
# View reflog
git reflog

# Restore to previous commit
git reset --hard HEAD@{N}  # where N is the entry number
```

## Useful Commands

```bash
# Check which commits are unique to feature branch
git log main..feature/llm-post-processing

# See divergence between branches
git log --oneline --graph --all

# View upstream tags/releases
git fetch upstream --tags
git tag -l

# Cherry-pick specific upstream commit
git cherry-pick <commit-hash>

# Create patch for feature
git diff main..feature/llm-post-processing > llm-feature.patch

# Apply patch
git apply llm-feature.patch
```

## Release Strategy

### Creating a Release

```bash
# Ensure everything is up to date
git checkout main
git pull upstream main
git push origin main

# Update and test feature branch
git checkout feature/llm-post-processing
git rebase main

# Tag release
git tag -a v1.0.0-llm -m "LLM post-processing feature v1.0.0"
git push origin v1.0.0-llm
```

### Building Distributable

```bash
git checkout feature/llm-post-processing
bun run tauri build
```

Binary will be in `src-tauri/target/release/bundle/`

## Contributing Back Upstream (Optional)

If you want to contribute LLM feature to upstream:

```bash
# Create clean feature branch from upstream main
git fetch upstream
git checkout -b llm-feature-for-upstream upstream/main

# Cherry-pick commits
git cherry-pick <commit-hash-1>
git cherry-pick <commit-hash-2>

# Push to your fork
git push origin llm-feature-for-upstream
```

Then create PR from `nbiish/nish-e:llm-feature-for-upstream` → `HandyHQ/handy:main`
