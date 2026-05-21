# Model Elements

## Element class hierarchy

```
StructuralElement          (base class — carbcomn.model.elements.structural)
    ├── Block
    │     ├── StandardBlock          (plain voussoir)
    │     ├── RidgeVoussoir          (voussoir + intrados ridge)
    │     └── CarbcomnVoussoir       (voussoir + ridge + cable slot)
    ├── Beam
    │     └── TieBeam
    ├── Column
    └── (virtual elements for contacts and interfaces)
```

---

## StructuralElement (base)

All structural elements share the following interface:

| Attribute / Method | Type | Description |
|--------------------|------|-------------|
| `name` | `str` | Unique name assigned at construction |
| `is_support` | `bool` | If `True`, block is treated as kinematically fixed by the solver |
| `is_block` | `bool` | `True` for voussoir-type elements |
| `is_beam` | `bool` | `True` for tie-beam elements |
| `is_column` | `bool` | `True` for column elements |
| `point` | `Point` | Centroid of the element geometry |
| `modelgeometry` | `Mesh` | The mesh used for DEM contact detection |
| `transformation` | `Transformation` | Applied transformation (translation, rotation) |
| `compute_elementgeometry()` | `Mesh` | Compute and return the typed element mesh |

---

## StandardBlock

```python
from carbcomn.model.elements.blocks.standard_block import StandardBlock

element = StandardBlock.from_refblock(refblock=rb, is_support=False, name="block_0_0")
```

Plain voussoir. Its `modelgeometry` equals the raw block mesh from the geometry template.

---

## RidgeVoussoir

```python
from carbcomn.model.elements.blocks.ridge_voussoir import RidgeVoussoir

element = RidgeVoussoir.from_refblock(refblock=rb, is_support=False, name="block_2_0")
```

Extends `StandardBlock` with a raised ridge along the intrados face. The ridge geometry is derived from the `RefBlock` dimensions.

---

## CarbcomnVoussoir

```python
from carbcomn.model.elements.blocks.carbcomn_voussoir import CarbcomnVoussoir

element = CarbcomnVoussoir.from_refblock(refblock=rb, is_support=False, name="block_2_0")
```

The full CARBCOMN element. Extends `RidgeVoussoir` with an intrados slot and post-tensioning cable channel.

Key additional parameters (set in `params["elements"]`):

| Parameter | Description |
|-----------|-------------|
| `intrados_halfwidth` | Half-width of the intrados slot [m] |
| `cable_halfwidth` | Half-width of the cable channel [m] |
| `wall_resolution` | Mesh resolution of the intrados wall curve |

---

## TieBeam

```python
from carbcomn.model import TieBeam

tiebeam = TieBeam(
    angle=15,         # cut angle at beam–voussoir interface [degrees]
    cut_right=False,
    cut_left=True,
    width=0.3,
    height=0.5,       # full floor height
    length=7.5,
    transformation=TX * T,
    name="tiebeam_west",
    is_beam=True,
)
```

---

## Column

```python
from carbcomn.model import Column

col = Column(width=0.2, depth=0.2, height=room_height, transformation=T, name="Column_0", is_column=True)
```

---

## FloorModel

`FloorModel` is a higher-level model container that holds both structural frame elements (columns, beams) and vault voussoirs in a single unified contact graph.

```python
from carbcomn.model import FloorModel

model = FloorModel()
model.add_element(column)
model.add_element(tiebeam)
model.add_modifier(tiebeam, support_voussoir, TrimModifier())
model.compute_contacts(tolerance=0.001, minimum_area=0.001)
```

| Method | Description |
|--------|-------------|
| `add_element(element)` | Add a structural element to the model |
| `remove_element(element)` | Remove an element |
| `find_element_with_name(name)` | Look up an element by name |
| `find_all_elements_of_type(type)` | Iterator over all elements of a given type |
| `add_modifier(host, target, modifier)` | Register a geometry modifier between two elements |
| `compute_contacts(tolerance, minimum_area)` | Detect and discretise all contact interfaces |
| `elements()` | Iterator over all elements |
| `contacts()` | Iterator over all contact objects |

---

## StructuralGrid

`StructuralGrid` generates the parametric layout of columns and beams from floor plan dimensions.

```python
from carbcomn.model import StructuralGrid

grid = StructuralGrid(width=6.0, depth=7.5, height=3.2)
```

| Attribute | Description |
|-----------|-------------|
| `columns` | List of column centreline lines |
| `beams` | List of beam centreline lines |
| `column_frames` | Coordinate frames at column positions |
| `beam_frame_west` | Frame for the west tie-beam |
| `beam_frame_east` | Frame for the east tie-beam |
| `all_points` | All grid intersection points |

---

## TrimModifier

```python
from carbcomn.model import TrimModifier

trim = TrimModifier()
model.add_modifier(tiebeam_west, support_block, trim)
```

Resolves geometric intersections between two elements before contact detection. Applied at the tie-beam / support-voussoir interface to ensure clean, non-overlapping contact polygons.
