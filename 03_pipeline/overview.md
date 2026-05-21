# Pipeline Architecture

The `carbcomn.core` pipeline is a **staged, session-based workflow**. Each stage is implemented as one or more independent Python scripts that read their inputs from a persistent `WorkflowSession` and write their outputs back to it. Stages can be re-run in isolation, allowing targeted iteration without repeating expensive earlier computations.

## Stage sequence

```
X00  Init               ─── define session + params
  │
X10  TNA                ─── FloorFormDiagram + Pattern → RefMesh
  │                         (skipped for arch / three-block examples)
X20  Geometry           ─── Template → block_meshes, block_frames
  │
X30  RefBlock           ─── block_meshes → typed StructuralElements
  │                         (skipped for StandardBlock-only workflows)
X40  DEM Model          ─── BlockModel.compute_contacts()
  │
X41  DEM Problem        ─── Problem + Solver → blockmodel_results
  │
X42  DEM Visualisation  ─── DEMViewer (read-only, no session write)
  │
X50  Structural Grid    ─── StructuralGrid + Column + TieBeam
  │                         (full-floor workflows only)
X51  Floor Assembly     ─── FloorModel merges vault + frame
```

## Session as the central data store

Every stage reads from and writes to a `WorkflowSession` object. The session stores named Python objects as JSON files inside a `<name>.session/data/` directory. This means:

- Scripts are self-contained and can be run in any order (as long as their dependencies have been written to the session first)
- Intermediate results are automatically persisted to disk — no data is lost if a script fails midway
- Parameters propagate automatically: all stages read from `session["params"]`, which is written once in `X00_init.py`

See [Session & Parameter Management](session.md) for details.

## Which stages does each example use?

| Example | TNA | RefBlock | Struct. Frame |
|---------|-----|----------|---------------|
| 000 — Three Blocks | — | — | — |
| 100 — Arch | — | — | — |
| 200 — Floor, no stagger | ✓ | — | — |
| 300 — Floor, running stagger | ✓ | — | ✓ |
| 400 — Floor, ridge voussoirs | ✓ | ✓ | ✓ |
| 500 — Floor, CARBCOMN voussoirs | ✓ | ✓ | ✓ |

The simpler examples are not subsets of the complex ones — they are independent starting points that demonstrate specific pipeline concepts without unnecessary complexity.
