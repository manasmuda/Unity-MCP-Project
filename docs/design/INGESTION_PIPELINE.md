# Ingestion Pipeline Design

## What Gets Parsed

```
Unity Project Folder
    |
    +- Assets/**/*.meta                    -> GUID extraction (Python YAML)
    +- Assets/**/*.prefab                  -> GameObject/component hierarchy (Python YAML)
    +- Assets/**/*.unity                   -> Scene hierarchy (Python YAML)
    +- Assets/**/*.asset                   -> ScriptableObjects (Python YAML)
    +- Assets/**/*.mat                     -> Material properties (Python YAML)
    +- Assets/**/*.controller              -> Animator controllers (Python YAML)
    +- Assets/**/*.cs                      -> C# scripts (plain text)
    |
    +- Library/Bee/*, Build output         -> UnityDataTools analyze -> SQLite
    +- AssetBundles/**                     -> UnityDataTools analyze + dump
    +- StreamingAssets/aa/**               -> UnityDataTools (Addressables)
    |
    +- ProjectSettings/*.asset             -> Project config (Python YAML)
    +- Packages/manifest.json              -> Package dependencies (JSON)
    +- Packages/packages-lock.json         -> Resolved packages (JSON)
```

## Chunking Strategy

Don't embed entire files. Chunk by semantic unit:

1. **Per-component chunks** - Each MonoBehaviour/component on a GameObject becomes its own chunk
2. **Per-material chunks** - Each material with its shader + property block
3. **Per-script chunks** - C# files chunked by class/method
4. **Per-scene-subtree chunks** - Scene hierarchy broken into subtrees
5. **Per-asset-reference chunks** - Cross-reference entries (which asset references what)
6. **Project config chunks** - Individual settings sections

## Metadata to Store with Each Chunk

```python
{
    "file_path": "Assets/Characters/Player/PlayerController.prefab",
    "guid": "abc123def456...",        # from .meta file
    "asset_type": "Prefab",
    "chunk_type": "component",        # component | material | script | hierarchy | config
    "component_type": "MonoBehaviour", # Unity type
    "script_reference": "PlayerMovement",
    "references": ["guid1", "guid2"], # GUIDs this chunk depends on
    "project_name": "MyGame",
    "file_size": 4096,
    "unity_version": "2022.3.10f1"
}
```
