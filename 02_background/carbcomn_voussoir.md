# The CARBCOMN Voussoir

## From standard masonry to CARBCOMN

A traditional voussoir is a wedge-shaped block whose geometry is entirely defined by its place in the vault — its faces are the joint surfaces with adjacent blocks, and its body is solid material. CARBCOMN introduces two **geometric modifications** that distinguish its voussoir from a standard masonry block:

1. A **ridge feature** along the intrados (underside) face
2. A **cable channel** passing through the block for post-tensioning

These features have structural and functional consequences that are reflected in the block element hierarchy of the pipeline.

<!-- [IMAGE PLACEHOLDER: Comparison of StandardBlock, RidgeVoussoir, and CarbcomnVoussoir geometries — isometric view] -->

## Block type hierarchy

The pipeline defines three voussoir types as subclasses of a common `StructuralElement` base class:

```
StructuralElement
    ├── StandardBlock
    │       └── A plain voussoir. Geometry equals the block mesh.
    │           Used for basic DEM testing and as the non-feature
    │           columns in ridge/CARBCOMN floor examples.
    │
    ├── RidgeVoussoir
    │       └── Adds a raised ridge of configurable width along
    │           the intrados face. The ridge is a geometric protrusion
    │           that interlocks with a corresponding slot in the floor
    │           finish. Assigned to selected columns via "feature_cols".
    │
    └── CarbcomnVoussoir
            └── The full CARBCOMN element. Extends RidgeVoussoir with:
                · An intrados slot (half-width: intrados_halfwidth)
                  creating a hollow channel for the post-tensioning cable
                · A cable channel (half-width: cable_halfwidth)
                  through the block body
                · A configurable intrados wall geometry (wall_resolution)
```

## The ridge feature

The ridge running along the intrados of both `RidgeVoussoir` and `CarbcomnVoussoir` serves two functions:

- **Structural interlocking** — the ridge engages with a corresponding groove in the adjacent voussoir column, providing a mechanical interlock that resists differential settlement and improves load distribution between columns
- **Floor finish integration** — the ridge profile can accommodate a floor finish panel that sits flush with the voussoir top surface

<!-- [IMAGE PLACEHOLDER: Detail of ridge feature on intrados face — section through joint between two voussoir columns] -->

## The cable channel

The post-tensioning cable channel is the defining feature of the `CarbcomnVoussoir`. It runs longitudinally through each voussoir in the feature columns, parallel to the span direction. When tensioned after assembly, the cables:

- Provide the horizontal tie force that replaces the abutments of a traditional vault
- Clamp the voussoirs together, improving frictional contact and increasing the effective compressive force at the joints
- Allow controlled prestress levels that can be tuned to the structural requirements of a given span and load

> **Note:** In the current version of the pipeline, the cable geometry is defined in the `CarbcomnVoussoir` class but **active cable prestress is not yet implemented as a DEM load case**. The block geometry is correct and contacts are computed normally; the prestress contribution will be added in a future pipeline version. See [Scope and Disclaimer](../01_introduction/scope_and_disclaimer.md).

<!-- [IMAGE PLACEHOLDER: CarbcomnVoussoir with cable channel highlighted — transparent view showing internal geometry] -->

## Element parameters

The geometry of each element type is controlled by the `params["elements"]` dictionary in the session. Key parameters:

| Parameter | Element | Description |
|-----------|---------|-------------|
| `feature_cols` | Ridge, CARBCOMN | List of column indices that receive the feature element type |
| `intrados_halfwidth` | CARBCOMN | Half-width of the intrados slot / cable housing |
| `cable_halfwidth` | CARBCOMN | Half-width of the cable channel within the slot |
| `wall_resolution` | CARBCOMN | Number of faces used to discretise the intrados wall curve |

All other block dimensions (height, width, joint angle) are inherited from the `RefBlock` geometry derived from the `RefMesh` and template.

## Structural role of each type in the CARBCOMN floor

In a complete CARBCOMN floor, the two block types serve complementary roles:

- **`StandardBlock` columns** — the majority of the vault. Plain geometry, no special structural behaviour beyond compression transfer. These are the gravity-carrying elements.
- **`CarbcomnVoussoir` columns** — the feature columns, spaced at regular intervals. They carry the post-tensioning cables and provide the intercolumn interlocking through the ridge. They define the structural redundancy and robustness of the system.

The ratio of feature to standard columns, their spacing, and the cable prestress level are design variables that control the overall structural performance. The pipeline enables systematic exploration of these variables through parameterised session scripts.

> **See also:** [Step 3 — RefBlock & Element Typing](../03_pipeline/refblock_step.md), [Example 500 — CARBCOMN Voussoirs](../04_examples/500_floor_stag_carbcomn.md)

> **References:** <!-- [PLACEHOLDER: Add CARBCOMN project publications and BRG references on the voussoir geometry] -->
