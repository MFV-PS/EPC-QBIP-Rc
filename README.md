# EPC-QBIP-Rc

Critical radius ($R_c$) determination in porous media using the Euler-Poincaré Characteristic (EPC) and Queue-Based Invasion Percolation (QBIP).

This repository accompanies the manuscript:

> Franco-Villegas, M., Nieto-Rivero, C.J.T., Morales-Chávez, S., Valdiviezo-Mijangos, O.C., Coconi-Morales, E., & Fuentes-Cruz, G. (2025). Euler-Poincaré Characteristic and Invasion Percolation for Critical Radius Determination: A Systematic Comparison in Synthetic Porous Structures. *Geofísica Internacional*. <!-- DOI pending -->

---

## Overview

The critical radius $R_c$ is the representative pore-throat radius at the percolation threshold, used as the characteristic length *l* in analytical permeability models ($k = Hl²$). This repository provides two independent methods for $R_c$ determination from 3D binary images of porous media:

- **EPC** — Topological approach based on the connectivity function $χ(r)$, computed via morphological opening at increasing radii. $R_c$ is identified by zero-crossing and derivative criteria.
- **QBIP** — Invasion percolation approach that simulates quasi-static drainage on the binary image. $R_c$ is extracted from the reconstructed pore-throat size distribution (PTSD).

Both methods operate directly on segmented 3D images without pore network extraction.

---

## Repository structure

```
EPC-QBIP-Rc/
├── QBIP_bilingual.ipynb     # QBIP implementation with capillary pressure reconstruction
├── EPC_bilingual.ipynb      # EPC implementation with connectivity function analysis
├── SPS/                     # Synthetic Porous Structures (binary .raw or .npy files)
└── README.md
```

---

## QBIP notebook

`QBIP_bilingual.ipynb`

Implements queue-based invasion percolation on 3D binary images and reconstructs capillary pressure curves for R_c determination.

### Pipeline

1. **Porous medium generation** — Creates synthetic porous structures (BPC, MDC, HTN, DCN) with controlled geometry and topology, or loads external binary images.
2. **Invasion simulation** — Runs QBIP via PoreSpy's `ibip` function with directional inlet specification.
3. **Capillary pressure reconstruction** — Groups invasion events into logarithmic bins by EDT-based radius, then reorders from largest to smallest radius to produce the pressure-controlled $S(r)$ curve and its derivative $dS/dlog(r)$.
4. **Characteristic radii extraction** — Computes $R_c$ (PTSD mode), $R_apex$ (Pittman graphical method), and $R_WGM$ (weighted geometric mean) from the reconstructed curve.
5. **Bin sensitivity analysis** — Evaluates the effect of `n_bins` (20–200) and `min_jump_size` (1e-8 to 1e-3) on $R_c$ to quantify discretization uncertainty.

### Key functions

| Function | Description |
|----------|-------------|
| `create_heterogeneous_tight_rock()` | Generates SPS with trimodal PSD and controlled topology |
| `run_invasion_fast()` | Executes QBIP invasion from a specified boundary face |
| `analyze_capillary_pressure_fast()` | Groups invasion events into bins and reconstructs S(r) |
| `calculate_r_critical_from_distribution()` | Extracts R_c as the PTSD mode (argmax of dS/dlog(r)) |
| `calculate_r_apex_from_distribution()` | Computes R_apex via Pittman's method |
| `calculate_weighted_geometric_mean_radius()` | Computes R_WGM over the invaded throat spectrum |
| `unified_pc_analysis_and_visualization()` | Complete post-processing: bins, radii, and figures |
| `collect_raw_events()` / `bin_and_get_rc()` | Sensitivity analysis of R_c to binning parameters |

### Outputs

- Volume-controlled invasion curve: $S$ vs $r$ in invasion order
- Pressure-controlled reconstruction: cumulative $S(r)$ with PTSD overlay
- Bin sensitivity matrix: $R_c$ as a function of $n_bins$ and min_jump_size

---

## EPC notebook

`EPC_bilingual.ipynb`

Implements the Euler-Poincaré Characteristic method for $R_c$ determination via the connectivity function $χ(r)$.

### Pipeline

1. **Porous medium generation** — Same SPS generator as QBIP, or loads external binary images.
2. **EDT computation** — Euclidean Distance Transform defines the radius range for morphological opening.
3. **Connectivity function** — Iterative morphological opening at n radii from 0 to r_max. At each radius, $χ$ is computed via scikit-image's Euler number calculation on the opened domain P(r) = (P ⊖ B(r)) ⊕ B(r).
4. **R_c identification** — Two criteria: zero-crossing ($χ(r) = 0$) and derivative (argmax |dχ/dr|).
5. **Visualization** — Static comparison panels and animated morphological opening sequences.

### Key functions

| Function | Description |
|----------|-------------|
| `create_heterogeneous_tight_rock()` | Generates SPS (shared with QBIP) |
| `calculate_euler_characteristic_robust()` | Computes $χ$ for a single binary volume |
| `calculate_connectivity_function()` | Iterates morphological opening and evaluates $χ(r)$ at n radii |
| `calculate_connectivity_function_enhanced()` | Extended version with checkpoint support for large volumes |
| `detect_abrupt_changes_in_connectivity()` | Identifies $R_c$ via zero-crossing and derivative criteria |
| `create_epc_static_comparison()` | Generates static figure: 3D structure at selected radii + $χ(r)$ curve |
| `create_animated_epc_evolution()` | Generates animated GIF of the morphological opening process |

### Outputs

- Connectivity function #χ(r)$ with zero-crossing and derivative $R_c$ annotated
- 3D visualization of pore structure evolution during morphological opening
- Animated sequences (optional, computationally intensive for large volumes)

---

## Synthetic Porous Structures

Four SPS with increasing topological complexity are included:

| Structure | Topology | χ₀ | Design modal radius |
|-----------|----------|-----|---------------------|
| BPC | Bundle of parallel capillaries (disconnected) | +40 | 0.15 µm |
| MDC | Multidirectional channels (connected grid) | −15 | 0.10 µm |
| HTN | Hierarchical tree network (MST with branches) | −127 | 0.10 µm |
| DCN | Degree-constrained network ($Z$ = 2–6) | −216 | 0.10 µm |

Domain: 7.5 × 7.5 × 7.5 µm³, voxel resolution: 0.05 µm (150³ voxels).

---

## Requirements

- Python 3.8+
- NumPy
- SciPy
- scikit-image
- PoreSpy ≥ 2.0
- Matplotlib

Install with:

```bash
pip install numpy scipy scikit-image porespy matplotlib
```

---

## Language

Both notebooks support bilingual output (English/Spanish). Set the language in the first cell:

```python
LANG = 'en'  # 'en' for English, 'es' for Spanish
```

---

## Usage

```python
# QBIP example
from QBIP_bilingual import create_heterogeneous_tight_rock, main
medium = create_heterogeneous_tight_rock(structure_type='DCN')
results = main(medium, voxel_size_um=0.05, invasion_face='left')

# EPC example
from EPC_bilingual import create_heterogeneous_tight_rock, main
medium = create_heterogeneous_tight_rock(structure_type='DCN')
results = main(medium, voxel_size_um=0.05, n_points=30)
```

---

## Citation

If you use this code, please cite:

```bibtex
@article{FrancoVillegas2025_EPC_QBIP,
  author  = {Franco-Villegas, Moisés and Nieto-Rivero, Carlos J.T. and Morales-Chávez, Sinai and Valdiviezo-Mijangos, Oscar C. and Coconi-Morales, Enrique and Fuentes-Cruz, Gorgonio},
  title   = {Euler-Poincaré Characteristic and Invasion Percolation for Critical Radius Determination: A Systematic Comparison in Synthetic Porous Structures},
  journal = {Geofísica Internacional},
  year    = {2025},
  note    = {In review}
}
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.
