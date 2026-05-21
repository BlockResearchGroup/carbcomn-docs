# Step 2 — Block Geometry Generation

**Script prefix:** `X20`  
**Session inputs:** `session["params"]`, `session["refmesh"]` (if TNA was run)  
**Session outputs:** `session["block_meshes"]`, `session["block_frames"]`

## Purpose

The geometry step **discretises the vault surface into a grid of voussoir block meshes**. It takes the equilibrium surface from the TNA step (or an analytically defined geometry for simpler cases) and produces a list of `compas.geometry.Mesh` objects — one per block — along with their reference frames.

These meshes are the physical geometry of the voussoirs: their faces are the contact surfaces that will be detected and analysed in the DEM step.

## Template classes

The geometry generation is handled by **template classes** that parameterise the block layout and cutting strategy. Each template takes a `RefMesh` (or equivalent analytic geometry) and a set of block count / thickness / stagger parameters and returns the discretised block grid.

| Template | Used in | Description |
|----------|---------|-------------|
| `FlatBarrelTemplate` | Examples 200–500 | Rectangular grid of voussoirs fitted to a barrel vault `RefMesh`; block faces are flat (planar cuts) |
| `ArchTemplate` | Example 100 | Parametric semicircular arch voussoirs; no `RefMesh` needed |

Additional template variants are available for different vault profiles (see [Templates API](../05_api/templates.md)).

## Code walkthrough — FlatBarrelTemplate

```python
from carbcomn.datastructures import RefMesh
from carbcomn.templates.barrel import FlatBarrelTemplate

refmesh: RefMesh = session["refmesh"]

template = FlatBarrelTemplate.from_spacing(
    refmesh=refmesh,
    voussoirs_count_span=20,    # number of blocks along the span direction
    voussoirs_count_length=7,   # number of blocks along the vault length
    thickness=0.3,              # voussoir thickness [m]
    stagger_type="running",     # "none" | "running"
    stagger_amount=0.2,         # fractional column offset (0–1)
)

block_meshes = template.blocks         # list[list[Mesh]] — [col][row]
block_frames = template.block_frames   # list[list[Frame]]
```

The output is a **two-dimensional list** indexed by `[column][row]`:
- `block_meshes[i][j]` — the mesh of the block in column `i`, row `j`
- `block_frames[i][j]` — the reference frame of that block (origin at centroid, axes aligned with the block's local coordinate system)

## Stagger patterns

The `stagger_type` parameter controls how adjacent vault columns are offset relative to each other:

| `stagger_type` | Description | Used in |
|----------------|-------------|---------|
| `"none"` | No offset — all block joints are aligned across columns | Example 200 |
| `"running"` | Each column is offset by `stagger_amount` × block length | Examples 300–500 |

Running bond stagger (analogous to brick bonding) improves the structural interlocking between adjacent vault columns and is the intended configuration for the CARBCOMN floor system.

<!-- [IMAGE PLACEHOLDER: Top-view comparison of no-stagger vs running-stagger block layouts] -->

## Floor height extraction

After generating the block grid, the template computes the overall floor height from the geometry:

```python
z_max = template.flat_top_z      # highest point of the block extrados
z_min = template.bot_min_z       # lowest point of the block intrados

session["params"]["floor"]["z_max"] = z_max
session["params"]["floor"]["z_min"] = z_min
session["params"]["floor"]["final_height"] = z_max - z_min
```

These values are written back into `params` so they are available to downstream scripts (e.g. for frame sizing in the structural grid step).

## Export to session

```python
session["block_meshes"] = block_meshes
session["block_frames"] = block_frames
session.sync()
```

> **See also:** [Step 3 — RefBlock & Element Typing](refblock_step.md), [Templates API](../05_api/templates.md), [Example 200](../04_examples/200_floor_nostag_standard.md)
