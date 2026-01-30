# Moltbot Home Assistant Add-on

**Production-ready Home Assistant add-on for Moltbot** — Your AI coding assistant, now fully integrated with Home Assistant!

[![Release](https://img.shields.io/github/v/release/Al3xand3r1987/moltbot-ha?style=flat-square)](https://github.com/Al3xand3r1987/moltbot-ha/releases)
[![License](https://img.shields.io/github/license/Al3xand3r1987/moltbot-ha?style=flat-square)](LICENSE)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Add--on-blue?style=flat-square&logo=home-assistant)](https://www.home-assistant.io/)

---

## Features

- ✅ **Automatic Updates with Rollback** — Safe, tested updates with instant rollback capability
- ✅ **Full HA Snapshot Integration** — Instant restore from snapshots (~2 min vs 10+ min)
- ✅ **Native Home Assistant Integration** — Ingress support, Sidebar Panel, and Notifications
- ✅ **Structured Persistent Storage** — Organized data, cache, and version management
- ✅ **Multi-Architecture Support** — Works on amd64, arm64, and armv7 (Raspberry Pi!)
  - Full functionality on amd64 and arm64
  - Core features on armv7 (Google Workspace integration unavailable)
- ✅ **Smart Version Management** — Keep multiple versions cached for quick rollback
- ✅ **Optional SSH access (key-based)** — For debug/CLI access when needed (disabled by default)
- ✅ **Comprehensive Tooling** — Includes Node.js, Bun, pnpm, TypeScript, GitHub CLI, gog CLI

---

## Quick Start

### 1. Add Repository

1. Go to **Settings → Add-ons → Add-on Store → ⋮ → Repositories**
2. Add this repository:
   ```
   https://github.com/Al3xand3r1987/moltbot-ha
   ```

**Note:** This is a **Home Assistant Add-on repository** (Add-on Store). It is **not a HACS integration**.

### 2. Install Add-on

1. Find **"Moltbot Gateway"** in the add-on store
2. Click **Install**
3. Wait for installation (5-10 minutes)

### 3. Configure

Set basic options in the **Configuration** tab:

```yaml
update_mode: stable
```

**Optional (recommended for OAuth without SSH):**

```yaml
easy_setup_ui: true
```

**Optional (only if you really need SSH):**

```yaml
ssh_port: 2222
ssh_authorized_keys: "ssh-ed25519 AAAA... user@host"
```

### 4. Start & Access

1. Click **Start**
2. Wait for first build (10-15 minutes)
3. Click **"OPEN WEB UI"** button

**That's it!** Your Moltbot is ready to use. 🎉

For detailed instructions, see [INSTALLATION.md](INSTALLATION.md).

---

## What's New in v1.0.0

### 🚀 Major Features

#### Automatic Update System
- **4 Update Modes:** `disabled`, `notify`, `stable`, `latest`
- **Safe Updates:** Download → Build → Test → Activate (with automatic rollback on failure)
- **Version Cache:** Keep up to 10 versions for instant rollback
- **Smoke Tests:** New versions are tested before activation

#### Snapshot Integration
- **Instant Restore:** Use cached versions from snapshots (~2 minutes vs 10+ minutes rebuild)
- **Automatic Detection:** Recognizes when restored from snapshot
- **Preserved State:** Exact version and configuration restored

#### Home Assistant Integration
- **Ingress Support:** "OPEN WEB UI" button for easy access
- **Sidebar Panel:** Moltbot icon in Home Assistant sidebar
- **Notifications:** Update available, update completed, failures
- **Health Monitoring:** Watchdog integration

#### Storage Restrukturierung
- **Organized Structure:** Separate data, cache, source directories
- **Automatic Migration:** Seamless upgrade from v0.2.14
- **Version Tracking:** Metadata for current, snapshot, and last-run versions

### 🔧 Improvements

- **Fixed:** TypeScript build errors (Bun installed via npm)
- **Removed:** Duplicate dependencies in Dockerfile
- **Enhanced:** Multi-architecture support (armv7 added)
- **Added:** Version pinning for reproducible builds

### 📚 Documentation

- **[INSTALLATION.md](INSTALLATION.md)** — Complete installation guide
- **[CONFIGURATION.md](CONFIGURATION.md)** — All options explained in detail
- **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)** — Common issues and solutions
- **[CHANGELOG.md](moltbot_gateway/CHANGELOG.md)** — Full release notes

---

## Configuration Options

### Core Options

| Option | Default | Description |
|--------|---------|-------------|
| `easy_setup_ui` | `false` | Optionale Setup-Seite im Ingress unter `/__setup/` (OAuth/API Keys ohne SSH) |
| `ssh_authorized_keys` | `""` | (Optional) public SSH keys (empty = SSH disabled) |
| `ssh_port` | `2222` | SSH port (only relevant if SSH is enabled) |

### Update Options

| Option | Default | Description |
|--------|---------|-------------|
| `update_mode` | `stable` | Update mode: `disabled`, `notify`, `stable`, `latest` |
| `pinned_version` | `""` | Pin to specific version (overrides `update_mode`) |
| `max_cached_versions` | `2` | Number of versions to cache (1-10) |
| `auto_cleanup_versions` | `true` | Auto-remove old cached versions |

### Gateway / Repo / Logging

| Option | Default | Description |
|--------|---------|-------------|
| `port` | `18789` | Gateway Port (WebSocket/Web UI) |
| `verbose` | `false` | Verbose Logging |
| `log_format` | `pretty` | Log-Ausgabe im HA Log Tab (`pretty`/`raw`) |
| `log_color` | `false` | ANSI colors for `pretty` logs |
| `log_fields` | `""` | Extra log fields (e.g., `connectionId,uptimeMs`) |
| `repo_url` | `https://github.com/moltbot/moltbot.git` | Upstream repo for Moltbot updates |
| `branch` | `""` | Optional branch checkout |
| `github_token` | `""` | Token for private repos (not required by default) |

For complete configuration details, see [CONFIGURATION.md](CONFIGURATION.md).

---

## Update Modes Explained

### `stable` (Recommended)
- Auto-updates to stable releases only
- No pre-releases (alpha, beta, rc)
- Tested before activation
- Best for production

### `notify`
- Checks for updates
- Sends Home Assistant notification
- Requires manual approval
- Conservative approach

### `latest`
- Includes pre-releases
- Early access to new features
- Higher risk of bugs
- For testing/development

### `disabled`
- No automatic updates
- Manual control only
- Use with `pinned_version`

---

## Architecture

### Storage Structure

```
/config/moltbot/
├── cache/              # Built versions (in snapshots!)
│   ├── v2026.1.24/    # Current version
│   └── v2026.1.23/    # Previous (rollback available)
├── data/              # User data (persistent)
│   ├── moltbot.json  # Main configuration
│   ├── state/         # Runtime state
│   └── workspace/     # Skills & files
├── .meta/             # Version tracking
│   ├── current_version
│   ├── snapshot_version
│   └── last_run_version
└── source/            # Temporary source code
    └── moltbot-src/
```

### Included Tools

- **Node.js 24** — JavaScript runtime
- **Bun 1.1.38** — Fast JavaScript runtime & package manager
- **pnpm 9.15.2** — Efficient package manager
- **TypeScript** — Type-safe JavaScript
- **GitHub CLI** — Interact with GitHub from CLI
- **gog CLI 0.6.1** — Google Workspace integration (Gmail, Calendar, Drive, etc.)
- **clawdhub** — Moltbot hub CLI

---

## Supported Architectures

| Architecture | Platform | Status |
|--------------|----------|--------|
| `amd64` | Intel/AMD 64-bit | ✅ Tested |
| `arm64` | ARM 64-bit (Raspberry Pi 4/5) | ✅ Tested |
| `armv7` | ARM 32-bit (Raspberry Pi 3) | ✅ Tested |

---

## Usage Examples

### Access Methods

#### 1. Web UI (Ingress)
- Add-ons → Moltbot Gateway → **OPEN WEB UI**
- OR: Home Assistant Sidebar → **Moltbot Icon**
- Setup (optional, if `easy_setup_ui: true`): open `/__setup/` in Ingress

#### 2. Direct Port
```
http://YOUR-HA-IP:18789
```

#### 3. SSH Configuration
```bash
ssh -p 2222 root@YOUR-HA-IP
cd /config/moltbot/data
nano moltbot.json
```

### Common Tasks

#### Configure Anthropic API Key
```bash
ssh -p 2222 root@YOUR-HA-IP
nano /config/moltbot/data/moltbot.json
```

Add:
```json
{
  "anthropic_api_key": "sk-ant-api03-your-key-here"
}
```

#### Check Current Version
```bash
cat /config/moltbot/.meta/current_version
```

#### Manual Rollback
In add-on configuration:
```yaml
update_mode: disabled
pinned_version: "v2026.1.23"
```

#### View Cached Versions
```bash
ls /config/moltbot/cache/
```

---

## Snapshot/Backup Best Practices

1. **Create Regular Snapshots** — Before updates or major changes
2. **Include Add-on Data** — Snapshots automatically include `/config/moltbot/`
3. **Instant Restore** — Cached versions enable 2-minute restores
4. **Version Preservation** — Exact state restored (no unwanted updates)

**Create Snapshot:**
- Settings → System → Backups → Create Backup

**Restore Snapshot:**
- Settings → System → Backups → Select Backup → Restore

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Build fails | Check disk space (need 2GB+), see [TROUBLESHOOTING.md](TROUBLESHOOTING.md#issue-add-on-build-fails) |
| SSH won't connect | Check `ssh_authorized_keys` and `ssh_port` (SSH is optional and runs key-only) |
| Web UI not loading | Check add-on is running, try direct port access |
| Update failed | Check logs, will auto-rollback to previous version |
| Slow responses | Consider using `claude-haiku-4.0` model |

For detailed solutions, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

---

## Migrating from v0.2.14

**Automatic migration!** No manual steps required.

On first start of v1.0.0:
- Old location: `/config/moltbot/.moltbot/` → `/config/moltbot/data/state/`
- Config: `/config/moltbot/.moltbot/moltbot.json` → `/config/moltbot/data/moltbot.json`
- Workspace: `/config/moltbot/workspace/` → `/config/moltbot/data/workspace/`

Your data is preserved automatically!

---

## Development & Contributing

### Repository Information

- **Original Fork:** [ngutman/moltbot-ha-addon](https://github.com/ngutman/moltbot-ha-addon)
- **This Repository:** [Al3xand3r1987/moltbot-ha](https://github.com/Al3xand3r1987/moltbot-ha)
- **Maintainer:** Alexander (Al3xand3r1987)
- **Status:** Production-ready fork with enhanced features

### Contributing

Contributions welcome! See [CLAUDE.md](CLAUDE.md) for development guidelines.

**Workflow:**
1. Fork repository
2. Create feature branch from `master`
3. Follow [conventional commits](https://www.conventionalcommits.org/)
4. Test on all architectures
5. Update documentation
6. Create Pull Request

### Reporting Issues

Found a bug? [Create an issue](https://github.com/Al3xand3r1987/moltbot-ha/issues/new)

**Include:**
- Home Assistant version
- Add-on version
- Architecture (amd64/arm64/armv7)
- Logs (from `ha addons logs local_moltbot`)
- Steps to reproduce

---

## Links

- **Moltbot:** [github.com/moltbot/moltbot](https://github.com/moltbot/moltbot)
- **gog CLI:** [gogcli.sh](https://gogcli.sh)
- **GitHub CLI:** [cli.github.com](https://cli.github.com)
- **Anthropic Console:** [console.anthropic.com](https://console.anthropic.com)

---

## Requirements

- **Home Assistant OS** or **Home Assistant Supervised**
- **2GB+ free disk space** (for builds and caching)
- **Anthropic API Key** ([Get one here](https://console.anthropic.com))
- **Supported Architecture** (amd64, arm64, or armv7)

---

## Performance

| Operation | Time |
|-----------|------|
| First install | 10-15 minutes |
| Subsequent start | 2-3 minutes |
| Update (download + build) | 5-10 minutes |
| Snapshot restore (with cache) | ~2 minutes |
| Snapshot restore (without cache) | 10-15 minutes |
| Rollback | Instant (uses cached version) |

---

## Support

- **Documentation:** [Installation](INSTALLATION.md) • [Configuration](CONFIGURATION.md) • [Troubleshooting](TROUBLESHOOTING.md)
- **Issues:** [GitHub Issues](https://github.com/Al3xand3r1987/moltbot-ha/issues)
- **Community:** [Home Assistant Community](https://community.home-assistant.io/)

---

## License

MIT License — See [LICENSE](LICENSE) for details.

---

## Credits

- **Original Add-on:** [ngutman](https://github.com/ngutman) — Thank you for the foundation!
- **Moltbot:** The amazing AI coding assistant by [moltbot](https://github.com/moltbot)
- **Enhanced Version:** [Al3xand3r1987](https://github.com/Al3xand3r1987)

---

**Ready to supercharge your Home Assistant with AI? Install now and start coding with Claude! 🚀**
