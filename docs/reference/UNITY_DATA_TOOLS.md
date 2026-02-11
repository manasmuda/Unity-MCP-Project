# UnityDataTools - Quick Reference

Official Unity-Technologies CLI for inspecting binary asset files.

- **Repo:** https://github.com/Unity-Technologies/UnityDataTools
- **Version:** v1.3.1
- **License:** Unity Companion License
- **Requires:** .NET 9.0 SDK

## Commands

| Command | Purpose |
|---|---|
| `analyze <path>` | Extract asset data into SQLite DB |
| `dump <path>` | Convert SerializedFiles to human-readable text |
| `archive list <path>` | List contents of a Unity Archive |
| `archive extract <path>` | Extract Unity Archive contents |
| `sf objectlist <file>` | List objects in a SerializedFile |
| `sf externalrefs <file>` | List external references |
| `find-refs <db>` | Trace reference chains (experimental) |

## Key Limitations

- TypeTrees must be enabled in player builds
- `find-refs` is experimental
- `--skip-references` is faster but disables ref tracing and duplicate detection
- Platform containers (APK, IPA) must be extracted first
- Unity Companion License - not permissive open-source

See `docs/architecture/ARCHITECTURE.md` for full details.
