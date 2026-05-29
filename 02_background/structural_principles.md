# Structural System

## Compression-dominant structural systems

Traditional masonry structures — arches, vaults, domes, flying buttresses — owe their longevity to a single fundamental property: **stone and brick are strong in compression but essentially incapable of carrying tension**. Because mortar joints in historical masonry are also very weak in tension, these structures survive only if their internal forces remain compressive everywhere. This constraint, far from being a limitation, is what gives masonry structures their characteristic geometry: the curved profiles of Gothic vaults, the pointed arches of Roman aqueducts, and the catenary form of suspension bridges are all expressions of the same underlying principle — the **funicular form**.

A funicular form for a given load distribution is one in which the structure carries loads purely through axial force, with no bending. For compression (pushing) forces this is the _inverted_ catenary — a shape that hangs in pure tension under gravity also stands in pure compression when flipped. The structural integrity of masonry relies on the internal thrust remaining within the material: as long as the line of thrust passes through the cross-section of every joint, no tension develops and the structure is in equilibrium.

## From 1D arches to 2D vaults

An arch is a one-dimensional funicular structure: it resolves loads along a single curved path to two supports. A **vault** extends this principle into two dimensions — loads are resolved along a network of curved ribs and webs, channelled to a set of perimeter supports. The structural geometry of a vault is no longer a single curve but a **surface**, and the internal forces are no longer a single thrust but a two-dimensional network of compressive resultants.

For a given load distribution, there is not a single correct vault geometry but a **family of equilibrium surfaces** — a thrust surface can be shifted up and down within the cross-section of the vault as long as it remains compressive. The designer's task is to find a geometry in which this thrust surface is well-centred, the forces are well-distributed, and the structure uses material efficiently.

The **Thrust Network Analysis (TNA)** method<sup>[4]</sup>, described in detail in the [next section](thrust_network_analysis.md), provides the computational framework for solving this two-dimensional form-finding problem. TNA is the first computational stage of the CARBCOMN pipeline.

## Unreinforced concrete masonry as a contemporary paradigm

The structural principles of historic masonry are directly applicable to modern digital concrete construction. Concrete, viewed as an artificial stone, shares the defining material property of masonry: it is strong in compression and weak in tension. This means that shaping concrete as a discretised compression-dominant assembly — rather than as a bending-dominant reinforced beam — is not only structurally appropriate but enables the elimination of steel reinforcement entirely.

This insight underpins a growing body of work on **Unreinforced Concrete Masonry (URCM)** as a pathway to circular concrete construction. By applying the design and construction paradigm of unreinforced masonry — funicular geometries, discrete dry-jointed blocks, no bonded reinforcement — it becomes possible to produce concrete structures that are fully demountable, reusable, and recyclable. The environmental benefits are compounded: less material is used through geometry-based efficiency, no steel is embedded (avoiding the main cause of durability-driven demolition), and end-of-life deconstruction and reuse replace crushing and recycling.<sup>[1]</sup>

The **fabrication advantages are equally significant**. Extrusion-based 3D concrete printing produces material that is orthotropic: it is stronger perpendicular to the printed layers than parallel to them, and the inter-layer bonds are a potential weakness under tension or shear. However, within a compression-dominant structural system, forces flow perpendicular to the printed layers — i.e. into compression — which precisely aligns with the printing direction's strongest axis. The structural logic of unreinforced masonry is thus **intrinsically compatible** with the mechanical properties of 3D-printed concrete.<sup>[2]</sup>

### Demonstrations: Striatus and Striatus 2.0 Phoenix

![Striatus (Venice, 2021) and Striatus 2.0 Phoenix (2024) — full-scale pedestrian bridges built from 3D-printed, unreinforced, dry-jointed concrete blocks by Block Research Group and Zaha Hadid Architects](../.gitbook/assets/striatusphoenix.png)

These principles have been demonstrated at full structural scale in two pedestrian bridge projects by the Block Research Group at ETH Zurich and Zaha Hadid Architects:

**Striatus** (Venice, 2021) is a 16 m-span arch bridge built entirely from 3D-concrete-printed, unreinforced blocks assembled without mechanical connections or adhesives — dry-jointed and held in place solely by geometry and gravity. The blocks were printed with layer orientations aligned to the principal compression trajectories, ensuring that forces flow perpendicular to the printed layers throughout the structure. The design demonstrates that horizontally-spanning structural elements, including pedestrian bridges, can be realised from 3D-printed concrete without reinforcement or bending resistance, provided the geometry is properly funicular.<sup>[2]</sup>

**Striatus 2.0: Phoenix** (2024) is a permanent successor to Striatus, improving on the original design in several aspects related to circularity, manufacturing precision, and assembly. The project documents the improved integrated design, engineering, and fabrication framework and provides a detailed comparison between the two iterations, further demonstrating the maturity of the URCM approach for real structural applications.<sup>[3]</sup>

Both projects exemplify the *strength through geometry* motto that underlies the CARBCOMN structural design philosophy: by aligning structural form with material behaviour, efficient structures can be built with less material, no embedded reinforcement, and full reversibility at end of life.

## The CARBCOMN floor system

![The CARBCOMN floor system: vaulted assembly of dry-jointed voussoir blocks post-tensioned with Fe-SMA bars, delivering a flat usable floor surface through compression-dominant geometry](../.gitbook/assets/structuralsystem_w.png)

The CARBCOMN floor system brings this structural logic into the context of a floor plate. It consists of a **vaulted floor composed of prefabricated voussoir blocks** assembled dry and post-tensioned with Fe-SMA (iron-based shape-memory alloy) bars. Three features distinguish it from conventional masonry:

1. **Flat floor profile** — the vault is shallow enough to deliver a usable, flat floor surface while retaining the structural efficiency of the curved compression shell beneath
2. **Fe-SMA post-tensioning** — iron-based shape-memory alloy bars running through channels in the voussoirs provide a controlled horizontal tensile tie, activated by resistive heating ("self-prestressing"); this stabilises the system against spreading and enables deployment without deep abutments
3. **Demountability** — the blocks are held in place by geometry and post-tension; they can be disassembled, reused, or replaced at end of life

The structural behaviour of this system is fundamentally that of a **compression shell with a tensile tie ring**: the vault carries loads in compression along trajectories determined by TNA, while the cables provide the horizontal reaction that would otherwise need to be carried by massive abutments or a stiff perimeter frame.

## Discrete assemblies and block mechanics

Because the floor is made of **discrete blocks** rather than a continuous material, its structural behaviour differs from that of a monolithic shell. The critical structural events are concentrated at the **contacts between blocks** — the joints. Each contact can transmit compressive normal forces and frictional tangential forces, but cannot transmit tension.

The structural analysis of discrete masonry assemblies is therefore a problem in **contact mechanics**: given the geometry of the blocks and the applied loads, do equilibrium forces exist at the contacts that satisfy the no-tension, friction-limited conditions? And if so, what are they?

This is the domain of the **Discrete Element Method (DEM)**, described in the [DEM section](discrete_element_modelling.md). DEM is the third and final computational stage of the CARBCOMN pipeline, and it operates on the block geometry produced by TNA form-finding and the subsequent discretisation step.

## Key concepts summary

| Concept            | Description                                                                                                 |
| ------------------ | ----------------------------------------------------------------------------------------------------------- |
| Funicular form     | A structural geometry that carries a given load in pure compression (or tension), with no bending           |
| Thrust surface     | The 2D generalisation of the thrust line — the surface through which compressive resultants flow in a vault |
| Force density      | The ratio of axial force to edge length in a network; the key variable in TNA optimisation                  |
| Contact constraint | The condition that contact forces at block interfaces must be compressive and within the friction cone      |
| Voussoir           | A wedge-shaped block forming part of an arch or vault; the elementary unit of the CARBCOMN floor            |

> **See also:** [Thrust Network Analysis](thrust_network_analysis.md) · [Discrete Element Modelling](discrete_element_modelling.md)

***

## References

1. Bhooshan, S., Dell'Endice, A., Ranaudo, F., Van Mele, T., & Block, P. (2024). Unreinforced concrete masonry for circular construction. *Architectural Intelligence*, 3, 7. https://doi.org/10.1007/s44223-023-00043-y
2. Bhooshan, S., Dell'Endice, A., Bhooshan, V., Van Mele, T., & Block, P. (2022). The Striatus bridge: Computational design and robotic fabrication of an unreinforced, 3D-concrete-printed, masonry arch bridge. *Architecture, Structures and Construction*, 2(4), 521–543. https://doi.org/10.1007/s44150-022-00051-y — Dell'Endice, A., Bouten, S., Van Mele, T., & Block, P. (2023). Structural design and engineering of Striatus, an unreinforced 3D-concrete-printed masonry arch bridge. *Engineering Structures*, 292, 116534. https://doi.org/10.1016/j.engstruct.2023.116534
3. Dell'Endice, A., Bodea, S., Van Mele, T., Block, P., Bhooshan, V., Bhooshan, S., Eiz, H., Chen, T., Lombois-Burger, H., de la Mothe, L. R., Nana, S., Megens, J., Sanin, S., & Bürgin, T. (2024). STRIATUS 2.0 PHOENIX — Improving circularity of 3D-concrete-printed unreinforced masonry structures. In P. Ayres, M. Ramsgaard Thomsen, B. Sheil, & M. Skavara (Eds.), *Fabricate 2024: Creating Resourceful Futures*. UCL Press. https://doi.org/10.2307/jj.11374766.15
4. Block, P. (2009). *Thrust Network Analysis: Exploring Three-Dimensional Equilibrium*. PhD thesis, Massachusetts Institute of Technology. https://dspace.mit.edu/handle/1721.1/49539
