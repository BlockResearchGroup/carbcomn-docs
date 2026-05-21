# Session API

## WorkflowSession

See [Session & Parameter Management](../03_pipeline/session.md) for a full usage guide.

```python
from carbcomn.session import WorkflowSession
```

### Constructor

```python
session = WorkflowSession(
    name="FloorStagStandardTest",   # session name (also the .session/ directory name)
    parentdir=None,                 # defaults to directory of the calling script
    autosync=False,                 # if True, writes to disk on every assignment
    autosave=False,
    new_instance=False,             # if True, destroys any existing singleton and creates fresh
)
```

### Data access

| Operation | Code | Description |
|-----------|------|-------------|
| Write | `session["key"] = value` | Store in memory (+ disk if `autosync=True`) |
| Read | `value = session["key"]` | Load from memory, or from disk if not in memory |
| Delete | `del session["key"]` | Remove from memory and disk |
| Check | `"key" in session` | Test for existence in memory |
| List keys | `session.keys()` | All keys in memory and/or data directory |

### Persistence

| Method | Description |
|--------|-------------|
| `session.sync()` | Write all in-memory data to disk |
| `session.sync_item(key, value)` | Write a single item to disk |
| `session.close(sync=True)` | Sync and clear in-memory data |
| `session.clear()` | Delete all data (memory + disk) |

### File layout

```
<parentdir>/
└── <name>.session/
    ├── tolerance.json
    └── data/
        ├── params.json
        ├── refmesh.json
        └── ...
```

All values are serialised as JSON using COMPAS's `compas.json_dump` / `compas.json_load`. COMPAS geometric objects (`Mesh`, `Frame`, `Point`, etc.) round-trip correctly through this serialisation.
