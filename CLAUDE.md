# Unity MCP RAG Server

A local MCP server that gives Claude deep contextual understanding of Unity projects by parsing assets, embedding them into a vector database, and exposing search/query tools via the MCP protocol.

## Quick Summary

- **What:** MCP server that ingests Unity project files and provides semantic search + structured queries
- **Stack:** Python (FastMCP) + Qdrant (vector DB) + sentence-transformers (embeddings) + UnityDataTools (binary assets)
- **Transport:** MCP over stdio (works with Claude Code and Claude Desktop)
- **Runs entirely locally** - no API keys or cloud services required

## Project Structure

```
Unity-MCP-Project/
|-- CLAUDE.md                  # This file - project intro and conventions
|-- docs/                      # All project documentation
|   |-- architecture/          # System architecture and high-level design
|   |   +-- ARCHITECTURE.md    # Full architecture doc with tech stack, diagrams, research
|   |-- design/                # Feature design and specifications
|   |   |-- INGESTION_PIPELINE.md  # What gets parsed, chunking strategy, metadata schema
|   |   +-- MCP_TOOLS.md          # Tool specifications and design decisions
|   |-- setup/                 # Deployment, environment, and build instructions
|   |   +-- DEPLOYMENT.md     # Docker Compose, Python/dotNET requirements, implementation phases
|   +-- reference/             # Quick-reference guides for external tools and SDKs
|       |-- UNITY_DATA_TOOLS.md  # UnityDataTools CLI command reference
|       +-- MCP_SDK.md           # MCP SDK setup and connection reference
+-- src/                       # Source code (to be created)
```

## Docs Folder Organization

All documentation lives under `docs/` in topic-based subfolders:

| Folder | Purpose | When to add here |
|---|---|---|
| `docs/architecture/` | System-level architecture, diagrams, tech stack decisions | High-level design that affects the whole system |
| `docs/design/` | Feature specs, pipeline design, tool definitions | Detailed design for a specific feature or subsystem |
| `docs/setup/` | Deployment configs, environment setup, build steps | Anything needed to get the project running |
| `docs/reference/` | Quick-reference cards for external tools and libraries | Cheat sheets and condensed API docs for dependencies |

**Conventions:**
- Use UPPER_SNAKE_CASE for doc filenames (e.g., `INGESTION_PIPELINE.md`)
- Each doc should be self-contained but can link to others via relative paths
- Keep reference docs concise; point to architecture docs for full details

## Key Architecture Decisions

1. **Hybrid retrieval** - Vector search for semantic queries + raw SQLite for structured queries
2. **UnityPy for text assets, UnityDataTools for binary** - keeps server in Python while leveraging official C# CLI for binary formats
3. **GUIDs as canonical keys** - Unity .meta GUIDs used for cross-referencing across the entire system
4. **Chunking by semantic unit** - per-component, per-material, per-script (not whole files)
5. **Incremental indexing** - only re-process changed files on subsequent ingests

## Development

- Python server code will live in `src/`
- Qdrant runs locally via Docker (see `docs/setup/DEPLOYMENT.md`)
- UnityDataTools requires .NET 9.0 SDK (see `docs/reference/UNITY_DATA_TOOLS.md`)
- Full architecture details: `docs/architecture/ARCHITECTURE.md`
