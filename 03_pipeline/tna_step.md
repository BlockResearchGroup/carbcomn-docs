# Step 1 — TNA Form-Finding

**Script prefix:** `X10`  
**Session inputs:** `session["params"]`  
**Session outputs:** `session["formdiagram"]`, `session["refmesh"]`

## Purpose

The TNA step computes the **equilibrium geometry of the vault mid-surface**. Given the floor plan dimensions and a desired vault rise, it finds a three-dimensional surface through which the vault's internal compressive forces can flow in equilibrium under self-weight, without any bending or tension.

The result is a `RefMesh` — the reference surface that all downstream geometry generation is built upon.

## When this step is used

TNA is only needed for **vaulted floor geometries** where the shape is not analytically prescribed. The simpler examples (`000_threeblocks`, `100_arch`) define geometry directly and skip this step. All full-floor examples (`200`–`500`) use TNA.

## Key classes

| Class | Role |
|-------|------|
| `Pattern` | Defines the combinatorial layout (topology) of the vault force network |
| `FloorFormDiagram` | Wraps the form diagram; manages boundary conditions and equilibrium computation |
| `RefMesh` | Output surface storing equilibrium node positions and force densities |

## Code walkthrough

### 1. Define the pattern

```python
from carbcomn.datastructures import Pattern

pattern = Pattern.from_barrelvault(
    width=floor_width,       # floor plan width [m]
    depth=floor_depth,       # floor plan depth [m]
    number_of_radials=20,    # nodes along the span direction
    number_of_rings=30,      # nodes along the length direction
)
```

`Pattern.from_barrelvault()` creates a rectangular grid of nodes connected by two families of edges:
- **Rib edges** — run in the spanning direction (along the arch), carry the main compressive thrust
- **Ring edges** — run perpendicular to the span (along the vault length), carry transverse forces

### 2. Build the form diagram

```python
from carbcomn.datastructures import FloorFormDiagram

form = FloorFormDiagram.from_pattern(
    pattern=pattern,
    shell_thickness=bottom_shell_thickness,
    horizontal_layer_thickness=horizontal_layer_thickness,
    density=2.300 * block_fill_percentage,   # effective density [t/m³]
)
```

The form diagram inherits the topology of the pattern and adds:
- **Load distribution** — self-weight is computed from shell geometry and density and distributed to nodes
- **Boundary conditions** — perimeter nodes are fixed at the support elevation

### 3. Assign force densities

Force densities control the **shape** of the resulting equilibrium surface:

```python
# Ring edges carry no horizontal force (q = 0) → no in-plane curvature perpendicular to span
form.edges_attribute("hmin", 0, keys=list(form.edges_where(is_rib=False)))
form.edges_attribute("hmax", 0, keys=list(form.edges_where(is_rib=False)))

# Rib edges carry a horizontal force component of 2.0
form.edges_attribute("hmin", 2.0, keys=list(form.edges_where(is_rib=True)))
form.edges_attribute("hmax", 2.0, keys=list(form.edges_where(is_rib=True)))

# Boundary ribs carry a reduced force (softer at supports)
for edge in form.edges_where(is_rib=True):
    if pattern.is_edge_on_boundary(edge):
        form.edge_attribute(edge, "hmin", 1.0)
        form.edge_attribute(edge, "hmax", 1.0)
```

By setting ring edges to zero and rib edges to a uniform value, this configuration produces a **barrel vault** profile — the same curvature at every cross-section along the vault length.

### 4. Solve equilibrium and extract the RefMesh

```python
form.update_equilibrium(zmax=zmax)   # zmax constrains the vault rise
refmesh = form.to_refmesh()
```

`update_equilibrium()` solves the vertical equilibrium linear system described in the [TNA Theory section](../02_background/thrust_network_analysis.md) and stores the resulting node heights. `to_refmesh()` assembles the `RefMesh` from the equilibrium positions and force density values.

### 5. Export to session

```python
session["formdiagram"] = form
session["refmesh"] = refmesh
session.sync()
```

## What the RefMesh contains

After TNA, the `RefMesh` stores at each node and edge:

| Attribute | Location | Description |
|-----------|----------|-------------|
| `x, y, z` | vertex | 3D equilibrium position |
| `pz` | vertex | vertical load component at the node |
| `q` | edge | force density |
| `is_rib` | edge | `True` for spanning-direction edges |

These are used by the geometry template in the next step to orient voussoir cuts and define the thrust surface the blocks should follow.

<!-- [IMAGE PLACEHOLDER: RefMesh rendered in viewer — show thrust surface coloured by force density] -->

> **See also:** [TNA Theory](../02_background/thrust_network_analysis.md), [Example 200](../04_examples/200_floor_nostag_standard.md)
