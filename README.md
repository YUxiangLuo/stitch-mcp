# Stitch MCP Helper CLI

> Stitch MCP OAuth setup assistant - automates Google Cloud authentication for Stitch API

## One command to connect any MCP client to Stitch

`npx @_davideast/stitch-mcp init` walks you through the complete Google Cloud authentication setup for Stitch API in a single interactive session. It guides you through installing `gcloud`, configuring authentication, managing IAM permissions, enabling APIs, and generating your client-specific MCP config.

- **Zero Config Setup:** One `npx` command handles gcloud install, authentication, project setup, and MCP configuration
- **Universal Client Support:** Works with Antigravity, Claude Code, Cursor, VSCode, or Gemini CLI
- **Smart Defaults:** Automatically detects existing gcloud installations and reuses active projects
- **Self-Healing:** Built-in `doctor` command diagnoses and reports setup issues

## Quick Start

```bash
npx @_davideast/stitch-mcp init
```

This single command will:
1. Install Google Cloud CLI (if needed)
2. Authenticate you with Google
3. Set up application credentials
4. Select a GCP project
5. Configure IAM permissions
6. Enable Stitch API
7. Generate MCP configuration for your client

**Example output:**
```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"],
      "env": {
        "STITCH_PROJECT_ID": "your-project-id"
      }
    }
  }
}
```

Copy this config into your MCP client settings and you're ready to use the Stitch MCP server.

## Verify Your Setup

```bash
npx @_davideast/stitch-mcp doctor
```

Runs health checks on:
- ✔ Google Cloud CLI installation
- ✔ User authentication
- ✔ Application credentials
- ✔ Project configuration
- ✔ Stitch API connectivity

## Logout

```bash
npx @_davideast/stitch-mcp logout

# Skip confirmation
npx @_davideast/stitch-mcp logout --force

# Clear all config
npx @_davideast/stitch-mcp logout --clear-config
```

## Deep Dive

### Installation

Can be configured with `npx` or installed globally.

```bash
npx @_davideast/stitch-mcp init
```

Or install globally if you prefer:

```bash
npm install -g @_davideast/stitch-mcp
stitch-mcp init
```

### Commands Reference

#### `init` - Interactive Setup

```bash
npx @_davideast/stitch-mcp init [options]
```

**Options:**
- `--local` - Install gcloud locally to project instead of user home
- `-y, --defaults` - Use default values for all prompts
- `-c, --client <client>` - Specify MCP client (antigravity, vscode, cursor, claude-code, gemini-cli)
- `-t, --transport <type>` - Transport type (http or stdio)

**What happens:**
1. **MCP Client Selection** - Choose your IDE/CLI
2. **gcloud Setup** - Install or detect Google Cloud CLI
3. **User Authentication** - OAuth login flow
4. **Application Credentials** - API-level authentication
5. **Project Selection** - Interactive picker with search
6. **IAM Configuration** - Set up required permissions
7. **API Enablement** - Enable Stitch API
8. **Connection Test** - Verify API access
9. **Config Generation** - Output ready-to-use MCP config

#### `doctor` - Health Checks

```bash
npx @_davideast/stitch-mcp doctor [options]
```

**Options:**
- `--verbose` - Show detailed error information

Diagnoses common setup issues and verifies:
- Google Cloud CLI is installed and accessible
- User is authenticated
- Application Default Credentials exist
- Active GCP project is configured
- Stitch API is reachable

#### `logout` - Revoke Credentials

```bash
npx @_davideast/stitch-mcp logout [options]
```

**Options:**
- `--force` - Skip confirmation prompts
- `--clear-config` - Delete entire gcloud config directory

Revokes both user authentication and Application Default Credentials. Useful for:
- Switching Google accounts
- Clearing authentication for testing
- Resetting state when troubleshooting

#### `proxy` - MCP Proxy Server

```bash
npx @_davideast/stitch-mcp proxy [options]
```

**Options:**
- `--transport <type>` - Transport type: 'stdio' or 'sse' (default: 'stdio')
- `--port <number>` - Port number (required for sse)
- `--debug` - Enable debug logging

This command is typically configured as the entry point in your MCP client settings. It handles:
- Automatic token refresh
- Request/response proxying
- Error handling
- Debug logging (when `--debug` is enabled to `/tmp/stitch-proxy-debug.log`)

### How It Works

#### Automatic gcloud Management

This library manages Google Cloud CLI:

- **Prefers global installation:** Uses existing `gcloud` if available
- **Auto-installs locally:** Downloads to `~/.stitch-mcp/google-cloud-sdk` if needed
- **Isolated configuration:** Separate config directory prevents config conflicts with other gcloud configurations

#### Two-Step Authentication

Two authentication flows are required for Stitch MCP server access:

1. **User Auth** (`gcloud auth login`)
   - Identifies you to Google Cloud
   - Opens browser for OAuth flow
   - **URL always printed to terminal** for manual copy-paste if browser fails

2. **Application Default Credentials** (`gcloud auth application-default login`)
   - Allows MCP server to make API calls on your behalf
   - Separate OAuth flow with API-level permissions

Both authentication URLs are automatically printed to your terminal, so you can always complete authentication manually if the browser doesn't open.

#### Transport Options

**Direct Connection (HTTP)** - Default for most clients:
```json
{
  "mcpServers": {
    "stitch": {
      "type": "http",
      "url": "https://stitch.googleapis.com/mcp",
      "headers": {
        "Authorization": "Bearer <token>",
        "X-Goog-User-Project": "<project-id>"
      }
    }
  }
}
```

**Proxy Mode (STDIO)** - Recommended for development:
```json
{
  "mcpServers": {
    "stitch": {
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"],
      "env": {
        "STITCH_PROJECT_ID": "<project-id>"
      }
    }
  }
}
```

Proxy mode handles token refresh automatically and provides debug logging.

### Troubleshooting

#### "Permission Denied" errors

Ensure:
- You have Owner or Editor role on the GCP project
- Billing is enabled on your project
- Stitch API is enabled

Run `doctor` to diagnose:
```bash
npx @_davideast/stitch-mcp doctor --verbose
```

#### Authentication URL not appearing

The tool now **always prints authentication URLs to the terminal** with a 5-second timeout to prevent hanging. If the URL doesn't appear:

1. Check your terminal output carefully
2. The URL starts with `https://accounts.google.com`
3. If still not visible, check `/tmp/stitch-proxy-debug.log` (if using proxy with `--debug`)

#### Already authenticated but showing logged in

The bundled gcloud SDK maintains separate authentication from your global gcloud installation. To fully clear authentication:

```bash
npx @_davideast/stitch-mcp logout --force --clear-config
```

#### API connection fails after setup

1. Run the doctor command:
   ```bash
   npx @_davideast/stitch-mcp doctor --verbose
   ```

2. Verify your project has billing enabled

3. Check that Stitch API is enabled:
   ```bash
   gcloud services list --enabled | grep stitch
   ```

4. Try re-authenticating:
   ```bash
   npx @_davideast/stitch-mcp logout --force
   npx @_davideast/stitch-mcp init
   ```

### Development

```bash
# Install dependencies
bun install

# Run locally
bun run dev init

# Run tests
bun test

# Build
bun run build

# Verify package
bun run verify-pack
```

## License

Apache 2.0 © David East

## Disclaimer

> [!WARNING]
> **Experimental Project** - This is an independent, experimental tool.

This project is:
- **NOT** affiliated with, endorsed by, or sponsored by Google LLC, Alphabet Inc., or the Stitch API team
- Provided **AS-IS** with **NO WARRANTIES** of any kind
- **NOT** guaranteed to be maintained, secure, or compatible with future API versions

"Stitch" and "Google Cloud" are trademarks of Google LLC.

**USE AT YOUR OWN RISK.**
