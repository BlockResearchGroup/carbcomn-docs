# Project Context

## CARBCOMN — Carbon-Negative Compression-Dominant Structures

**CARBCOMN** (*CARBon-negative COMpression domNant structures for decarbonized and deconstructable concrete buildings*) is funded under the **EU Horizon Europe EIC Pathfinder Challenge** programme (Grant No. HORIZON-EIC-101161535), which supports high-risk, high-reward research at the frontier of emerging technologies and industrial application. The project directly targets the AEC sector challenge *"Digitalisation for a Novel Triad of Design, Fabrication, and Materials"*.

The project addresses the construction industry's carbon footprint through a disruptive combination of three interlocking technologies: **carbon-negative concrete** produced via CO₂ mineralisation curing, **compression-dominant unreinforced masonry** geometry derived from funicular form-finding, and **iron-based shape-memory alloy (Fe-SMA) post-tensioning** for system activation and robustness. Together, these enable a flat floor system assembled from discrete, dry-joint blocks that are structurally efficient, materially lean, and fully demountable for reuse.

The structural design paradigm follows the principle of *strength through geometry*: internal forces are directed along compression-only trajectories, making steel reinforcement in the concrete unnecessary and allowing the use of carbon-negative, carbonation-cured concrete mixtures that are incompatible with conventional rebar.

<!-- [IMAGE PLACEHOLDER: CARBCOMN project overview diagram / render of complete floor system] -->

## Partners and Responsibilities

The CARBCOMN consortium brings together **5 research institutions** and **6 industry partners** across 7 European countries:

| Partner | Country | Role |
|---------|---------|------|
| Ghent University (UGent) | Belgium | Project coordinator; carbon-negative concrete, LCA |
| ETH Zürich — Block Research Group (BRG) | Switzerland | Structural design, computational pipeline (WP2) |
| EMPA | Switzerland | Material characterisation, durability |
| TU Darmstadt | Germany | Structural systems, topology optimisation |
| University of Patras (UPAT) | Greece | Fe-SMA post-tensioning, structural robustness |
| TESIS | Spain | Digital fabrication, 3D printing |
| Orbix | Belgium | Carbon mineralisation, slag-based concrete |
| incremental3d (In3D) | — | Extrusion-based 3D printing of concrete |
| Mario Cucinella Architects (MCA) | Italy | Architectural design integration |
| re-fer | Switzerland | Fe-SMA product development |
| Zaha Hadid Architects (ZHA) | UK | Computational design, discretisation |

The **Block Research Group (BRG) at ETH Zurich** leads the structural design and computational work package (Work Package 2), with responsibility for:

- Formalising the structural design principles of the CARBCOMN floor system
- Developing the `carbcomn.core` computational pipeline for form-finding, discretisation, and DEM analysis
- Delivering the software tools that allow design iteration and structural verification across all project phases

## CARBCOMN.core in the project ecosystem

The CARBCOMN project involves academic and industry partners developing an integrated pipeline across several disciplines. Computationally, this pipeline is organised into distinct but interoperable workflows, all of which share a common data model:

| Workflow | Description |
|----------|-------------|
| **Structural Design** | Form-finding, block discretisation, and DEM structural analysis — the subject of this deliverable |
| **Architectural Design** | Architectural expression and integration of the CARBCOMN system into building contexts |
| **LCA Analysis** | Life cycle quantification and optimisation of the carbon footprint of components |
| **Design-to-Fabrication** | Print-path generation and robotic control for 3D-printed block production |
| **Material Design** | Concrete mixture data, carbonation properties, and material performance records |

The `carbcomn.core` package provides the centralised data model that underpins all of these workflows, allowing data to be shared, augmented, and extended across partners without loss of fidelity.

The `carbcomn.core` code repository is hosted on GitHub at **[https://github.com/BlockResearchGroup/CARBCOMN.core](https://github.com/BlockResearchGroup/CARBCOMN.core)**. Access can be provided to CARBCOMN consortium members — contact dellendice@arch.ethz.ch, maiaavelino@arch.ethz.ch, or van.mele@arch.ethz.ch with your GitHub username.

## Deliverable D2.1

This documentation constitutes **Deliverable D2.1** (*CARBCOMN Model and Structural Design Workflow*) of the CARBCOMN project. It provides:

1. A full description of the computational pipeline implemented in `carbcomn.core`
2. A theoretical foundation for the structural methods employed
3. Documented worked examples demonstrating the pipeline on reference cases
4. Guidance on solver options for Discrete Element Method (DEM) analysis

D2.1 represents the current state of the pipeline as of the first reporting period. The pipeline is under active development and further deliverables will document subsequent refinements.

## Broader project context

CARBCOMN targets a major conceptual shift in how concrete load-bearing structures are designed and built. Current practice relies on moment-resisting reinforced concrete frames, in which the concrete must crack to work properly and a significant mass of steel reinforcement is required. The CARBCOMN approach acts on three levels simultaneously:

- **Material level** — replacing Portland cement with carbonation-cured secondary-material mixtures (e.g. stainless-steel slag powders) that sequester CO₂ as a hardening mechanism, achieving carbon-negative or near-carbon-neutral concrete
- **Fabrication level** — using extrusion-based 3D printing to produce discrete blocks with geometries optimised for structural efficiency, high surface-to-volume ratio (improving carbonation), and precise stereotomy suited to dry assembly
- **Design level** — applying compression-dominant structural principles derived from historical unreinforced masonry, combined with topology optimisation, to achieve 45–70% material savings compared to conventional reinforced concrete designs

The Fe-SMA post-tensioning system provides system-level prestress and robustness without embedded reinforcement: bars are tensioned by resistive heating ("self-prestressing"), and their high ductility (failure strain > 30%) contributes to structural robustness under horizontal actions including seismic loading.

The system is designed from the outset for **circularity**: dry-joint blocks without bonded reinforcement can be disassembled, inspected, and reused, with a target service life enabling at least two reuse cycles (≥ 150 years total)[^1].

## Current Status and Open-Source Release

The `carbcomn.core` repository is currently **private** and under active development. It will be released as open-source software at the conclusion of the project. This documentation is provided to the consortium and project evaluators as a deliverable in advance of the public release.

> **See also:** [Scope and Disclaimer](scope_and_disclaimer.md) for a discussion of the pipeline's role in the broader research context.

[^1]: Van Tittelboom, K., & Matthys, S. (Eds.) (2025). *Green Paper on Compression Dominant Carbon Curing Concrete Structures*. CARBCOMN Consortium. https://doi.org/10.5281/zenodo.17277986
