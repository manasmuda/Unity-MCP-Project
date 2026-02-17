# CoplayDev Unity MCP - Setup, Use Cases & Tips

Companion reference for using [CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp) alongside our Unity MCP RAG Server. Coplay provides **live editor control** while our server provides **deep project understanding** — they work best together.

---

## Quick Setup

### Prerequisites

- Unity 2021.3 LTS or later
- Python 3.10+
- [uv](https://docs.astral.sh/uv/getting-started/installation/) package manager
- Claude Code or Claude Desktop

### 1. Install the Unity Package

In Unity Editor:

```
Window > Package Manager > + > Add package from git URL
```

Paste:

```
https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity#main
```

Alternative methods:
- **Asset Store**: search "MCP for Unity"
- **OpenUPM**: `openupm add com.coplaydev.unity-mcp`

### 2. Start the Server in Unity

```
Window > MCP for Unity > Start Server
```

This launches an HTTP server on `localhost:8080`.

### 3. Configure Claude Code

Option A — CLI:

```bash
claude mcp add unityMCP --transport http http://localhost:8080/mcp
```

Option B — add to `.mcp.json` at project root:

```json
{
  "mcpServers": {
    "unityMCP": {
      "url": "http://localhost:8080/mcp"
    }
  }
}
```

### 4. Verify

- Unity MCP panel shows green **"Connected"** status
- In Claude Code, run `/mcp` to confirm `unityMCP` is listed

---

## Available Tools (35+)

| Category | Tools | What They Do |
|---|---|---|
| **GameObjects** | `manage_gameobject`, `find_gameobjects` | Create, delete, move, rename, query objects |
| **Components** | `manage_components` | Add, remove, modify components on objects |
| **Scripts** | `create_script`, `manage_script`, `apply_text_edits`, `validate_script` | Write C# scripts with Roslyn validation |
| **Scenes** | `manage_scene` | Load, save, create scenes |
| **Prefabs** | `manage_prefabs` | Create, instantiate, modify prefabs |
| **Materials** | `manage_material`, `manage_shader`, `manage_texture` | Material/shader/texture operations |
| **Animation** | `manage_animation` | Animation clip and controller management |
| **VFX** | `manage_vfx` | Visual effects management |
| **Assets** | `manage_asset` | Import, move, delete project assets |
| **Editor** | `manage_editor`, `execute_menu_item`, `read_console`, `refresh_unity` | Editor state, menus, console output |
| **Testing** | `run_tests`, `get_test_job` | Run and monitor Unity tests |
| **Performance** | `batch_execute` | Batch multiple operations (10-100x faster) |
| **Extensibility** | `execute_custom_tool` | Run user-defined custom tools |

---

## Most Useful Use Cases

### 1. Rapid Scene Prototyping

```
"Create a 5x5 grid of cubes spaced 2 units apart,
 assign random colors to each"
```

Builds out scene geometry instantly — great for blockouts and layout iteration.

### 2. Script Scaffolding

```
"Create a PlayerController script with WASD movement,
 jumping, and ground detection using raycasts"
```

Generates `.cs` files with Roslyn-based compile validation before you even switch to Unity.

### 3. Batch Asset Operations

```
"Add a Rigidbody and BoxCollider to every GameObject tagged 'Pickup'"
"Rename all child objects of 'Level1' to use snake_case"
"Switch all materials from Standard to URP/Lit shader"
```

Use `batch_execute` for bulk operations — 10-100x faster than individual calls.

### 4. Debugging & Inspection

```
"Read the last 20 console errors"
"Show me all GameObjects in the current scene with MeshRenderer"
"What components are on the Player object?"
```

Query editor state without switching windows.

### 5. Test-Driven Development

```
"Run all EditMode tests and show me failures"
```

Integrate Unity test runs directly into your AI workflow.

### 6. Prefab Workflows

```
"Create a prefab from the 'Enemy' GameObject"
"Instantiate the 'Checkpoint' prefab at positions (0,0,10), (0,0,20), (0,0,30)"
```

### 7. Combined with Our RAG Server

Use our Unity MCP RAG server to **understand** the project, then Coplay to **act**:

```
Step 1 (RAG):  "Which scripts handle inventory management?"
Step 2 (RAG):  "What assets reference the InventoryItem ScriptableObject?"
Step 3 (Coplay): "Add a 'stackable' bool field to the InventoryItem script"
Step 4 (Coplay): "Create a new ScriptableObject asset for HealthPotion"
```

---

## Security & Privacy

| Aspect | Detail |
|---|---|
| **Network** | Loopback only (`localhost:8080`) by default — no external access |
| **Data sent externally** | None — all communication stays on your machine |
| **Telemetry** | Anonymous (no code, project names, or personal data). Opt out: `DISABLE_TELEMETRY=true` |
| **Remote access** | Disabled by default. Requires explicit opt-in + forces HTTPS |
| **License** | MIT — commercial use permitted |
| **Affiliation** | Community project, not officially by Unity Technologies |

### Recommended Settings for Work Projects

```bash
# Disable telemetry
export DISABLE_TELEMETRY=true

# Or set in your shell profile (~/.bashrc, ~/.zshrc)
echo 'export DISABLE_TELEMETRY=true' >> ~/.bashrc
```

- Do NOT enable remote/LAN access
- Always use git — the AI can modify and delete project files
- Review changes before saving scenes

---

## Tips

1. **Always use version control.** Coplay can create, modify, and delete GameObjects, scripts, and assets. Git is your safety net.

2. **Use `batch_execute` for bulk work.** Individual calls have HTTP overhead. Batching is 10-100x faster.

3. **Validate scripts before switching to Unity.** The `validate_script` tool uses Roslyn to catch C# errors without waiting for Unity recompilation.

4. **Start with `read_console`** when debugging. Get Unity's console output without alt-tabbing.

5. **Combine both MCP servers.** Register both `unityMCP` (Coplay) and `unity-rag` (our server) in your `.mcp.json`. Use RAG for understanding, Coplay for action.

6. **Use the `#beta` branch** for latest features:
   ```
   https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity#beta
   ```

7. **Check the roadmap** for upcoming features: [Project Roadmap](https://github.com/CoplayDev/unity-mcp/wiki/Project-Roadmap)

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Server won't start | Verify `uv --version` works in your terminal |
| Client can't connect | Confirm server is running in Unity MCP panel |
| Port 8080 in use | Check for other services on that port |
| Tools not appearing | Restart Claude Code after configuring `.mcp.json` |
| Script validation fails | Ensure Unity project compiles cleanly first |

For more help: [Wiki](https://github.com/CoplayDev/unity-mcp/wiki) | [Discord](https://discord.gg/y4p8KfzrN4)
