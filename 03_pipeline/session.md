# Session & Parameter Management

## The WorkflowSession

`WorkflowSession` is the data backbone of the pipeline. It acts as a persistent key-value store: Python objects are written to the session by one script and read back by any downstream script.

```python
from carbcomn.session import WorkflowSession

session = WorkflowSession(name="FloorStagStandardTest")

# Write a value
session["params"] = params
session.sync()           # persist everything to disk

# Read a value (auto-loads from disk if not in memory)
params = session["params"]
```

The session stores data as JSON files inside a `<name>.session/data/` directory, created alongside the `X00_init.py` script. For example:

```
FloorStagStandardTest.session/
├── tolerance.json
└── data/
    ├── params.json
    ├── refmesh.json
    ├── formdiagram.json
    ├── block_meshes.json
    ├── block_frames.json
    ├── blockmodel.json
    ├── blockmodel_results.json
    ├── grid.json
    └── buildingmodel.json
```

## Singleton behaviour

`WorkflowSession` is a **singleton per process**: once created with a given name, any subsequent call to `WorkflowSession(name=...)` in the same Python process returns the same instance. This means every script in a workflow simply calls `WorkflowSession(name="...")` at the top and gets the shared session without needing to pass it as an argument.

## Auto-sync and lazy loading

By default the session operates in **manual sync mode**: data written with `session["key"] = value` is held in memory and not written to disk until `session.sync()` is called explicitly (or `session.close(sync=True)`).

Setting `autosync=True` at construction writes every assignment to disk immediately, at the cost of additional I/O.

Values are **lazily loaded** from disk on first access: if `session["refmesh"]` is not in memory, the session looks for `refmesh.json` in the data directory and deserialises it automatically. This means a downstream script does not need to know whether an earlier script already ran in the same process.

## The params dictionary

All geometric and analysis parameters are stored in a single `params` dictionary written by the `X00_init.py` script:

```python
params = {
    "viewer": {
        "unit": "m",
        "camera": {"position": [-3.5, -4.0, 4.0], "target": [0, 0.5, 0.5]},
    },
    "grid": {
        "width": 6,          # floor plan width [m]
        "depth": 7.5,        # floor plan depth [m]
        "height": 3.2,       # storey height [m]
    },
    "floor": {
        "height": 0.5,                      # vault rise [m]
        "thickness": 0.3,                   # voussoir thickness [m]
        "voussoirs_count_span": 20,         # blocks along the span
        "voussoirs_count_length": 7,        # blocks along the length
        "stagger_type": "running",          # "none" | "running"
        "stagger_amount": 0.2,              # fractional offset (0–1)
        "number_of_radials": 20,            # TNA form diagram resolution
        "number_of_rings": 30,
        # ... material and shell thickness parameters
    },
    "elements": {
        "column_width": 0.2,
        "column_depth": 0.2,
        "beam_width": 0.3,
        "beam_depth": 0.5,
    },
    "problem": {},   # populated by solver scripts
}

session["params"] = params
session.sync()
```

All downstream scripts read `params = session["params"]` and extract what they need. Changing a parameter in `X00_init.py` and re-running the relevant downstream scripts is the standard way to iterate on a design.
