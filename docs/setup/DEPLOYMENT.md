# Deployment Setup

## Docker Compose (Qdrant + Server)

```yaml
version: '3.8'
services:
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_data:/qdrant/storage

  # Optional: run the MCP server as a container too
  # unity-rag:
  #   build: .
  #   depends_on:
  #     - qdrant
  #   environment:
  #     - QDRANT_URL=http://qdrant:6333

volumes:
  qdrant_data:
```

## Python Requirements

```
mcp>=1.0
qdrant-client>=1.7
sentence-transformers>=2.2
fastembed>=0.2
pyyaml>=6.0
unitypy>=1.10
```

## .NET Requirements (for UnityDataTools)

```
.NET 9.0 SDK
```

## Implementation Phases

### Phase 1: Core Infrastructure
- [ ] Set up Python project structure
- [ ] Install and build UnityDataTools CLI
- [ ] Set up Qdrant (Docker or in-memory for dev)
- [ ] Create basic FastMCP server skeleton

### Phase 2: Ingest Pipeline
- [ ] YAML parser for .prefab, .unity, .asset, .meta files
- [ ] GUID extraction from .meta files
- [ ] C# script reader
- [ ] UnityDataTools CLI wrapper (analyze + dump)
- [ ] ProjectSettings and Packages parser
- [ ] Chunking logic (per-component, per-material, etc.)
- [ ] Embedding generation and Qdrant storage

### Phase 3: MCP Tools
- [ ] `ingest_unity_project` tool
- [ ] `search_assets` tool (vector search)
- [ ] `query_project_db` tool (SQL on SQLite)
- [ ] `get_asset_by_guid` / `get_asset_by_path` tools
- [ ] `find_references_to` / `find_dependencies_of` tools
- [ ] `find_duplicates` tool
- [ ] `get_scene_hierarchy` tool
- [ ] `get_project_summary` tool

### Phase 4: Polish & Testing
- [ ] Test with real Unity projects
- [ ] Optimize chunking for retrieval quality
- [ ] Add progress reporting during ingest
- [ ] Add incremental re-indexing (only changed files)
- [ ] Documentation and setup guide
