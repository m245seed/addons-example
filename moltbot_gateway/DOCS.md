# Moltbot Gateway Documentation

This add-on runs the Moltbot Gateway on Home Assistant OS, with optional secure remote access via SSH (tunnel/CLI).

## Overview

- **Gateway** runs locally on the HA host (binds to loopback by default)
- **Optional: SSH server (Key-based)** provides secure remote access for Debug/CLI (disabled by default)
- **Persistent storage** under `/config/moltbot` survives add-on updates
- On first start, runs `moltbot setup` to create a minimal config

## Installation

1. In Home Assistant: **Settings → Add-ons → Add-on Store → ⋮ → Repositories**
2. Add: `https://github.com/Al3xand3r1987/moltbot-ha`
3. Reload the Add-on Store and install **Moltbot Gateway**

## Configuration

### Add-on Options

| Option | Description |
|--------|-------------|
| `easy_setup_ui` | If enabled: exposes a simple setup page at `/__setup/` in the Ingress UI (no SSH needed) |
| `ssh_authorized_keys` | (Optional) Your public key(s) for SSH access. If empty: SSH is disabled. Required for SSH tunnel/CLI access. |
| `ssh_port` | SSH server port (default: `2222`, only relevant if SSH is enabled) |
| `port` | Gateway WebSocket port (default: `18789`) |
| `repo_url` | Moltbot source repository URL |
| `branch` | Branch to checkout (uses repo's default if omitted) |
| `github_token` | Token for private repository access |
| `verbose` | Enable verbose logging |
| `log_format` | Log output format in the add-on Log tab: `pretty` or `raw` |
| `log_color` | Enable ANSI colors for pretty logs (may be ignored in the UI) |
| `log_fields` | Comma-separated metadata keys to append (e.g. `connectionId,uptimeMs,runId`) |

### First Run

The add-on performs these steps on startup:

1. Clones or updates the Moltbot repo into `/config/moltbot/source/moltbot-src`
2. Installs dependencies and builds the gateway
3. Runs `moltbot setup` if no config exists
4. Ensures `gateway.mode=local` if missing
5. Starts the gateway

### Moltbot Configuration

SSH into the add-on and run the configurator.

Note: SSH is only available if `ssh_authorized_keys` is set in the add-on options.

```bash
ssh -p 2222 root@<ha-host>
cd /config/moltbot/source/active
pnpm moltbot onboard
```

Or use the shorter flow:

```bash
pnpm moltbot configure
```

The gateway auto-reloads config changes. Restart the add-on only if you change SSH keys or build settings:

```bash
ha addons restart local_moltbot
```

### OAuth / API key setup without SSH (Ingress)

If you **don’t want to use SSH** (e.g., you prefer “ChatGPT/Codex OAuth” instead of an API key), you can do the setup directly through the Home Assistant UI:

1. Set `easy_setup_ui: true` in the add-on options and restart the add-on
2. Add-ons → **Moltbot Gateway** → **OPEN WEB UI**
3. Open the setup page:
   - If no configuration exists yet: it will appear automatically
   - Otherwise: open `/__setup/`
4. From there:
   - **Start wizard** (recommended) → choose “OpenAI Codex (ChatGPT OAuth)” or set API keys
   - Optional: API keys are written to `/config/moltbot/data/state/.env`

Note: OAuth tokens are automatically refreshed by Moltbot. A manual re-login is only needed if the provider invalidates the session.

## Usage

### SSH Tunnel Access

The gateway listens on loopback by default. Access it via SSH tunnel:

Note: Requires SSH enabled via `ssh_authorized_keys` in the add-on options.

```bash
ssh -p 2222 -N -L 18789:127.0.0.1:18789 root@<ha-host>
```

Then point Moltbot.app or the CLI at `ws://127.0.0.1:18789`.

### Bind Mode

Configure bind mode via the Moltbot CLI (over SSH), not in the add-on options.
Use `pnpm moltbot configure` or `pnpm moltbot onboard` to set it in `moltbot.json`.

## Data Locations

| Path | Description |
|------|-------------|
| `/config/moltbot/data/moltbot.json` | Main configuration |
| `/config/moltbot/data/state/` | State data (including tokens, e.g., `agent/auth.json`) |
| `/config/moltbot/data/workspace` | Agent workspace |
| `/config/moltbot/source/moltbot-src` | Source repository (for builds/updates) |
| `/config/moltbot/source/active` | Active version (symlink to cache) |
| `/config/moltbot/cache/` | Built versions (for rollback & snapshots) |
| `/config/moltbot/.ssh` | SSH keys |
| `/config/moltbot/.config` | App configs (gh, etc.) |

## Included Tools

- **gog** — Google Workspace CLI ([gogcli.sh](https://gogcli.sh))
- **gh** — GitHub CLI ([cli.github.com](https://cli.github.com))
- **clawdhub** — Skill marketplace CLI
- **hass-cli** — Home Assistant CLI

## Troubleshooting

### SSH doesn't work
Ensure `ssh_authorized_keys` is set in the add-on options with your public key.

### Gateway won't start
Check logs:
```bash
ha addons logs local_moltbot -n 200
```

### Build takes too long
The first boot runs a full build and may take several minutes. Subsequent starts are faster.

## Security Notes

- For `bind=lan/tailnet/auto`, enable gateway auth in `moltbot.json`
- The add-on uses host networking for SSH access
- Consider firewall rules for the SSH port if exposed to LAN

## Links

- [Moltbot](https://github.com/moltbot/moltbot) — Main repository
- [Documentation](https://docs.clawd.bot) — Full documentation
- [Community](https://discord.com/invite/clawd) — Discord server
- [gog CLI](https://gogcli.sh) — Google Workspace CLI
- [GitHub CLI](https://cli.github.com) — GitHub CLI
