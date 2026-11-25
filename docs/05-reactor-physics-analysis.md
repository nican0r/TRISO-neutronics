# Reactor Physics Analysis

Two sensitivity analyses are run against the baseline eigenvalue model: a fuel temperature coefficient (Doppler feedback) sweep, and a manufacturing tolerance study on packing fraction and kernel radius. Both use the same compact geometry and ENDF/B-VIII.0 nuclear data as the baseline run.

---

## Fuel temperature coefficient (Doppler)

The fuel temperature coefficient (FTC) measures how reactivity changes as the UCO kernel temperature changes, with all other materials held fixed. Isolating the kernel temperature from the moderator temperature ensures the coefficient reflects only the Doppler broadening of fuel resonances, not a mixed fuel+moderator response.

**Method.** Four eigenvalue calculations are run at kernel temperatures of 900 K, 1050 K, 1200 K, and 1873 K (~1600 °C, the HTGR accident temperature), with the buffer, PyC, SiC, and graphite matrix held at 1200 K. The analysis is repeated at both BOL (fresh fuel) and EOL (15% FIMA kernel composition from the depletion run).

Each k-inf is converted to reactivity, ρ = (k − 1)/k, in pcm, with σ_ρ = σ_k/k². The FTC is the slope dρ/dT of a weighted linear regression of ρ against temperature (weights 1/σ_ρ), which gives pcm/K directly; the quoted uncertainty is the 1σ slope error propagated from the Monte Carlo uncertainties. Fitting k-inf itself and scaling by 10⁵ is not equivalent: it differs from dρ/dT by a factor of k², which matters here because k-inf is ~1.10 at BOL and ~0.93 at EOL.

At 900 K and 1200 K the tabulated cross sections are used directly. At 1050 K and 1873 K, OpenMC uses sqrt(T) interpolation between the bounding tabulated points (900/1200 K and 1200/2500 K respectively). The Doppler coefficient falls off roughly as 1/√T, so the single slope over 900–1873 K is an average over that interval rather than the local value at the 1200 K operating point.

**Results.**

| Burnup | T_fuel | k-inf | σ_k (pcm) | ρ (pcm) | σ_ρ (pcm) |
|---|---|---|---|---|---|
| BOL | 900 K | 1.12846 | 256 | +11383 | 201 |
| BOL | 1050 K | 1.11212 | 254 | +10082 | 205 |
| BOL | 1200 K | 1.09865 | 224 | +8979 | 186 |
| BOL | 1873 K | 1.06112 | 240 | +5760 | 213 |
| EOL | 900 K | 0.94377 | 201 | −5958 | 226 |
| EOL | 1050 K | 0.93901 | 249 | −6495 | 282 |
| EOL | 1200 K | 0.92832 | 207 | −7722 | 240 |
| EOL | 1873 K | 0.89468 | 189 | −11772 | 236 |

| Burnup | FTC = dρ/dT (pcm/K) |
|---|---|
| BOL | −5.57 ± 0.28 |
| EOL | −6.08 ± 0.32 |

The sign is negative at both burnup points, as expected: Doppler broadening of U-238 resonances increases parasitic absorption as temperature rises, reducing reactivity. The mechanism is a fundamental safety feature of HTGR fuel — it provides immediate negative feedback when the fuel heats up.

The magnitude sits at the strong end of the published full-core HTGR range of −2 to −6 pcm/K (Kuijper et al., NSE 153 (2006); IAEA-TECDOC-978); the EOL value is marginally outside it, by much less than its uncertainty. A value toward the strong end is expected: published values are for compacts surrounded by large graphite moderator blocks that thermalise the spectrum. The compact-only reflective geometry has far less moderator-to-fuel ratio, producing a harder neutron spectrum (see [Depletion — Spectral hardening](04-depletion.md)), so more neutrons slow down through the U-238 resolved resonance region and the Doppler effect carries more weight. The compact-only FTC cannot be compared directly to full-core published values without adding the graphite sleeve and moderator blocks to the geometry.

**Burnup dependence.** The FTC does not change measurably with burnup: the BOL–EOL difference is 0.5 ± 0.4 pcm/K, about 1.2σ. The nominally stronger EOL value is mostly a normalisation effect rather than stronger Doppler physics. Since dρ/dT = (1/k) · d(ln k)/dT, the same fractional change in k-inf is worth more reactivity when k-inf is lower. The fractional coefficient d(ln k)/dT — the part that reflects the resonance physics — is about −6.0 pcm/K at BOL and −5.5 pcm/K at EOL, i.e. marginally weaker at EOL if anything, and the 1/k factor (0.91 at BOL, 1.07 at EOL) reverses the ordering.

That the fractional Doppler effect is nearly unchanged is physically reasonable, although the individual contributions below were not tallied separately:

- U-238, the primary Doppler absorber, depletes by only a few percent at 15% FIMA; most of the burnup is U-235 fission.
- Pu-240 accumulates through successive captures (U-238 → Pu-239 → Pu-240). Its 1.06 eV resonance is almost purely a capture resonance, so like the U-238 resonances it adds negative fuel-temperature feedback. Because the graphite is held at 1200 K here, only the Doppler-broadening part of that feedback is captured, not the larger contribution from a moderator-driven thermal spectrum shift.
- Fission products and plutonium remove thermal neutrons, hardening the spectrum and lowering k-inf.

Resolving a real burnup trend would need tighter statistics and per-nuclide capture tallies (U-238, Pu-240) at each temperature.

---

## Manufacturing tolerance sensitivity

This analysis quantifies how much k-inf changes when the two most variable AGR-1 fabrication parameters — packing fraction and kernel diameter — are shifted by their manufacturing specification tolerances.

**Perturbation scheme.** Five BOL eigenvalue calculations are run: a nominal baseline and four perturbations, each varying one parameter at a time.

| Case | Packing fraction | Kernel ⌀ |
|---|---|---|
| Nominal | 30% | 350 µm |
| PF − 2 pp | 28% | 350 µm |
| PF + 2 pp | 32% | 350 µm |
| Kernel − 10 µm | 30% | 340 µm |
| Kernel + 10 µm | 30% | 360 µm |

The ±2 percentage-point packing fraction shift is an absolute change (28% and 32%), consistent with how reactor physics sensitivity studies report perturbations. The ±10 µm kernel diameter shift (±5 µm radius) comes directly from the AGR-1 manufacturing specification in INL/EXT-10-19476 Table 3.

For the kernel perturbations, only the kernel sphere radius changes. All coating-layer outer radii are held at nominal values, so the buffer layer absorbs the delta and the OPyC outer radius — which sets the lattice pitch — remains unchanged. This isolates the fuel-volume effect from any packing geometry effect.

Each run uses the same settings as the baseline eigenvalue run (200 batches, 50 inactive, 5,000 particles per batch), giving σ(k-eff) ≈ 10–20 pcm and Δk statistical uncertainty ≈ 14–28 pcm. This is sufficient to resolve sensitivity differences of a few hundred pcm with better than 10% precision.

The analysis is BOL only. Manufacturing tolerances affect the initial fuel loading; the BOL k-eff sensitivity is the most direct measure of how fabrication variation translates to reactivity uncertainty. How this sensitivity evolves with burnup is deferred to a later stage.
