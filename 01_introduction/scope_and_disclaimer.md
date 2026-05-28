# Scope

## Usage of the pipeline and role in CARBCOMN

The `carbcomn.core` pipeline is a **design and analysis tool**, providing tools for structurally informed design optimisation and  verification. Its purpose is to enable researchers and engineers to:

- Rapidly generate geometrically valid, structurally informed discretisations of the CARBCOMN floor system
- Subject those discretisations to equilibrium analysis under gravity and imposed loads using state-of-the-art Discrete Element Method (DEM) solvers
- Inspect and compare results across different geometries, block types, and solver configurations

**The pipeline is a starting point for structural research and exploration.**

Outputs from the pipeline — equilibrium geometries, contact force distributions, reaction vectors — are intended to seed further investigation: parameter studies, optimisation loops, sensitivity analyses, comparison against physical prototypes, and detailed finite element follow-up.

## Current development status

> ⚠️ The `carbcomn.core` repository is under **very active development** and is subject to frequent changes. APIs, data formats, and workflow conventions may change between versions.

The examples documented here reflect the state of the pipeline as of the first reporting period of the CARBCOMN project. As the structural research advances and new element types and analysis capabilities are added, this documentation will be updated accordingly.

## What is not yet implemented

The following capabilities are planned but not yet present in the current version of the pipeline:

- **Automated optimisation** — parameter sweeps and geometry optimisation are performed manually by re-running scripts with modified parameters
- **Post-tensioning analysis** — the cable geometry is defined in `CarbcomnVoussoir` but active cable prestress is not yet implemented as a DEM load case
- **Non-linear material response** — all current analyses assume rigid blocks with Mohr-Coulomb contact; deformable block analysis is planned via LMGC90
- **Fabrication export** — CAD/CAM export of fabrication-ready voussoir geometry is planned as part of a later work package

## Relationship to other CARBCOMN work packages

The `carbcomn.core` pipeline sits at the intersection of several project work packages:

- It consumes **material and geometric constraints** from WP1 (material development and testing)
- It produces **structural models and analysis results** that feed WP3 (physical prototyping and experimental validation)
- Its outputs inform **architectural and fabrication decisions** in WP4

The pipeline should be understood as the computational backbone of the structural design loop, not as an isolated software product.
