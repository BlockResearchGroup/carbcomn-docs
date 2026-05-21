# Step 6 — Structural Frame & Floor Assembly

**Script prefixes:** `X50` (Structural Grid), `X51` (Floor Assembly)  
**Session inputs:** `session["params"]`, `session["grid"]`, `session["buildingmodel"]`, `session["blockmodel"]`  
**Session outputs:** `session["buildingmodel"]` (updated), `session["grid"]`

## Purpose

The structural frame and floor assembly steps build the surrounding **column-and-beam frame** that supports the vault, and then integrate the vault voussoirs into a single unified `FloorModel` with a global contact graph spanning both the frame and the vault.

These steps are only present in full-floor workflows (examples `300`, `400`, `500`). The earlier examples (`000`–`200`) analyse the vault in isolation, without a surrounding frame.

## X50 — Structural Grid

The `StructuralGrid` defines the parametric layout of columns and tie-beams from the floor plan dimensions:

```python
from carbcomn.model import Column, FloorModel, StructuralGrid, TieBeam

grid = StructuralGrid(width=grid_width, depth=grid_depth, height=grid_height)
buildingmodel = FloorModel()

# Add columns at all four grid corners
for idx, column in enumerate(grid.columns):
    midpoint = column.midpoint
    T = Translation.from_vector(...)   # position below the floor slab
    col = Column(
        width=col_width, depth=col_depth,
        height=room_height,
        transformation=T,
        name=f"Column_{idx}",
        is_column=True,
    )
    buildingmodel.add_element(col)

# Add tie-beams along west and east edges
tiebeam_west = TieBeam(
    angle=15,           # angled cut at the beam–voussoir interface
    cut_right=False,
    cut_left=True,
    width=beam_width,
    height=floor_height,
    length=grid_depth,
    transformation=TX * T,
    name="tiebeam_west",
    is_beam=True,
)
buildingmodel.add_element(tiebeam_west)
buildingmodel.add_element(tiebeam_east)

buildingmodel.compute_contacts(tolerance=0.001)

session["buildingmodel"] = buildingmodel
session["grid"] = grid
session.sync()
```

The tie-beams span the full depth of the floor and provide the horizontal reaction to the vault thrust. Their `angle` parameter controls the geometry of the cut where the support voussoir meets the beam.

<!-- [IMAGE PLACEHOLDER: Structural grid viewer — columns (red), tie-beams (blue), grid frames] -->

## X51 — Full Floor Assembly

The floor assembly step merges the vault voussoirs from the `BlockModel` into the `FloorModel` and recomputes global contacts:

```python
from carbcomn.model import TrimModifier

# Translate vault blocks to their final position in the storey
T = Translation.from_vector([0, 0, grid_height - z_max])
for element in blockmodel.elements():
    if element.is_block:
        element.transformation = T
        buildingmodel.add_element(element)

# Apply TrimModifier at beam–voussoir interfaces
trim = TrimModifier()
for block in buildingmodel.find_all_elements_of_type(StructuralElement):
    if block.is_support:
        if block.name.endswith("_0"):
            buildingmodel.add_modifier(tiebeam_west, block, trim)
        else:
            buildingmodel.add_modifier(tiebeam_east, block, trim)

# Recompute contacts across the full model
buildingmodel.compute_contacts(tolerance=0.001, minimum_area=0.001)

session["buildingmodel"] = buildingmodel
session.sync()
```

### The TrimModifier

`TrimModifier` resolves **geometric intersections** between the angled cut of the tie-beam and the corresponding face of the support voussoir. Without it, the overlapping geometry would produce spurious contacts or missed contact interfaces. The modifier trims the voussoir face to match the beam face exactly before contact detection runs.

### Contact recomputation

After merging, `compute_contacts()` is called on the full `FloorModel`. This detects:
- All inter-voussoir contacts (as in the vault-only `BlockModel`)
- All voussoir-to-beam contacts at the support interfaces
- All beam-to-column contacts at the frame joints

The resulting contact graph is a unified topology of the entire floor system.

<!-- [IMAGE PLACEHOLDER: Full FloorModel — columns, tie-beams, vault voussoirs, all contacts in one view] -->

> **See also:** [Example 300](../04_examples/300_floor_stag_standard.md), [Example 400](../04_examples/400_floor_stag_ridge.md)
