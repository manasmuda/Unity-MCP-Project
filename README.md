# Unity MCP RAG Server

A local MCP server that gives Claude deep contextual understanding of Unity projects by parsing assets, embedding them into a vector database, and exposing search/query tools via the MCP protocol.

- **Stack:** Python (FastMCP) + Qdrant (vector DB) + sentence-transformers (embeddings) + UnityDataTools (binary assets)
- **Transport:** MCP over stdio (works with Claude Code and Claude Desktop)
- **Runs entirely locally** - no API keys or cloud services required

## Prerequisites

### Python 3.12+

Required. Check with:
```bash
python3 --version
```

### Docker (for Qdrant in production mode)

Qdrant can run in-memory for development (no Docker needed), but production use requires Docker.

Install Docker: https://docs.docker.com/engine/install/

Verify:
```bash
docker --version
docker compose version
```

### .NET 9.0 SDK (for UnityDataTools)

Required only if you need to parse binary Unity assets (AssetBundles, SerializedFiles, player builds).

Install: https://dotnet.microsoft.com/en-us/download/dotnet/9.0

Verify:
```bash
dotnet --version
```

### UnityDataTools (optional, for binary asset parsing)

Official Unity-Technologies CLI for inspecting binary asset files.

```bash
# Clone (outside this project - it's .gitignored)
git clone https://github.com/Unity-Technologies/UnityDataTools.git

# Build (requires .NET 9.0 SDK)
cd UnityDataTools
dotnet build -c Release

# Binary will be at: UnityDataTools/UnityDataTool/bin/Release/net9.0/UnityDataTool
```

Set the path in your environment:
```bash
export UNITY_DATA_TOOL_PATH="/path/to/UnityDataTool/bin/Release/net9.0/UnityDataTool"
```

See `docs/reference/UNITY_DATA_TOOLS.md` for full command reference.

## Setup

### 1. Clone the repo

```bash
git clone <repo-url>
cd Unity-MCP-Project
```

### 2. Create virtual environment and install dependencies

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 3. Start Qdrant (choose one)

**Option A - In-memory (development, no Docker needed):**

No setup needed. The server will use `QdrantClient(':memory:')` by default in dev mode.

**Option B - Docker (persistent storage):**

```bash
docker compose up -d
```

This starts Qdrant on `localhost:6333`. Data persists in a Docker volume.

### 4. Run the MCP server

```bash
source .venv/bin/activate
python src/unity_mcp_rag/server.py
```

### 5. Connect to Claude

**Claude Code (project config):**

Add to `.mcp.json` at project root:
```json
{
  "mcpServers": {
    "unity-rag": {
      "command": "python",
      "args": ["src/unity_mcp_rag/server.py"],
      "env": {
        "QDRANT_URL": "${QDRANT_URL:-http://localhost:6333}"
      }
    }
  }
}
```

Or via CLI:
```bash
claude mcp add unity-rag -- python src/unity_mcp_rag/server.py
```

**Claude Desktop:**

Add to config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, `%APPDATA%\Claude\claude_desktop_config.json` on Windows):
```json
{
  "mcpServers": {
    "unity-rag": {
      "command": "python",
      "args": ["/absolute/path/to/src/unity_mcp_rag/server.py"],
      "env": {
        "QDRANT_URL": "http://localhost:6333"
      }
    }
  }
}
```

## Project Structure

```
Unity-MCP-Project/
|-- CLAUDE.md                  # Project conventions (for Claude Code)
|-- README.md                  # This file - setup and usage guide
|-- requirements.txt           # Python dependencies
|-- docker-compose.yml         # Qdrant container config
|-- .gitignore
|-- docs/                      # All project documentation
|   |-- architecture/          # System architecture and high-level design
|   |-- design/                # Feature design and specifications
|   |-- setup/                 # Deployment and environment instructions
|   +-- reference/             # Quick-reference guides for external tools
+-- src/
    +-- unity_mcp_rag/         # Main Python package
```

## Documentation

- Architecture: `docs/architecture/ARCHITECTURE.md`
- Ingestion pipeline design: `docs/design/INGESTION_PIPELINE.md`
- MCP tool specs: `docs/design/MCP_TOOLS.md`
- Deployment details: `docs/setup/DEPLOYMENT.md`
- UnityDataTools reference: `docs/reference/UNITY_DATA_TOOLS.md`
- MCP SDK reference: `docs/reference/MCP_SDK.md`
