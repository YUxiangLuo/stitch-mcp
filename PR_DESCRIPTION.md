# Fix VSCode MCP Configuration Format & Add Token Hint

## Summary

This PR fixes the VSCode MCP configuration format to comply with the [official VS Code MCP documentation](https://code.visualstudio.com/docs/copilot/customization/mcp-servers) and adds helpful instructions for users to obtain access tokens when using the Direct (HTTP) transport mode.

## Changes

### 1. VSCode Configuration Format Fix

**Before:**
```json
{
  "servers": {
    "stitch": {
      "url": "https://stitch.googleapis.com/mcp",
      "type": "http",
      "headers": {
        "Accept": "application/json",
        "Authorization": "Bearer $STITCH_ACCESS_TOKEN",
        "X-Goog-User-Project": "$GOOGLE_CLOUD_PROJECT"
      }
    }
  }
}
```

**After (HTTP mode):**
```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "stitch-access-token",
      "description": "Google Cloud Access Token (run: gcloud auth print-access-token)",
      "password": true
    }
  ],
  "servers": {
    "stitch": {
      "type": "http",
      "url": "https://stitch.googleapis.com/mcp",
      "headers": {
        "Authorization": "Bearer ${input:stitch-access-token}",
        "X-Goog-User-Project": "<project-id>"
      }
    }
  }
}
```

**After (STDIO mode):**
```json
{
  "servers": {
    "stitch": {
      "type": "stdio",
      "command": "npx",
      "args": ["@_davideast/stitch-mcp", "proxy"],
      "env": {
        "STITCH_PROJECT_ID": "<project-id>"
      }
    }
  }
}
```

### 2. Token Hint for Direct (HTTP) Mode

Since `stitch-mcp init` installs gcloud to a non-standard location (`~/.stitch-mcp/google-cloud-sdk/`), users don't know how to obtain access tokens. 

Now, when using HTTP transport, the CLI shows:

```
To get your access token, run:
  CLOUDSDK_CONFIG=~/.stitch-mcp/config ~/.stitch-mcp/google-cloud-sdk/bin/gcloud auth print-access-token

Note: Access tokens expire after 1 hour. Consider using stdio transport for automatic refresh.
```

This hint is now shown for all HTTP-mode clients:
- ✅ VSCode
- ✅ Cursor
- ✅ Antigravity
- ✅ Claude Code

## Why These Changes?

1. **VSCode compliance**: The previous config used `$ENV_VAR` syntax which is not supported by VS Code's mcp.json format. The official format requires `${input:variable-id}` for sensitive data.

2. **User experience**: Users who complete `init` with bundled gcloud have no idea where gcloud is installed or how to get tokens for HTTP mode clients.

3. **Encourage STDIO**: The warning about token expiration encourages users to use STDIO transport which handles token refresh automatically.

## Testing

```bash
# Test VSCode HTTP config
bun run dev init --client vscode --transport http --yes

# Test VSCode STDIO config  
bun run dev init --client vscode --transport stdio --yes

# Test other clients
bun run dev init --client cursor --transport http --yes
bun run dev init --client claude-code --transport http --yes
```

## Related

- [VS Code MCP Server Documentation](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)
