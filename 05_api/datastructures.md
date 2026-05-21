# Datastructures

## RefMesh

The `RefMesh` is the primary output of the TNA stage. It is a mesh that stores the equilibrium geometry of the vault mid-surface and the force densities on its edges.

```python
from carbcomn.datastructures import RefMesh
```

### Key attributes (per vertex)

| Attribute | Type | Description |
|-----------|------|-------------|
| `x, y, z` | `float` | 3D equilibrium position |
| `pz` | `float` | Vertical load component at the node |

### Key attributes (per edge)

| Attribute | Type | Description |
|-----------|------|-------------|
| `q` | `float` | Force density |
| `is_rib` | `bool` | `True` for spanning-direction (arch-direction) edges |

### Key methods

| Method | Returns | Description |
|--------|---------|-------------|
| `edge_midpoint(edge)` | `Point` | Midpoint of an edge |
| `edge_length(edge)` | `float` | Length of an edge |
| `vertex_coordinates(v)` | `list[float]` | `[x, y, z]` of a vertex |
| `vertices()` | iterator | All vertex keys |
| `edges()` | iterator | All edge pairs `(u, v)` |

---

## RefBlock

`RefBlock` is an intermediate data structure produced by the geometry templates. It wraps a block mesh together with its reference frame and its position (column, row) in the vault grid.

```python
from carbcomn.datastructures.refblock import RefBlock
```

| Attribute | Type | Description |
|-----------|------|-------------|
| `mesh` | `Mesh` | The raw block mesh |
| `frame` | `Frame` | Local coordinate frame (origin at block centroid) |
| `col` | `int` | Column index in the vault grid |
| `row` | `int` | Row index in the vault grid |

`RefBlock` objects are consumed by `StandardBlock.from_refblock()`, `RidgeVoussoir.from_refblock()`, and `CarbcomnVoussoir.from_refblock()`.

---

## FloorFormDiagram

`FloorFormDiagram` wraps the 2D form diagram used in TNA. It manages boundary conditions, force density assignment, and equilibrium computation.

```python
from carbcomn.datastructures import FloorFormDiagram

form = FloorFormDiagram.from_pattern(
    pattern=pattern,
    shell_thickness=0.06,
    horizontal_layer_thickness=0.04,
    density=2.300 * block_fill_percentage,
)
```

### Key methods

| Method | Description |
|--------|-------------|
| `from_pattern(pattern, ...)` | Class method — construct from a `Pattern` |
| `update_equilibrium(zmax)` | Solve vertical equilibrium; writes node heights in place |
| `to_refmesh()` | Construct and return a `RefMesh` from the current equilibrium solution |
| `edges_attribute(attr, value, keys)` | Set an attribute on a list of edges (e.g. force density bounds) |
| `edges_where(is_rib=True)` | Iterator over edges matching a predicate |
| `vertex_residual(v)` | Equilibrium residual vector at a node (should be near zero after solve) |

---

## Pattern

`Pattern` defines the combinatorial topology (connectivity) of the TNA form diagram.

```python
from carbcomn.datastructures import Pattern

pattern = Pattern.from_barrelvault(
    width=6.0,
    depth=7.5,
    number_of_radials=20,
    number_of_rings=30,
)
```

| Method | Description |
|--------|-------------|
| `from_barrelvault(width, depth, n_radials, n_rings)` | Rectangular radial-ring grid for a barrel vault |
| `is_edge_on_boundary(edge)` | `True` if the edge lies on the plan perimeter |
