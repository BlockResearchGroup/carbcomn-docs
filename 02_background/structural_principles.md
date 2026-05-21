# Structural Principles

## Compression-dominant structural systems

Traditional masonry structures — arches, vaults, domes, flying buttresses — owe their longevity to a single fundamental property: **stone and brick are strong in compression but essentially incapable of carrying tension**. Because mortar joints in historical masonry are also very weak in tension, these structures survive only if their internal forces remain compressive everywhere. This constraint, far from being a limitation, is what gives masonry structures their characteristic geometry: the curved profiles of Gothic vaults, the pointed arches of Roman aqueducts, and the catenary form of suspension bridges are all expressions of the same underlying principle — the **funicular form**.

A funicular form for a given load distribution is one in which the structure carries loads purely through axial force, with no bending. For compression (pushing) forces this is the *inverted* catenary — a shape that hangs in pure tension under gravity also stands in pure compression when flipped. The structural integrity of masonry relies on the internal thrust remaining within the material: as long as the line of thrust passes through the cross-section of every joint, no tension develops and the structure is in equilibrium.

<!-- [IMAGE PLACEHOLDER: Illustration of thrust line in an arch — safe / unsafe configurations] -->

## From 1D arches to 2D vaults

An arch is a one-dimensional funicular structure: it resolves loads along a single curved path to two supports. A **vault** extends this principle into two dimensions — loads are resolved along a network of curved ribs and webs, channelled to a set of perimeter supports. The structural geometry of a vault is no longer a single curve but a **surface**, and the internal forces are no longer a single thrust but a two-dimensional network of compressive resultants.

For a given load distribution, there is not a single correct vault geometry but a **family of equilibrium surfaces** — a thrust surface can be shifted up and down within the cross-section of the vault as long as it remains compressive. The designer's task is to find a geometry in which this thrust surface is well-centred, the forces are well-distributed, and the structure uses material efficiently.

The **Thrust Network Analysis (TNA)** method, described in detail in the [next section](thrust_network_analysis.md), provides the computational framework for solving this two-dimensional form-finding problem. TNA is the first computational stage of the CARBCOMN pipeline.

## The CARBCOMN floor system

The CARBCOMN floor system brings this structural logic into the context of contemporary construction. It consists of a **vaulted floor composed of prefabricated voussoir blocks** post-tensioned with carbon-fibre cables. Three features distinguish it from conventional masonry:

1. **Flat floor profile** — the vault is shallow enough to deliver a usable, flat floor surface while retaining the structural efficiency of the curved compression shell beneath
2. **Carbon-fibre post-tensioning** — cables running through channels in the voussoirs provide a controlled horizontal tensile tie, stabilising the system against spreading and enabling the use of the system without deep abutments
3. **Demountability** — the blocks are held in place by geometry and post-tension; they can be disassembled, reused, or replaced at end of life

<!-- [IMAGE PLACEHOLDER: Cross-section of CARBCOMN floor system showing vault geometry, voussoir blocks, and cable path] -->

The structural behaviour of this system is fundamentally that of a **compression shell with a tensile tie ring**: the vault carries loads in compression along trajectories determined by TNA, while the cables provide the horizontal reaction that would otherwise need to be carried by massive abutments or a stiff perimeter frame.

## Discrete assemblies and block mechanics

Because the floor is made of **discrete blocks** rather than a continuous material, its structural behaviour differs from that of a monolithic shell. The critical structural events are concentrated at the **contacts between blocks** — the joints. Each contact can transmit compressive normal forces and frictional tangential forces, but cannot transmit tension.

The structural analysis of discrete masonry assemblies is therefore a problem in **contact mechanics**: given the geometry of the blocks and the applied loads, do equilibrium forces exist at the contacts that satisfy the no-tension, friction-limited conditions? And if so, what are they?

This is the domain of the **Discrete Element Method (DEM)**, described in the [DEM section](discrete_element_method.md). DEM is the third and final computational stage of the CARBCOMN pipeline, and it operates on the block geometry produced by TNA form-finding and the subsequent discretisation step.

## Key concepts summary

| Concept | Description |
|---------|-------------|
| Funicular form | A structural geometry that carries a given load in pure compression (or tension), with no bending |
| Thrust surface | The 2D generalisation of the thrust line — the surface through which compressive resultants flow in a vault |
| Force density | The ratio of axial force to edge length in a network; the key variable in TNA optimisation |
| Contact constraint | The condition that contact forces at block interfaces must be compressive and within the friction cone |
| Voussoir | A wedge-shaped block forming part of an arch or vault; the elementary unit of the CARBCOMN floor |

> **References:** <!-- [PLACEHOLDER: Add key references — Block & Ochsendorf 2007, Heyman 1966, etc.] -->
