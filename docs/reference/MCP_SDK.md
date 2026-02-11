# MCP SDK - Quick Reference

## Protocol Primitives

- **Tools** (model-controlled): Functions Claude can call
- **Resources** (app-controlled): Read-only data identified by URIs
- **Prompts** (user-controlled): Reusable message templates

## Python SDK

```bash
pip install mcp
```

## Connecting to Claude Code

Project config (`.mcp.json` at project root):
```json
{
  "mcpServers": {
    "unity-rag": {
      "command": "python",
      "args": ["/path/to/unity_rag_server.py"],
      "env": {
        "QDRANT_URL": "${QDRANT_URL:-http://localhost:6333}"
      }
    }
  }
}
```

CLI alternative:
```bash
claude mcp add unity-rag -- python /path/to/unity_rag_server.py
```

## Connecting to Claude Desktop

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

## Available SDKs

| SDK | Status |
|---|---|
| Python | Stable (includes FastMCP) |
| TypeScript | Stable |
| C# | Stable |
| Kotlin | Stable |
| Go | Nearing stable |
| Rust | Available |

See `docs/architecture/ARCHITECTURE.md` for full details.
