# Gabby

A performance-optimized fork of [Pi](https://github.com/earendil-works/pi) (the AI coding agent CLI). Gabby runs alongside the official `pi` command as a separate binary with its own config directory, so you can use both without conflicts.

## Why

Pi is great, but cold-boot startup has room for improvement. Gabby experiments with optimizations that reduce time-to-interactive without changing the core feature set.

## What's Different

| | pi (official) | gabby (fork) |
|---|---|---|
| **Command** | `pi` | `gabby` |
| **Config dir** | `~/.pi/agent/` | `~/.gabby/agent/` |
| **Env prefix** | `PI_*` | `GABBY_*` |

### Optimizations (so far)

1. **Migration sentinel** — `runMigrations()` now skips all filesystem checks after the first boot on a new version. Previously, 5+ synchronous I/O operations (existsSync, readdirSync, etc.) ran on every startup checking for conditions that almost never change.

2. **SettingsManager deduplication** — `SettingsManager.create()` was called 3 times for the same cwd during startup, each reading `settings.json`, acquiring file locks, and parsing JSON. Gabby creates it once and reuses it.

3. **Configurable model refresh timeout** — Default raised from 15s to 30s, and configurable via `GABBY_MODEL_REFRESH_TIMEOUT_MS`. This reduces spurious "Model refresh timed out" errors on slow connections where the pi.dev catalog payload takes time to download.

### Planned

- Lazy model catalog refresh (show cached models immediately, refresh in background after TUI renders)
- Parallel provider catalog fetches
- Extension compilation caching (jiti output to disk)
- Async settings I/O

See [DEVELOPMENT.md](./DEVELOPMENT.md) for build, rebuild, and upstream merge instructions.

## License

MIT — same as the original Pi project.
