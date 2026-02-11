# MCP Tools Specification

## Tools to Expose

### `ingest_unity_project`
Ingest a Unity project: parse all assets and populate the vector DB.
Handles .prefab, .unity, .asset, .meta, .cs, AssetBundles, and build output.

### `search_assets`
Semantic search across all ingested Unity assets.
Example: 'player movement controller' or 'transparent shader'.
Parameters: `query: str`, `top_k: int = 10`

### `list_asset_types`
List all asset types and their counts in the ingested project.
Parameters: `project: str = None`

### `get_asset_by_guid`
Get full parsed content of a specific asset by its Unity GUID.
Parameters: `guid: str`

### `get_asset_by_path`
Get full parsed content of an asset by its file path.
Parameters: `file_path: str`

### `find_references_to`
Find all assets that reference a given asset (incoming dependencies).
Parameters: `asset_name: str`

### `find_dependencies_of`
Find all assets that a given asset depends on (outgoing dependencies).
Parameters: `asset_name: str`

### `find_duplicates`
Find duplicate assets across the project using CRC comparison.

### `query_project_db`
Run a read-only SQL query against the UnityDataTools SQLite database.
Available tables: `objects`, `serialized_files`, `types`, `refs`, `assets`, `mono_script`.
Available views: `view_potential_duplicates`, `view_material_shader_refs`, etc.
Parameters: `sql: str`

### `get_scene_hierarchy`
Get the full GameObject hierarchy of a scene.
Parameters: `scene_name: str`

### `get_project_summary`
Get a high-level summary of the ingested project: stats, asset counts, package dependencies, Unity version, render pipeline, etc.

## Design Decisions

1. **Hybrid retrieval** - Use vector search for semantic queries ("find player movement logic") AND direct SQLite for structured queries ("list all textures > 2048px"). Both are exposed as separate MCP tools.

2. **Keep the SQLite DB** - UnityDataTools `analyze` produces a rich SQLite DB with pre-built views. Don't throw it away after embedding. Expose it as a direct query tool alongside vector search.

3. **GUIDs as the canonical key** - Unity `.meta` files contain GUIDs that uniquely identify every asset. Store these in vector DB metadata for cross-referencing.

4. **UnityPy vs UnityDataTools** - Use UnityPy (Python) for text-based YAML files (.prefab, .unity, .asset, .meta). Use UnityDataTools (C# CLI) for binary serialized files and AssetBundles. This keeps the main server in Python while leveraging the official tool for binary formats.

5. **Local-only** - Everything runs locally (Qdrant in Docker, embeddings via sentence-transformers, MCP over stdio). No API keys or cloud services required.

6. **Incremental indexing** - Track file modification times and GUIDs. On re-ingest, only process changed files. Store a manifest of indexed files with their timestamps and content hashes.
