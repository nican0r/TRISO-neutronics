# Stage 0 Results

Reference baseline for the Stage 0 handoff audit (step-5).
Produced with `python -m triso.run` using the configuration described below.

## Eigenvalue run

| Metric | Value |
|--------|-------|
| k-eff | **1.10184 ± 0.00103** |
| Cross-section library | ENDF/B-VIII.0 (`data/endfb80_hdf5/cross_sections.xml`) |
| Temperature | 1200 K (uniform — kernel, layers, and matrix) |
| Total batches | 200 |
| Inactive batches | 50 |
| Particles per batch | 5 000 |
| Active particle-histories | 750 000 |
| Statepoint file | `statepoint.200.h5` |

## Geometry

| Metric | Value |
|--------|-------|
| Target packing fraction | 0.30 |
| Compact radius | 0.62 cm |
| Compact height | 2.50 cm |
| Particle outer radius (OPyC) | 0.0390 cm |
| Boundary conditions | All-reflective (k-inf lattice) |

> **Note**: `pack_spheres` is stochastic; the actual packing fraction
> varies ±0.001–0.005 per geometry build. The target (0.30) is the
> design value; exact particle count differs between runs.

## Kernel reaction rates

Fresh fuel, UCO kernel, per source neutron. Absorption = capture + fission.

### Full spectrum (direct tally)

From the `kernel reaction rates` tally in `statepoint.200.h5` (0–20 MeV, 150 active batches).

| Score | Mean | Std. dev. | Relative |
|---|---|---|---|
| Fission | 4.525423 × 10⁻¹ | 7.63 × 10⁻⁴ | 0.17% |
| Absorption | 9.923903 × 10⁻¹ | 1.20 × 10⁻³ | 0.12% |

Fission / absorption: **0.4560 ± 0.0010** (capture-to-fission α = 1.193).
Consistent with k-eff: 1.10184 / 0.45254 gives ν ≈ 2.435.

### Thermal only, < 0.625 eV (reconstructed)

Not a direct tally — the reaction-rate tally has no energy filter. Computed as
thermal kernel flux × atom density × effective thermal cross section:

- Thermal kernel flux from the 3-group flux tally in `statepoint.200.h5`: 0.18517 ± 0.00065.
- Effective cross sections from the 100-group `kernel spectrum` tally in
  `output/depletion/openmc_simulation_n0.h5` folded with 1200 K ENDF/B-VIII.0 data
  (flat-in-lethargy within each group): Σ_f = 0.868 cm⁻¹, Σ_a = 1.055 cm⁻¹.

| Score | Mean | Std. dev. | Relative |
|---|---|---|---|
| Fission | 1.6073 × 10⁻¹ | 6.3 × 10⁻⁴ | 0.39% |
| Absorption | 1.9537 × 10⁻¹ | 7.6 × 10⁻⁴ | 0.39% |

Fission / absorption: **0.823** (α = 0.215). Thermal neutrons account for ≈ 36% of
kernel fissions and ≈ 20% of kernel absorptions.

> **Note**: the standard deviations are statistical only. The within-group
> cross-section averaging adds a method error of up to ~1%. Using the depletion
> run's own thermal flux for the magnitude gives values 1.2% higher. The same
> fold is not valid above the thermal range: 100 groups cannot resolve the
> U-238 resonances (it gives 0.277 for the full-spectrum ratio vs 0.456 tallied).

## Depletion run (step-6)

Produced with `python -m triso.depletion data/chain_endfb71_thermal.xml` from `output/depletion/`.

| Setting | Value |
|---|---|
| Power density | 61.6 W/cm³ (compact volume) |
| Target burnup | 15% FIMA |
| Total irradiation | 620.2 days (≈ AGR-1 duration) |
| Time-step scheme | 5×1d + 5×10d + 12×47.1d (22 steps) |
| Integrator | CECMIntegrator (CE/CM predictor-corrector, 2 solves/step) |
| Particles per batch | 2 000 |
| Batches per step | 100 (30 inactive) |
| Chain file | `data/chain_endfb71_thermal.xml` (ENDF/B-VII.1, 3819 nuclides) |
| Normalization | `fission-q` |
| Results file | `output/depletion/depletion_results.h5` |

### k-eff vs. burnup

| Step | Time [d] | k-eff | σ |
|---:|---:|---:|---:|
| 0 | 0.0 | 1.10207 | 0.00236 |
| 1 | 1.0 | 1.09526 | 0.00238 |
| 2 | 2.0 | 1.09430 | 0.00231 |
| 3 | 3.0 | 1.09251 | 0.00221 |
| 4 | 4.0 | 1.08972 | 0.00247 |
| 5 | 5.0 | 1.08803 | 0.00197 |
| 6 | 15.0 | 1.08469 | 0.00250 |
| 7 | 25.0 | 1.08013 | 0.00229 |
| 8 | 35.0 | 1.08176 | 0.00223 |
| 9 | 45.0 | 1.07763 | 0.00200 |
| 10 | 55.0 | 1.07382 | 0.00227 |
| 11 | 102.1 | 1.05999 | 0.00214 |
| 12 | 149.2 | 1.04757 | 0.00227 |
| 13 | 196.3 | 1.02931 | 0.00231 |
| 14 | 243.4 | 1.01512 | 0.00222 |
| 15 | 290.5 | 1.00123 | 0.00203 |
| 16 | 337.6 | 0.98676 | 0.00191 |
| 17 | 384.7 | 0.98139 | 0.00204 |
| 18 | 431.8 | 0.96917 | 0.00204 |
| 19 | 478.9 | 0.95444 | 0.00203 |
| 20 | 526.0 | 0.94552 | 0.00187 |
| 21 | 573.1 | 0.93303 | 0.00207 |
| 22 | 620.2 | 0.92479 | 0.00217 |

k-eff crosses unity at approximately **day 290** (end-of-life for the compact in isolation).
k-eff swing: **1.102 → 0.925** over 620 days (Δk ≈ −0.177, ≈ −285 pcm/day average).

---

## Audit tolerance

The handoff audit (`scripts/audit_stage0.py`) accepts a re-run as
matching if |k_new − 1.10184| ≤ 3 σ_new (approximately 3 × 0.001 ≈ 0.003).
