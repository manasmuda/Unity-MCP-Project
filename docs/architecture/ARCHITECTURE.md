# Unity MCP RAG Server - Architecture & Research Document

## Project Goal

Build a local MCP server that:
1. Accepts a Unity project as input
2. Parses all Unity asset files into readable formats
3. Embeds parsed data into a vector database (RAG pipeline)
4. Exposes search/query tools to Claude via MCP protocol

This gives Claude deep contextual understanding of Unity project asset structures, references, dependencies, and content.

---

## Architecture Overview

```
┌─────────────────┐     ┌──────────────────────────────┐     ┌─────────────┐
│  Unity Project  │────▶│  MCP Server (Python)         │◀───▶│ Claude Code │
│  (folder)       │     │                              │     │ / Desktop   │
└─────────────────┘     │  1. Ingest pipeline          │     └─────────────┘
                        │     ├─ UnityDataTools (CLI)   │
                        │     ├─ UnityPy (Python)       │
                        │     └─ YAML parser            │
                        │  2. Chunking + Embedding      │
                        │     └─ sentence-transformers  │
                        │  3. Vector DB (Qdrant)        │
                        │  4. MCP tools (search,        │
                        │     inspect, list, query)     │
                        └──────────────────────────────┘
```

---

## Technology Stack

| Component | Choice | Why |
|---|---|---|
| **Asset parser (binary)** | [UnityDataTools](https://github.com/Unity-Technologies/UnityDataTools) v1.3.1 (CLI) | Official Unity-Technologies tool. Reads AssetBundles, SerializedFiles, player build output. Outputs SQLite DB and human-readable text dumps. |
| **Asset parser (YAML/text)** | [UnityPy](https://pypi.org/project/UnityPy/) (Python) | Python-native parser for `.prefab`, `.unity`, `.asset`, `.meta` YAML files. Keeps entire server in one language. |
| **Server framework** | Python + [FastMCP](https://github.com/modelcontextprotocol/python-sdk) (`pip install mcp`) | First-class MCP support, fastest to build, great for RAG |
| **Vector DB** | [Qdrant](https://qdrant.tech/) (local Docker or in-memory) | Official MCP reference server exists, runs locally, no API key needed |
| **Embeddings** | `sentence-transformers/all-MiniLM-L6-v2` via FastEmbed | Runs locally, no API calls, good quality for code/structured data |
| **Connection to Claude** | MCP over stdio | Works with both Claude Code and Claude Desktop |

---

## UnityDataTools - Detailed Reference

### Repository
- **URL:** https://github.com/Unity-Technologies/UnityDataTools
- **Owner:** Unity-Technologies (official)
- **License:** Unity Companion License (not MIT/Apache - review terms before redistribution)
- **Language:** C# (.NET 9.0)
- **Current version:** v1.3.1 (last pushed January 27, 2026)
- **Native dependency:** `UnityFileSystemApi` (C/C++ library, bundled in repo and also shipped with Unity Editor 2022.1+ at `{UnityEditor}/Data/Tools/`)

### Supported Unity File Types

| File Type | Support |
|---|---|
| AssetBundles (Unity Archives) | Full - open, list, extract, analyze, dump |
| SerializedFiles (binary objects) | Full - analyze, dump to text, list objects/refs |
| Player build output (level0, sharedAssets, globalgamemanagers, resources.assets) | Supported (requires TypeTrees enabled in build) |
| Compressed Player builds (`data.unity3d`) | Supported (treated as archives) |
| Addressables build output (`StreamingAssets/aa`) | Full, including BuildLayout JSON |
| Entities content (`StreamingAssets/ContentArchives`) | Supported |
| BuildReport files (`.buildreport`) | Supported (v1.3.0+) |
| Addressables BuildLayout JSON | Supported |
| WebGL `.data` files | Supported via archive command |
| AssetDatabase Artifacts (Library folder) | Partial/experimental |

### CLI Commands

#### `analyze` - Extract data into SQLite
```bash
UnityDataTool analyze <path> [options]
  -o, --output-file <file>        # Output database filename (default: database.db)
  -p, --search-pattern <pattern>  # File search pattern (default: *)
  -s, --skip-references           # Skip CRC and reference extraction (much faster)
  -v, --verbose                   # Show more info during analysis
  --no-recurse                    # Do not recurse into sub-directories
```

SQLite output includes tables: `objects`, `serialized_files`, `types`, `refs`, `assets`, `mono_script`
Pre-built views: `view_potential_duplicates`, `view_material_shader_refs`, `view_material_texture_refs`, `script_object_view`

#### `dump` - Convert SerializedFiles to human-readable text
```bash
UnityDataTool dump <path> [options]
  -o, --output-path <path>        # Output folder
  -f, --output-format <format>    # Output format (default: text)
  -s, --skip-large-arrays         # Skip dumping large arrays
  -i, --objectid <id>             # Only dump a specific object by ID
```

#### `archive` - List or extract Unity Archive contents
```bash
UnityDataTool archive list <archive-path>
UnityDataTool archive extract <archive-path> [options]
  -o, --output-path <path>        # Output directory (default: archive)
```

#### `serialized-file` (alias: `sf`) - Quick inspection
```bash
UnityDataTool sf objectlist <filename> [-f Text|Json]
UnityDataTool sf externalrefs <filename> [-f Text|Json]
```

#### `find-refs` - Trace reference chains (experimental)
```bash
UnityDataTool find-refs <database> [options]
  -i, --object-id <id>            # ID of object to analyze
  -n, --object-name <name>        # Name of objects to analyze
  -t, --object-type <type>        # Type filter when using -n
  -o, --output-file <file>        # Output filename
  -a, --find-all                  # Find all chains (not just first)
```

### Installation

```bash
# Clone
git clone https://github.com/Unity-Technologies/UnityDataTools.git

# Requires .NET 9.0 SDK
# https://dotnet.microsoft.com/en-us/download/dotnet/9.0

# Build
dotnet build -c Release

# Output: UnityDataTool/bin/Release/net9.0/UnityDataTool[.exe]
```

Prebuilt binaries available from GitHub Actions artifacts (Windows + Mac).

### Important Limitations

1. **TypeTrees required** for player builds - must enable `ForceAlwaysWriteTypeTrees` in Unity Editor Preferences
2. **Experimental status** - "provided on an 'as-is' basis, not officially supported by Unity"
3. **`find-refs` is experimental** - may not work in all cases
4. **`--skip-references` trade-off** - dramatically faster (208s->9s, 500MB->68MB) but disables ref tracing, duplicate detection, material/shader views
5. **Duplicate SerializedFile names cause SQL errors** - analyze each build separately
6. **`Resources/unity default resources` always fails** - no TypeTrees, expected error
7. **Platform containers** (APK, IPA) must be extracted first
8. **Unity Companion License** - not permissive open-source, review for redistribution

### Alternative / Complementary Tools

| Tool | URL | Use Case |
|---|---|---|
| **UnityPy** | pypi.org/project/UnityPy | Python-native asset extractor/patcher, great for scripting |
| **AssetRipper** | github.com/AssetRipper/AssetRipper | GUI for loading/converting/exporting assets (C#) |
| **UABEA** | github.com/nesrak1/UABEA | Asset Bundle Extractor Advanced (modding/research) |
| **AssetStudio** | github.com/Perfare/AssetStudio | Asset explorer/extractor (no longer actively maintained) |

---

## MCP Server - Detailed Reference

### MCP Protocol Overview

MCP (Model Context Protocol) is the standard for connecting AI models to external tools and data. Three core primitives:

- **Tools** (model-controlled): Functions Claude can call (search, query, ingest)
- **Resources** (app-controlled): Read-only data identified by URIs
- **Prompts** (user-controlled): Reusable message templates

### Python SDK

```bash
pip install mcp
```

The official SDK bundles FastMCP. Basic server:

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Unity RAG Server")

@mcp.tool()
def search_assets(query: str, top_k: int = 10) -> str:
    """Semantic search across all ingested Unity assets."""
    # embed query, search Qdrant, return results
    pass

if __name__ == "__main__":
    mcp.run()  # stdio transport by default
```

### Available MCP SDKs

| SDK | Repository | Status |
|---|---|---|
| **Python** | modelcontextprotocol/python-sdk | Stable, includes FastMCP |
| **TypeScript** | modelcontextprotocol/typescript-sdk | Stable |
| **C#** | modelcontextprotocol/csharp-sdk | Stable (Microsoft collab) |
| **Kotlin** | modelcontextprotocol/kotlin-sdk | Stable (JetBrains collab) |
| **Go** | modelcontextprotocol/go-sdk | Nearing stable (Google collab) |
| **Rust** | Official | Available |

### Connecting to Claude Code

Option A - Project config (`.mcp.json` at project root):
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

Option B - CLI:
```bash
claude mcp add unity-rag -- python /path/to/unity_rag_server.py
```

Scopes: `local` (default, private), `project` (shared via VCS), `user` (all projects).

### Connecting to Claude Desktop

Config file locations:
- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "unity-rag": {
      "command": "python",
      "args": ["/path/to/unity_rag_server.py"],
      "env": {
        "QDRANT_URL": "http://localhost:6333"
      }
    }
  }
}
```

### Reference RAG MCP Servers

| Server | Repository | Notes |
|---|---|---|
| **Qdrant MCP Server** (official) | github.com/qdrant/mcp-server-qdrant | Best reference impl. `qdrant-store` + `qdrant-find` tools |
| **mcp-rag-server** | github.com/kwanLeeFrmVi/mcp-rag-server | Document indexing and retrieval |
| **ChromaDB MCP Server** | PulseMCP registry | Semantic document search |
| **Pinecone MCP Server** | PulseMCP registry | Vector search and RAG |
