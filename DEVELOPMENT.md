# Gabby — Development Guide

## Prerequisites

- Node.js >= 22.19.0
- npm >= 10
- Git

## Initial Setup

```bash
# Clone the repo
git clone git@github.com:jdai-png/gabby.git
cd gabby

# Install dependencies (skip postinstall scripts to avoid downloading fd/rg binaries)
npm install --ignore-scripts

# Full monorepo build (tui → ai → agent → protocol → client → coding-agent → server)
npm run build
```

## Rebuilding After Code Changes

Make your changes in `packages/coding-agent/src/`, then rebuild:

```bash
# From the repo root — rebuild everything
npm run build

# Or just the coding-agent package (faster, if only that changed)
cd packages/coding-agent && npm run build
```

`gabby` runs directly from `dist/cli.js` in the dev tree — no reinstall, no `npm link`. Changes take effect immediately on next run.

## Running Locally

```bash
# Run from the repo root using the dev build
node packages/coding-agent/dist/cli.js

# Or use the gabby command if symlinked (macOS/Linux)
# The symlink was created at /opt/homebrew/bin/gabby → ~/Desktop/gabby/packages/coding-agent/dist/cli.js
gabby
```

## Type Checking

```bash
# Check just the coding-agent package (fast)
cd packages/coding-agent && npx tsgo --noEmit

# Check all packages (slow, includes test files)
cd /path/to/gabby && npx tsgo --noEmit
```

## Merging Upstream Changes

Upstream is `earendil-works/pi`. The fork tracks it via the `upstream` remote.

```bash
# One-time: add the upstream remote (if not already set)
git remote add upstream git@github.com:earendil-works/pi.git

# Fetch latest from upstream
git fetch upstream

# Update main branch
git checkout main
git merge upstream/main          # fast-forwards if no local commits on main

# Rebase the feature branch onto updated main
git checkout feat/cold-boot-optimizations
git rebase main

# Resolve any conflicts, then:
#   git add <resolved-files>
#   git rebase --continue

# Push the rebased branch
git push --force-with-lease origin feat/cold-boot-optimizations
```

### Conflict Resolution Tips

Upstream changes that are most likely to conflict:

| File | Why conflicts happen |
|------|---------------------|
| `packages/coding-agent/package.json` | We changed `bin`, `piConfig.name`, `piConfig.configDir` |
| `packages/coding-agent/src/main.ts` | We changed SettingsManager creation flow |
| `packages/coding-agent/src/migrations.ts` | We added the version sentinel |
| `packages/coding-agent/src/core/model-runtime.ts` | We changed the default timeout |

During rebase, check our diff first to understand what to preserve:

```bash
# See what gabby changed vs upstream main
git diff upstream/main -- packages/coding-agent/
```

### After Merging Upstream

```bash
# Rebuild everything
npm run build

# Verify it boots
gabby --version

# Install any new managed binaries (fd, rg) if needed
# Copy from official pi install:
#   cp ~/.pi/agent/bin/* ~/.gabby/agent/bin/
```

## Config Directory

Gabby uses `~/.gabby/agent/` — separate from official Pi's `~/.pi/agent/`.

| File/Dir | Purpose |
|----------|---------|
| `settings.json` | User preferences |
| `auth.json` | API keys / OAuth tokens |
| `models-store.json` | Cached model catalog |
| `sessions/` | Chat session history |
| `skills/` | Custom skills (SKILL.md files) |
| `rules/` | Per-project rules |
| `extensions/` | Custom extensions |
| `prompts/` | Custom prompt templates |
| `bin/` | Managed binaries (fd, rg) |

## Project Structure

```
gabby/
├── packages/
│   ├── coding-agent/    ← main CLI (where most work happens)
│   │   └── src/
│   │       ├── main.ts           ← entry point / startup flow
│   │       ├── migrations.ts     ← one-time setup migrations
│   │       ├── core/
│   │       │   ├── model-runtime.ts    ← model loading & refresh
│   │       │   ├── settings-manager.ts ← settings persistence
│   │       │   ├── resource-loader.ts  ← extensions/skills/prompts
│   │       │   └── ...
│   │       └── modes/
│   │           └── interactive/  ← TUI code
│   ├── ai/               ← provider/model abstractions
│   ├── agent/            ← agent session logic
│   ├── tui/              ← terminal UI framework
│   ├── protocol/         ← RPC protocol definitions
│   ├── client/           ← SDK client
│   ├── server/           ← backend server
│   └── storage/          ← session persistence backends
├── scripts/              ← build/release/test scripts
└── GABBY.md              ← fork philosophy
```
