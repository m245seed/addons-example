# Moltbot HA Add-on – Installation Guide

A complete step-by-step guide to install the **Moltbot Gateway** Home Assistant add-on.

---

## Prerequisites

Before installing the Moltbot add-on, make sure you have:

- **Home Assistant OS** or **Home Assistant Supervised**
- **At least 2 GB of free storage** (builds & cache)
- **Optional:** SSH access (only if you need debug/CLI access inside the container)
- **Anthropic API key** (get it from [console.anthropic.com](https://console.anthropic.com))

### Supported architectures

- `amd64` (Intel/AMD 64-bit)
- `aarch64` (ARM 64-bit, e.g., Raspberry Pi 4/5)
- `armv7` (ARM 32-bit, e.g., Raspberry Pi 3)

---

## Installation

### Step 1: Add the repository to Home Assistant

1. Open your **Home Assistant UI**
2. Go to **Settings → Add-ons → Add-on Store**
3. Click **⋮** (three dots) in the top-right
4. Select **Repositories**
5. Add this repository URL:
   ```text
   https://github.com/Al3xand3r1987/moltbot-ha
   ```
6. Click **Add**

The repository should now appear in the Add-on Store.

---

### Step 2: Install the Moltbot Gateway add-on

1. Scroll down in the **Add-on Store** and search for **“Moltbot Gateway”**
2. Open the add-on
3. Click **Install**
4. Wait for the installation to finish (typically 5–10 minutes)
   - The initial build includes:
     - Node.js, Bun, pnpm, TypeScript
     - GitHub CLI, gog CLI
     - Moltbot source code and dependencies

---

### Step 3: Configure the add-on

#### Base configuration

1. After installation, open the **Configuration** tab
2. Set at least this option (recommended):

```yaml
update_mode: stable
```

**Important option:**
- `update_mode`: update behavior (see [CONFIGURATION.md](CONFIGURATION.md))

#### Optional configuration

```yaml
easy_setup_ui: false  # Optional: setup page in Ingress at /__setup/ (OAuth/API keys without SSH)
pinned_version: ""  # Optional: pin to a specific version
max_cached_versions: 2  # How many versions to keep cached
auto_cleanup_versions: true  # Automatically remove old versions
ssh_port: 2222  # Optional: SSH port (only relevant if SSH is enabled)
ssh_authorized_keys: "ssh-ed25519 AAAA... user@host"  # Optional: enable SSH (public key only, no password)
```

A full list of options is in [CONFIGURATION.md](CONFIGURATION.md).

---

### Step 4: Start the add-on

1. Open the **Info** tab
2. Enable **Start on boot** (recommended)
3. Enable **Watchdog** (optional, for automatic restarts)
4. Click **Start**

**On first start, the add-on will:**
- clone/initialize the Moltbot source
- install dependencies
- build the gateway and UI
- this can take **10–15 minutes** (depending on hardware)

**Watch the logs:**
- Open the **Log** tab
- Look for messages like `using version: ...` or `setup proxy started ...`
- Note: the gateway port is `18789` by default (see option `port`)

---

### Step 5: Open Moltbot

There are three ways to open Moltbot:

#### Option A: Web UI (Ingress) – recommended

1. Open the add-on **Info** tab
2. Click **OPEN WEB UI**
3. The Moltbot UI opens inside Home Assistant (Ingress)

**Optional setup (no SSH):** If you set `easy_setup_ui: true`, you can also open `/__setup/` in Ingress (OAuth/API keys).

**OR**

1. Look in the **Home Assistant sidebar**
2. Find the **Moltbot icon** (robot)
3. Click it to open the UI

#### Option B: Direct access via port

Direct access:
```text
http://YOUR-HA-IP:18789
```

#### Option C: Configure via SSH

For CLI configuration:

```bash
# From your computer
ssh -p 2222 root@YOUR-HA-IP

# Inside the container
cd /config/moltbot/data
nano moltbot.json
```

**Note:** SSH is optional and only runs if `ssh_authorized_keys` is set in the add-on configuration.

---

### Step 6: Configure your Anthropic API key

#### Via Setup UI (Ingress, no SSH) – recommended

1. In add-on configuration, set `easy_setup_ui: true` (optional, but very convenient for first-time setup)
2. Open the add-on Web UI (Ingress)
3. Open `/__setup/`
4. Paste your **Anthropic API key** and click **Save**

**Note:** The setup page writes keys to `/config/moltbot/data/state/.env` (not `moltbot.json`).

#### Via Web UI (easiest)

1. Open the Moltbot Web UI (see Step 5)
2. Go to **Settings**
3. Enter your **Anthropic API key**
4. Click **Save**

#### Via SSH

```bash
ssh -p 2222 root@YOUR-HA-IP

# Edit configuration
cd /config/moltbot/data
nano moltbot.json
```

Add the API key (example with placeholder):
```json
{
  "anthropic_api_key": "sk-ant-api03-your-key-here",
  ...
}
```

Save and restart the add-on.

---

### Step 7: Verify the installation

1. **Check logs:**
   - Easiest: open the **Log** tab in the add-on UI.
   - Via CLI (if you have HA CLI access):
     1. List add-ons and find the exact slug/identifier:
        ```bash
        ha addons list
        ```
     2. Then view logs:
        ```bash
        ha addons logs <ADDON_SLUG>
        ```

   **Common log hints:**
   - `using version: vYYYY.M.DD` (or similar)
   - `setup proxy started ... bind=127.0.0.1:8099 ...`
   - Optional (if SSH is enabled): `sshd listening on ...`

2. **Test the Web UI:**
   - Click **OPEN WEB UI**
   - You should see the Moltbot UI
   - Try a simple message, e.g., “Hello!”

3. **Check the file structure:**
   ```bash
   ssh -p 2222 root@YOUR-HA-IP
   ls -la /config/moltbot/
   ```

   You should see something like:
   ```text
   cache/
   data/
   .meta/
   source/
   ```

---

## First-time setup checklist

After installation, you should:

- [ ] Install and successfully start the add-on
- [ ] Reach the Web UI via **OPEN WEB UI**
- [ ] Configure the Anthropic API key
- [ ] Send a test message and receive a reply
- [ ] (Optional) Verify SSH access works (only if enabled)
- [ ] Create a snapshot/backup (recommended)

---

## Update strategy

The add-on includes an **automatic update system** with these modes:

- **`stable` (default)**: auto-update only to stable releases
- **`notify`**: notify when an update is available (manual approval)
- **`latest`**: includes pre-releases (alpha, beta, rc)
- **`disabled`**: no automatic updates

Configure this via `update_mode` on the add-on **Configuration** tab.

More details: [CONFIGURATION.md](CONFIGURATION.md#update-modes).

---

## Next steps

After a successful installation:

1. **Configure Telegram/WhatsApp** (optional)
   - See [CONFIGURATION.md](CONFIGURATION.md#messaging-integrations)

2. **Create your first snapshot**
   - Settings → System → Backups → Create backup
   - So you can restore if needed

3. **Explore skills**
   - Use the Web UI to discover available skills
   - Skills live under `/config/moltbot/data/workspace/`

4. **Community**
   - Report issues: [GitHub Issues](https://github.com/Al3xand3r1987/moltbot-ha/issues)
   - Contribute: see [CLAUDE.md](CLAUDE.md)

---

## Migration from v0.2.14

If you upgrade from an older version (v0.2.14), the add-on migrates your data **automatically** on first start:

- Old path: `/config/moltbot/.moltbot/`
- New path: `/config/moltbot/data/`

**Your data stays intact:**
- `moltbot.json` configuration
- state data
- workspace and skills

No manual steps required.

---

## Troubleshooting

If you run into installation issues, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for:

- build failures
- SSH connection problems
- API key issues
- update issues
- snapshot restore issues

---

## Support

Need help?

- **Documentation**: [README.md](README.md), [CONFIGURATION.md](CONFIGURATION.md)
- **Issues**: [GitHub Issues](https://github.com/Al3xand3r1987/moltbot-ha/issues)
- **Community**: Home Assistant Community Forums

---

**Installation complete! Have fun with Moltbot in Home Assistant.**
