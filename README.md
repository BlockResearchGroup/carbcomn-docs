# CARBCOMN.core — Computational Pipeline Documentation

**ETH Zurich, Block Research Group**

---

> **Note:** The underlying source code is currently under active development in a private repository and will be released as open-source software at the conclusion of the project.

---

## What is CARBCOMN?

CARBCOMN is an EU Horizon Pathfinder Challenge project developing a novel floor system based on **carbon-fibre reinforced masonry voussoirs**. The system combines the compressive efficiency of traditional vaulted masonry construction with the tensile capacity of post-tensioned carbon-fibre cables, producing a flat floor system that is structurally efficient, materially lean, and fully demountable.

The project brings together partners from across Europe, combining expertise in structural engineering, material science, digital fabrication, and architecture.

## Role of ETH Zurich / Block Research Group

The Block Research Group (BRG) at ETH Zurich is responsible for the **structural design and computational pipeline** that underpins the CARBCOMN floor system. This includes:

- Developing the theoretical framework for the structural behaviour of the system
- Implementing the end-to-end computational pipeline for form-finding, discretisation, and structural analysis
- Providing the tools necessary for iterative design and optimisation across project phases

## What this documentation covers

This documentation describes **`carbcomn.core`**, the core Python library implementing the design and analysis pipeline. It is structured as follows:

| Section | Contents |
|---------|----------|
| [Theory](02_theory/structural_principles.md) | Structural principles behind compression-dominant floor systems, Thrust Network Analysis, Discrete Element Method |
| [Pipeline Reference](03_pipeline/overview.md) | Step-by-step description of the full TNA → Geometry → DEM pipeline |
| [Examples](04_examples/overview.md) | Six worked examples of increasing complexity, from a simple three-block test to the full CARBCOMN floor |
| [API Reference](05_api/overview.md) | Documentation of all classes and functions in `carbcomn.core` |
| [Solvers](06_solvers/overview.md) | Description of the available DEM solvers and guidance on choosing between them |
| [Installation](installation.md) | Environment setup and installation instructions |

## How to read this documentation

If you are new to the project, we recommend starting with the [Theory](02_theory/structural_principles.md) section to build familiarity with the structural concepts, then reading the [Pipeline Reference](03_pipeline/overview.md) before working through the [Examples](04_examples/overview.md) in order.

If you are already familiar with TNA and DEM analysis, you can go directly to the [Pipeline Reference](03_pipeline/overview.md) or to a specific example.

---

*For questions or issues related to this documentation, please contact the Block Research Group at ETH Zurich.*
