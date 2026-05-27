# Solver Comparison

This page summarises the practical trade-offs between the available solvers for typical CARBCOMN analysis tasks. For full per-solver descriptions, see the individual solver pages.

## Comparison table

| Property | LMGC90 | CRA | RBE | 3DEC |
| --- | :---: | :---: | :---: | :---: |
| License | Open-source (LMGC, Montpellier) | Open-source (BRG) | Open-source (BRG) | Commercial (Itasca) |
| Formulation | NSCD (implicit) | Coupled equilibrium-kinematics | Direct equilibrium | DEM (explicit) |
| Approach | Dynamic / quasi-static | Static | Static | Dynamic |
| Mechanical parameters | $\mu$, restitution, density | $\mu$, density | $\mu$, density | $\mu$, joint stiffness, density, $E$, $G$ |
| Max feasible assembly size (blocks) | $10^3$–$10^4$ | $10^1$–$10^2$ | $10^2$–$10^3$ | $10^3$–$10^4$ |
| Computationally efficient | ◐ | ✗ | ✓ | ◐ |
| Settlements / displacement capacity | ✓ | — | — | ✓ |
| Deformable blocks | ◐ (not yet exposed in CARBCOMN) | — | — | ✓ |
| Infeasibility localisation | — | ✓ (via penalty) | ✓ (via penalty) | — |
| Install complexity | Medium | Low | Low | High (license required) |

## Recommendation for CARBCOMN workflows

* **Default:** Use **LMGC90** for all standard analyses. It handles the full floor models (300–500 blocks) robustly and supports the support-displacement load cases needed for min/max thrust analysis.
* **Validation:** Run **CRA** on the simpler examples (`000_threeblocks`, `100_arch`) to cross-validate LMGC90 results. CRA's limit-analysis answer is mathematically rigorous on small assemblies and serves as the trustworthy reference for benchmarking dynamic solvers.
* **Early iteration:** Use **RBE** when iterating quickly over parameter variations on classical geometries (arches, vaults, planar interfaces) where the qualitative force flow matters more than precise magnitudes. **Do not rely on RBE stability verdicts for complex 3D assemblies.**
* **Final verification:** Use **3DEC** for high-stakes analyses where validated commercial-grade joint models, regulatory documentation, or dynamic loading scenarios are required, if a license is available.

## Comparing solvers on the same model

The arch example (`100`) provides the cleanest comparison: scripts `141_dem_problem_LMGC90.py` and `142_dem_problem_CRA.py` solve the identical problem with LMGC90 and CRA respectively. Comparing their contact force outputs is a good way to build confidence in the LMGC90 results.

```python
# In 141_dem_problem_LMGC90.py
problem.solve(Solver.LMGC90())

# In 142_dem_problem_CRA.py
problem.solve(Solver.CRA())
```