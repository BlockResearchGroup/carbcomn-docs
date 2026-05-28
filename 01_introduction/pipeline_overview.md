# Pipeline Overview

The `carbcomn.core` pipeline transforms a set of high-level design parameters into a fully discretised, analysis-ready structural model of the CARBCOMN floor system. It is composed of three main computational stages — **form-finding**, **geometry generation**, and **discrete element analysis** — each of which produces data that is consumed by the next.

## Stages at a Glance

```mermaid
flowchart LR
    subgraph INIT["X00 — Init"]
        P["params.json"]
    end

    subgraph TNA["X10 — TNA Form-Finding"]
        direction TB
        FD["FloorFormDiagram\n+ Pattern"]
        RM["RefMesh"]
        FD --> RM
    end

    subgraph GEOM["X20 — Geometry"]
        direction TB
        TPL["FlatBarrelTemplate\n/ ArchTemplate"]
        BM["block_meshes\nblock_frames"]
        TPL --> BM
    end

    subgraph REFBLOCK["X30 — RefBlock"]
        direction TB
        RB["RefBlock"]
        EL["StandardBlock\nRidgeVoussoir\nCarbcomnVoussoir"]
        RB --> EL
    end

    subgraph DEM["X40–X42 — DEM Analysis"]
        direction TB
        MDL["BlockModel\n+ contacts"]
        SOL["Problem\n+ Solver"]
        RES["Results"]
        MDL --> SOL --> RES
    end

    subgraph FRAME["X50–X51 — Frame Assembly (optional)"]
        direction TB
        GRD["StructuralGrid\nColumn · TieBeam"]
        FM["FloorModel"]
        GRD --> FM
    end

    INIT --> TNA
    INIT --> GEOM
    TNA --> GEOM
    GEOM --> REFBLOCK
    REFBLOCK --> DEM
    DEM --> FRAME
```

## Data Flow

All intermediate data is persisted through the **`WorkflowSession`** system, which stores named Python objects as JSON files on disk. This means:

* Each pipeline stage is an independent script that reads from and writes to the session
* Any stage can be re-run in isolation without repeating previous stages
* Parameters are defined once (in the `init` script) and propagated automatically

```mermaid
flowchart TD
    S(["WorkflowSession\n(JSON on disk)"])

    X00["X00 init.py\n→ writes params"] --> S
    S --> X10["X10 tna.py\nreads params\n→ writes refmesh, formdiagram"]
    X10 --> S
    S --> X20["X20 geometry.py\nreads params, refmesh\n→ writes block_meshes, block_frames"]
    X20 --> S
    S --> X30["X30 refblock.py\nreads block_meshes, block_frames\n→ writes block_elements"]
    X30 --> S
    S --> X40["X40 dem_model.py\nreads block_elements\n→ writes blockmodel"]
    X40 --> S
    S --> X41["X41 dem_problem.py\nreads blockmodel\n→ writes blockmodel_results"]
    X41 --> S
```

## Script Numbering Convention

Workflow scripts follow a numbered prefix convention that makes the execution order explicit:

| Prefix | Stage                                                   |
| ------ | ------------------------------------------------------- |
| `X00`  | Init — define session and parameters                    |
| `X10`  | TNA — compute equilibrium form diagram                  |
| `X20`  | Geometry — generate block meshes and frames             |
| `X30`  | RefBlock — build typed structural elements              |
| `X40`  | DEM Model — assemble `BlockModel` and compute contacts  |
| `X41`  | DEM Problem — define and solve the mechanical problem   |
| `X42`  | DEM Visualisation — inspect results interactively       |
| `X50`  | Structural Grid — build column and beam layout          |
| `X51`  | Floor Assembly — integrate vault into full `FloorModel` |

Not every workflow uses every stage. The simpler examples (three-block test, arch) skip TNA and jump directly to geometry. See [Examples Overview](../04_examples/overview.md) for a comparison.

## Element Type Hierarchy

As the pipeline progresses from geometry to analysis, block meshes are progressively typed into structural element classes:

![Element type hierarchy — from raw mesh to typed structural elements](../.gitbook/assets/element_hierarchy.png)

The element type assigned to a given block controls its geometry for DEM analysis but does not change the pipeline structure — the `BlockModel` and solver are agnostic to element type.

> **See also:** [Pipeline Reference](../03_pipeline/overview.md) for a detailed description of each stage, and [Theory](../02_theory/structural_principles.md) for the structural background.
