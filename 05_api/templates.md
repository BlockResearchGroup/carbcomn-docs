# Templates

Templates are factory classes that take a `RefMesh` (or analytic geometry) and discretisation parameters, and produce the `block_meshes` and `block_frames` arrays consumed by the geometry step.

## FlatBarrelTemplate

The primary template for CARBCOMN floor generation. Produces a rectangular grid of voussoirs fitted to a barrel vault `RefMesh`, with flat (planar) block faces.

```python
from carbcomn.templates.barrel import FlatBarrelTemplate

template = FlatBarrelTemplate.from_spacing(
    refmesh=refmesh,
    voussoirs_count_span=20,     # blocks in the arch direction
    voussoirs_count_length=7,    # blocks in the vault length direction
    thickness=0.3,               # voussoir thickness [m]
    stagger_type="running",      # "none" | "running"
    stagger_amount=0.2,          # column offset fraction (0–1)
)

block_meshes = template.blocks           # list[list[Mesh]]  — [col][row]
block_frames = template.block_frames     # list[list[Frame]] — [col][row]
z_max = template.flat_top_z             # highest extrados z
z_min = template.bot_min_z             # lowest intrados z
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `refmesh` | `RefMesh` | Equilibrium surface from TNA |
| `voussoirs_count_span` | `int` | Number of blocks along the span (arch) direction |
| `voussoirs_count_length` | `int` | Number of blocks along the vault length |
| `thickness` | `float` | Voussoir thickness [m] |
| `stagger_type` | `str` | `"none"` or `"running"` |
| `stagger_amount` | `float` | Column offset as a fraction of block length (used with `"running"`) |

---

## ArchTemplate

Generates voussoir geometry for a semicircular or segmental arch. Does not require a `RefMesh`.

```python
from carbcomn.templates.arch import ArchTemplate

template = ArchTemplate(
    span=3.0,
    rise=1.5,
    depth=0.5,
    thickness=0.3,
    n_blocks=10,
)

block_meshes = template.blocks
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `span` | `float` | Total arch span [m] |
| `rise` | `float` | Arch rise from springing to crown [m] |
| `depth` | `float` | Arch depth (out-of-plane) [m] |
| `thickness` | `float` | Voussoir thickness [m] |
| `n_blocks` | `int` | Number of voussoirs (excluding supports) |

---

## BarrelTemplate / VariableBarrelTemplate

<!-- [PLACEHOLDER: Document additional template variants as they are developed] -->

Additional barrel vault template variants are available for non-flat block face geometries. Their API mirrors `FlatBarrelTemplate`. See source in `src/carbcomn/templates/barrel.py` for current status.
