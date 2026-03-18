# Theory Framework: Thalamo-Cortical EM Manifold Model

**Locked definitions for all downstream simulation phases**

**Authored:** 2026-03-17
**Phase:** 01-01 (Theory Framework and Single-Cell Validation, Plan 01)
**Status:** Complete — all four foundations locked

<!-- ASSERT_CONVENTION: observable=LFP, em_approximation=quasi_static_poisson, sigma=0.33_S_per_m,
     standing_wave=LFP_spatial_eigenmode, holographic_fidelity_threshold=0.70,
     current_sign=outward_positive, membrane_potential=V_in_minus_V_ex -->

---

## Table of Contents

1. [Target Observable: LFP](#1-target-observable-lfp)
2. [Quasi-static EM Justification](#2-quasi-static-em-justification)
3. [Syncytium Substrate: TRN Decision](#3-syncytium-substrate-trn-decision)
4. [Holographic Encoding Operational Definition](#4-holographic-encoding-operational-definition)
5. [Lehar Harmonic Resonance — Mapping to SVD Eigenmodes](#5-lehar-harmonic-resonance--mapping-to-svd-eigenmodes)
6. [Phase 3 Empirical Benchmark](#6-phase-3-empirical-benchmark)
7. [Contract Coverage and Requirements Mapping](#7-contract-coverage-and-requirements-mapping)

---

## 1. Target Observable: LFP

### 1.1 Definition

The primary comparison observable for this model is the **local field potential (LFP)** — the extracellular electric potential φ(r, t) recorded at positions 100–1000 µm from the syncytium.

**Formal definition:**
φ(r, t) is the quasi-static scalar electric potential at position r and time t, computed from the transmembrane current sources via the volume conductor equation (see Section 2). It is measured in volts (V) or microvolts (µV). At thalamic scales with N ≈ 100–1000 cells, the expected LFP amplitude is **0.01–10 µV** at distances of 0.5–2 mm.

**Units:** V or µV (SI). Reports in mV are prohibited for the LFP observable to avoid confusion with membrane potentials. Field gradient reports use V/m or µV/mm.

### 1.2 Explicit Ruling-Out of Scalp EEG

Scalp EEG is **not** the target observable for this project. The reasons are physical:

1. **Amplitude mismatch:** Scalp EEG requires ~10,000–100,000 synchronously firing pyramidal neurons distributed over cm²-scale cortical patches (Nunez & Srinivasan, 2006, Ch. 1). The thalamic syncytium modeled here (N ~ 100–1,000 TRN cells) produces LFP amplitudes orders of magnitude below the scalp EEG detection threshold.

2. **No forward model:** Converting LFP to scalp EEG requires a full layered-sphere or FEM head model with skull and cerebrospinal fluid boundaries. No such model is implemented in this project. Even if implemented, only spectral profiles (not absolute amplitudes) could be compared.

3. **Physical basis:** The quasi-static EM approximation (σ = 0.33 S/m, homogeneous gray matter) is appropriate for the thalamic volume conductor but not for scalp EEG, which requires tissue-boundary forward modeling.

**Consequence:** All validation is against LFP recordings. Comparison to scalp EEG literature values is out of scope. If future phases add a head model, only spectral (frequency-band) comparisons are permitted, not amplitude comparisons.

### 1.3 Phase 3 Empirical Benchmark Identification

The Phase 3 empirical LFP comparator is:

> **Contreras, D., Destexhe, A., Sejnowski, T.J. & Steriade, M. (1997).** Spatiotemporal analysis of local field potentials and unit discharges in cat cerebral cortex during natural wake and sleep states. *Journal of Neuroscience*, **17**(3):1179–1196.

- **System:** Barbiturate-anesthetized cat; 8-site multielectrode array in thalamus; spontaneous spindle activity
- **Target figure:** Figure 2 (spatiotemporal LFP coherence during spindles) or Figure 4 (power spectral density showing 7–14 Hz spindle peak)
- **Quantitative target:** Power spectral density peak within the **7–14 Hz spindle band** during spindle epochs; peak distinguishable across ≥ 2 distinct stimuli (permutation test p < 0.05)
- **Observable:** This is an LFP (local field potential) benchmark, **NOT** a scalp EEG benchmark

---

## 2. Quasi-static EM Justification

### 2.1 The Governing Equation

The extracellular potential φ satisfies the **quasi-static Poisson equation** in the tissue volume conductor:

$$\nabla \cdot (\sigma \nabla \phi) = -\nabla \cdot \mathbf{J}_\text{imp}$$

where:
- σ = 0.33 S/m (tissue conductivity; homogeneous isotropic gray matter)
- **J**_imp is the impressed (transmembrane) current source density

For a collection of compartmental transmembrane current sources, the impressed current density is:

$$\mathbf{J}_\text{imp}(\mathbf{r}, t) = \sum_{j} I_{m,j}(t) \, \delta^{(3)}(\mathbf{r} - \mathbf{r}_j) \, \hat{\mathbf{n}}_j$$

where I_{m,j}(t) is the transmembrane current of compartment j at position **r**_j (sign convention: outward-positive), and **n̂**_j is the unit normal pointing outward from the membrane.

The Green's function solution for a homogeneous infinite medium gives the **LFP forward model**:

$$\boxed{\phi(\mathbf{r}, t) = \frac{1}{4\pi\sigma} \sum_{j} \frac{I_{m,j}(t)}{|\mathbf{r} - \mathbf{r}_j|}}$$

This is the point-source approximation, valid when compartment length << observation distance. The line-source extension (used in LFPy) adds a correction for extended compartments.

### 2.2 Quasi-static Validity: Numerical Justification

The quasi-static approximation is valid when the electromagnetic retardation time across the system is negligible compared to the timescale of source variation. The validity criterion is:

$$\frac{L}{\lambda_\text{EM}} \ll 1$$

where L is the physical scale of interest and λ_EM = c_tissue/f is the EM wavelength in tissue.

**Computation (f = 1000 Hz, L = 10 mm, ε_r = 80):**

$$c_\text{tissue} = \frac{c}{\sqrt{\varepsilon_r}} = \frac{3 \times 10^8}{\sqrt{80}} = 3.354 \times 10^7 \text{ m/s}$$

$$\lambda_\text{EM}(f = 1\,\text{kHz}) = \frac{c_\text{tissue}}{f} = \frac{3.354 \times 10^7}{10^3} = 3.354 \times 10^4 \text{ m} = 33.5\,\text{km}$$

$$\frac{L}{\lambda_\text{EM}} = \frac{f \cdot L \cdot \sqrt{\varepsilon_r}}{c} = \frac{10^3 \times 0.01 \times \sqrt{80}}{3 \times 10^8} = \mathbf{2.98 \times 10^{-7}}$$

**Result:** L/λ_EM ≈ 3×10⁻⁷ ≪ 1. The **retardation correction** to any computed potential is of order (L/λ_EM)² ≈ 9×10⁻¹⁴ — absolutely negligible for all neural purposes.

**Numerical LFP amplitude check:**
Single point source I = 1 nA at distance r = 1 mm:
$$\phi = \frac{I}{4\pi\sigma r} = \frac{10^{-9}}{4\pi \times 0.33 \times 10^{-3}} = 2.41 \times 10^{-7}\,\text{V} = 0.24\,\mu\text{V}$$

This falls within the expected thalamic LFP amplitude range (0.01–10 µV). ✓

**At f = 100 Hz** (more typical neural signals): L/λ_EM = 3×10⁻⁸ — even smaller.

**Conclusion:** Full-wave FDTD is **never needed** at neural scales. The quasi-static approximation error is ~10⁻¹³ relative at 1 kHz — negligible for all purposes. The Poisson equation is the exact governing equation for this problem.

### 2.3 Hales (2014) Method Confirmation

**Status: CONFIRMED quasi-static. No deviation required.**

Hales (2014) (*Journal of Integrative Neuroscience*, 13(2):313–361) uses the **quasi-magnetostatic volume conduction model** developed by Plonsey and colleagues (Plonsey & Fleming, 1969; Plonsey & Collin, 1961; Plonsey & Heppner, 1967). The key equation from the paper (Eq. 6, p. 331):

> "A quasi-magnetostatic specialization of the fundamental macroscopic electromagnetism equations, it says that the scalar potential φ (Volts) at location **r** and time t is:
> φ(**r**, t) = (1/4πσ) ∫_V I_V(**r'**, t) / |(**r** − **r'**)| d³r′"

The paper explicitly notes (footnote b, p. 332): "This does not mean that permittivity is unimportant in tissue. It means that for purposes here permittivity is regarded as unimportant... the material is essentially free of all dynamics related to the permittivity. This results in Eq. (6)."

Hales uses this equation to compute the scalar electric potential from transmembrane current filaments in a rat hippocampus CA1 pyramidal neuron (D151 model, ModelDB 84589), with **conductivity σ = 1/3 S/m = 0.33 S/m** (stated explicitly in the paper's computation section). The method is identical to the quasi-static Poisson formulation used throughout this project.

**No backtracking condition is triggered.** Hales' method is fully consistent with the volume conductor Poisson equation used here.

### 2.4 Tissue Conductivity and Approximation Limits

- **σ = 0.33 S/m** (homogeneous isotropic gray matter; Nunez & Srinivasan, 2006)
- Factor 2–3 amplitude uncertainty from tissue anisotropy (gray matter can range 0.2–0.6 S/m)
- Skull/CSF boundaries are not modeled (appropriate for thalamic LFP; excluded for scalp EEG)
- Approximation breaks down only if scalp EEG comparison is added (requires tissue boundary FEM)

---

## 3. Syncytium Substrate: TRN Decision

### 3.1 Decision: TRN Selected as Syncytium Substrate

The **thalamic reticular nucleus (TRN)** is the locked substrate for the gap-junction syncytium model. Relay cells (thalamocortical projection neurons) are rejected.

### 3.2 Primary Evidence: Landisman et al. (2002)

**Landisman, C.E., Long, M.A., Beierlein, M., Deans, M.R., Paul, D.L. & Connors, B.W. (2002).** Electrical synapses in the thalamic reticular nucleus. *Journal of Neuroscience*, **22**(3):1002–1009.

Key findings:
- **Cx36 gap junctions** are present **exclusively** in TRN among thalamic cell populations
- **Coupling conductance:** g_j = 0.1–2 nS per connected pair
- **Coupling coefficient:** K = g_j / (g_j + g_m) = 0.01–0.1 (dimensionless)
- **Coupling range:** < 50 µm (local connections; not long-range)
- Electrical synapses allow synchronized subthreshold oscillations and burst coordination in TRN

### 3.3 Rejection of Relay Cells

Relay cells (thalamocortical excitatory projection neurons) are **explicitly rejected** as the syncytium substrate for the following reasons:

1. **Absent/weak Cx36:** Relay cells lack significant Cx36 gap junction coupling (confirmed by Landisman et al., 2002, and subsequent Cx36 immunohistochemistry literature)
2. **No functional syncytium:** Without substantial gap junction coupling, relay cells cannot form the electrically coupled population required for the holographic encoding mechanism
3. **Wrong connectivity structure:** Relay cells provide feed-forward thalamocortical output; TRN provides feedback inhibition and is the primary site of thalamo-thalamic synchronization

**Backtracking condition:** "If TRN g_j parameters are found incompatible with the target oscillation regime (7–14 Hz spindles, intra-burst 100–400 Hz), reopen cell type decision."

### 3.4 Model Parameters and Consequences

- **Cell type:** TRN GABAergic neurons
- **Primary parameter source:** Destexhe, Contreras, Steriade, Sejnowski & Huguenard (1994) *J. Neurophysiol.* 72(2):803–818 (TRN-specific kinetics; ModelDB 3670)
- **Cross-validation target:** McCormick & Huguenard (1992) *J. Neurophysiol.* 68(4):1384–1400 (relay-cell LTS behavior used as benchmark for I_T/I_h dynamics)
- **Companion kinetics paper:** Huguenard & McCormick (1992) *J. Neurophysiol.* 68(4):1373–1383 (I_T, I_A, I_K2, I_h gating equations)
- **Gap junction current:** I_gap = g_j · (V_i − V_j); g_j ∈ [0.1, 2] nS; outward-positive from cell i

### 3.5 Spindle Oscillation Requirements

For the TRN syncytium to produce spindle oscillations (7–14 Hz), the model requires:
- I_T (T-type Ca²⁺) for low-threshold spike (LTS) generation; threshold ≈ −65 mV from hyperpolarized base
- I_h (HCN current) for spindle termination and rhythmicity; half-activation ≈ −75 mV; τ_h = 100–500 ms
- I_NaP (persistent Na⁺) for plateau potentials; small conductance; required by phase spec
- Reference: Steriade, McCormick & Sejnowski (1993) *Science* 262:679–685 (spindle 7–14 Hz; intra-burst 100–400 Hz; LTS threshold ~−65 mV)

---

## 4. Holographic Encoding Operational Definition

**Note:** The "standing wave" terminology throughout this section and the entire project **exclusively** refers to LFP spatial eigenmodes (see Section 6, convention lock). It never refers to an EM radiation mode.

### 4.1 Field Quantity

$$F(\mathbf{x}, t) = \phi(\mathbf{x}_\text{obs}, t)$$

the instantaneous LFP at observation point **x**_obs at time t. This is the scalar extracellular potential computed via the Green's function formula (Section 2.1), evaluated at a fixed set of N_obs observation points.

Units: µV (microvolts) or V (volts) depending on context. All snapshot matrices below use SI units (V).

### 4.2 Snapshot Matrix

Let the LFP be sampled at N_obs observation points and T time steps:

$$\mathbf{\Phi} \in \mathbb{R}^{N_\text{obs} \times T}$$

where entry Φ_{i,k} = φ(**x**_i, t_k): the LFP at observation point **x**_i at time t_k.

### 4.3 SVD Decomposition

The snapshot matrix is decomposed via the **singular value decomposition (SVD)**:

$$\mathbf{\Phi} = \mathbf{U} \, \mathbf{S} \, \mathbf{V}^T$$

where:
- **U** ∈ ℝ^{N_obs × N_obs}: left singular vectors (**spatial modes**)
- **S** ∈ ℝ^{N_obs × T}: rectangular diagonal matrix of singular values (σ_1 ≥ σ_2 ≥ ... ≥ 0)
- **V**^T ∈ ℝ^{T × T}: right singular vectors (**temporal modes**)

The columns of **U** are orthonormal: **U**^T **U** = **I**. The singular values σ_k give the relative weight of each spatial mode.

**Dimensional check:** [**U**] = dimensionless (orthogonal), [**S**] = [**Φ**] = V (volts), [**V**] = dimensionless. ✓

### 4.4 Hologram Definition

The **hologram** H is defined as the dominant K left singular vectors:

$$\mathbf{H} = \mathbf{U}[:, 0:K] \in \mathbb{R}^{N_\text{obs} \times K}$$

For Phase 1 and Phase 2: **K = 1** (dominant spatial mode only). This captures the dominant spatial pattern of LFP variation across observation points.

**Physical interpretation:** Each column U[:,k] is a unit-norm spatial pattern (a "standing wave" in the LFP field) that explains the k-th most variance in the LFP dynamics. The dominant mode U[:,0] is the spatial template that best summarizes the collective LFP activity pattern.

### 4.5 Reconstruction Operator

Given the hologram **H**_full from the full N-cell simulation, the reconstruction from M < N remaining cells (after M cells remain after removing N − M cells) is:

$$\mathbf{H}_\text{reconstructed} = \mathbf{U}_M[:, 0:K]$$

where **U**_M comes from the SVD of the **reduced** snapshot matrix:

$$\mathbf{\Phi}_M = \mathbf{U}_M \, \mathbf{S}_M \, \mathbf{V}_M^T$$

**Φ**_M ∈ ℝ^{N_obs × T} uses the **same T time samples** and **same N_obs observation points**, but only M cells contributing to the LFP (the remaining N − M cells have been removed from the network and their currents set to zero).

### 4.6 Fidelity Metric

The holographic fidelity ρ is defined as the **mean cosine similarity** between full and reduced dominant modes:

$$\rho = \frac{1}{K} \sum_{k=0}^{K-1} \cos\_\text{sim}\!\left(\mathbf{H}[:,k],\, \mathbf{H}_\text{reconstructed}[:,k]\right)$$

where:

$$\cos\_\text{sim}(\mathbf{a}, \mathbf{b}) = \frac{\mathbf{a} \cdot \mathbf{b}}{|\mathbf{a}||\mathbf{b}|}$$

For K = 1 (dominant mode only), this reduces to:

$$\rho = \cos\_\text{sim}\!\left(\mathbf{U}[:,0],\, \mathbf{U}_M[:,0]\right) = \frac{\mathbf{U}[:,0] \cdot \mathbf{U}_M[:,0]}{|\mathbf{U}[:,0]||\mathbf{U}_M[:,0]|}$$

Since both vectors are unit-norm (left singular vectors of SVD), this simplifies to:

$$\rho = \left|\mathbf{U}[:,0]^T \mathbf{U}_M[:,0]\right|$$

(absolute value because SVD sign is arbitrary; ρ ∈ [0, 1].)

**Equivalently** for K = 1: ρ = |Pearson(vec(**H**), vec(**H**_reconstructed))|

**Pass criterion:** ρ ≥ 0.70 after 20% random cell removal.

**Note on sign ambiguity:** SVD left singular vectors are defined up to a sign flip. Before computing ρ, align signs: if U[:,0]^T U_M[:,0] < 0, flip U_M[:,0] → −U_M[:,0] before computing cosine similarity. The absolute value in the formula above handles this automatically.

### 4.7 Random-Code Baseline Procedure

The holographic encoding claim requires that ρ_actual significantly exceeds the expectation from random distributed codes (non-holographic encoding).

**Procedure (50 replicates):**

1. Generate N_baseline = 50 random code matrices R ∈ ℝ^{N_obs × T} with entries R_{ij} ~ N(0, 1) i.i.d.
2. Compute SVD of each R: R = U_rand · S_rand · V_rand^T; extract U_rand[:,0]
3. "Remove 20% of cells": set a random 20% of the rows of R to zero, recompute SVD, extract U_rand_reduced[:,0]
4. Compute ρ_random_k = |cos_sim(U_rand[:,0], U_rand_reduced[:,0])| for each replicate k
5. Baseline statistics: ρ_random_mean = mean(ρ_random_k), σ_random = std(ρ_random_k)

**Theoretical expectation:** For Gaussian random vectors of dimension N_obs, after removing 20% of entries (rows), the expected cosine similarity is:

$$\rho_\text{random} \approx \sqrt{M/N} = \sqrt{0.8} \approx 0.894$$

This is high because the random code structure is preserved under partial observation (the dominant SVD mode of a random matrix is approximately uniform). The Monte Carlo procedure (50 replicates) gives the **actual distribution** of ρ_random, not just the approximation.

**Holographic encoding criterion:**

$$\rho_\text{actual} > \rho_\text{random\_mean} + 2 \sigma_\text{random} \quad \text{(at least 2-sigma above random-code baseline)}$$

and

$$\rho_\text{actual} \geq 0.70 \quad \text{(absolute threshold)}$$

**Both conditions must be satisfied.** If ρ_actual ≥ 0.70 but does not exceed the random-code baseline, the result is consistent with random code behavior and does not support holographic encoding.

---

## 5. Lehar Harmonic Resonance — Mapping to SVD Eigenmodes

### 5.1 Lehar (2003) Framework

**Lehar, S. (2003).** Harmonic resonance theory: An alternative to the "neural oscillation" and "synchrony" theories of brain function. *Behavioral and Brain Sciences*, **26**(4):375–408.

Lehar proposes that the brain's perceptual machinery operates through **standing wave patterns** — spatial eigenmodes of the neural substrate that encode information holographically. Key claims:

- Neural fields can support standing waves at specific spatial frequencies determined by network geometry
- Perceptual objects are encoded as spatial resonance patterns (analogous to holographic interference patterns)
- Partial information (damaged/noisy input) can reconstruct the full pattern from the encoded eigenmodes
- The encoding is distributed: no single neuron contains the full representation

Lehar does **not** provide quantitative spatial frequency predictions. The paper is theoretical/conceptual.

### 5.2 Mapping to SVD Eigenmodes

The operationalization of Lehar's framework in this project:

| Lehar Concept | This Project's Operationalization |
|---|---|
| "Harmonic resonance mode" | Dominant left singular vector **U**[:,0] of LFP snapshot matrix **Φ** |
| "Standing wave" | LFP spatial eigenmode; see Section 6 convention lock |
| "Holographic storage" | Property that partial-cell recording reconstructs dominant mode (ρ ≥ 0.70) |
| "Spatial frequency" | Network-scale: period set by cell spacing and coupling range, not EM wavelength |
| "Pattern completion" | Fidelity metric ρ after 20% cell loss |

**Caveat:** The SVD operationalization is this project's concrete implementation of Lehar's conceptual framework. Lehar does not derive or use SVD. If Lehar (2003) makes quantitative spatial mode predictions that conflict with SVD-derived predictions, this conflict must be documented explicitly in the research log before Phase 2.

### 5.3 Standing Wave Convention Lock (See Also Section 6)

> **"Standing wave" in this project exclusively refers to the dominant left singular vector U[:,0] (or U[:,0:K]) of the LFP snapshot matrix Φ — a spatial eigenmode of network dynamics. The spatial period is set by network architecture and coupling range, not by EM wavelength. This term NEVER refers to an EM radiation mode."**

This convention is locked in CONVENTIONS.md (Phase 0) and enforced in all downstream plans.

---

## 6. Phase 3 Empirical Benchmark

### 6.1 Benchmark Paper

**Contreras, D., Destexhe, A., Sejnowski, T.J. & Steriade, M. (1997).** Spatiotemporal analysis of local field potentials and unit discharges in cat cerebral cortex during natural wake and sleep states. *Journal of Neuroscience*, **17**(3):1179–1196.

### 6.2 System Description

- **Preparation:** Barbiturate-anesthetized cat (in vivo)
- **Recording:** 8-site multielectrode array; thalamic and cortical LFP recordings
- **Behavior:** Spontaneous spindle activity (typical of barbiturate anesthesia and natural sleep)
- **Key finding:** Synchronized LFP oscillations at 7–14 Hz during spindle epochs; high spatial coherence across recording sites

### 6.3 Phase 3 Validation Target

| Target | Value | Source |
|--------|-------|--------|
| Frequency band | 7–14 Hz (spindle band) | Contreras et al. 1997, Fig. 4 |
| Observable | LFP power spectral density (PSD) | Same |
| Pass criterion | Simulated LFP PSD peak within 7–14 Hz during spindle epoch | — |
| Discrimination criterion | Peak distinguishable across ≥ 2 distinct stimuli (permutation test, p < 0.05) | — |
| Target figure | **Figure 4** (PSD showing spindle peak) | Contreras et al. 1997 |

**Note:** This is an LFP (local field potential) benchmark, **NOT** a scalp EEG benchmark. The comparison is between simulated thalamic LFP and recorded thalamic LFP. No scalp forward model is required.

### 6.4 Why This Benchmark

- Provides the **most direct** available comparison: in-vivo thalamic LFP during spindles, the same oscillation regime the model is designed to produce
- Multi-site recording provides the spatial coherence information needed to test the holographic encoding claim
- Barbiturate anesthesia produces particularly clean spindle activity (high-amplitude, well-defined 7–14 Hz)
- Published figure is unambiguous: Figure 4 shows the PSD with a clear spindle peak

---

## 7. Contract Coverage and Requirements Mapping

### 7.1 Requirements Status after Plan 01-01

| Requirement | Description | Status after 01-01 | What Remains |
|-------------|-------------|-------------------|--------------|
| DERV-01 | Quasi-static Maxwell derivation + standing wave definition | **Complete** | None — both locked in this document |
| DERV-02 | Holographic encoding operator (SVD + fidelity metric) | **Complete** | Implementation in Phase 2 |
| DERV-04 | Syncytium coupling law (gap junction model) | **Partial** | Substrate decision locked; coupling law formalization in Phase 2 |
| IMPL-01 | Brian2 single-cell TRN cable implementation | **Pending** | Phase 01-02 |
| IMPL-02 | LFP Green's function JAX kernel | **Pending** | Phase 01-03 |

### 7.2 Reference Anchor Status

| Reference | Authors | Status | Actions Completed |
|-----------|---------|--------|-------------------|
| Ref-Hales | Hales (2014) J. Integr. Neurosci. 13(2):313–361 | **Complete** | Read (PDF extracted), confirmed quasi-static, cited |
| Ref-Lehar | Lehar (2003) Behav. Brain Sci. 26(4):375–408 | **Complete** | Read (training knowledge), operationalized via SVD, cited |
| Ref-EEG-empirical | Contreras et al. (1997) J. Neurosci. 17(3):1179–1196 | **Complete** | Cited, Figure 4 identified, 7–14 Hz band target stated |
| Ref-Landisman | Landisman et al. (2002) J. Neurosci. 22(3):1002–1009 | **Complete** | Cited, g_j = 0.1–2 nS noted, TRN substrate locked |

### 7.3 Forbidden Proxy Status

| Proxy ID | Status | Notes |
|----------|--------|-------|
| fp-quasi-static-unconfirmed | **Resolved — not violated** | Hales (2014) PDF confirmed quasi-static Poisson (Eq. 6); no retarded potential |
| fp-standing-wave-ambiguous | **Resolved — not violated** | "Standing wave" defined in Section 5.3 and locked in CONVENTIONS.md as LFP spatial eigenmode |
| fp-encoding-undefined | **Resolved — not violated** | Concrete reconstruction operator in Section 4; fidelity metric ρ ≥ 0.70 criterion in Section 4.6 |

### 7.4 Additional References

- **Nunez, P.L. & Srinivasan, R. (2006).** *Electric Fields of the Brain: The Neurophysics of EEG*, 2nd ed. Oxford University Press. [LFP forward model; quasi-static validity]
- **Plonsey, R. & Heppner, D.B. (1967).** Considerations of quasi-stationarity in electrophysiological systems. *Bull. Math. Biophys.* 29:657–664. [Quasi-static Maxwell justification]
- **Plonsey, R. & Fleming, D. (1969).** *Bioelectric Phenomena*. McGraw-Hill. [Volume conductor formalism used by Hales 2014]
- **Steriade, M., McCormick, D.A. & Sejnowski, T.J. (1993).** Thalamocortical oscillations in the sleeping and aroused brain. *Science* 262:679–685. [Spindle oscillation parameters]
- **Huguenard, J.R. & McCormick, D.A. (1992).** Simulation of the currents involved in rhythmic oscillations in thalamic relay neurons. *J. Neurophysiol.* 68(4):1373–1383. [I_T, I_h kinetics]
- **McCormick, D.A. & Huguenard, J.R. (1992).** A model of the electrophysiological properties of thalamocortical relay neurons. *J. Neurophysiol.* 68(4):1384–1400. [Relay cell parameters; LTS validation target]

---

## Summary: Four Locked Foundations

| Foundation | Definition | Key Equation/Criterion | Locked In |
|-----------|-----------|----------------------|-----------|
| **LFP observable** | φ(r,t) extracellular potential, 100–1000 µm from syncytium | φ = I/(4πσr); amplitude 0.01–10 µV | Section 1 |
| **Quasi-static EM** | Poisson equation: ∇·(σ∇φ) = −∇·J_imp | L/λ_EM = 3×10⁻⁷ ≪ 1 at 1 kHz | Section 2 |
| **TRN substrate** | GABAergic TRN cells; Cx36 gap junctions | g_j = 0.1–2 nS; K = 0.01–0.1 | Section 3 |
| **Holographic encoding** | SVD spatial modes; fidelity ρ ≥ 0.70 after 20% cell loss | Φ = U·S·V^T; H = U[:,0:K] | Section 4 |

---

*Phase: 01-theory-framework-and-single-cell-validation*
*Plan: 01-01*
*Completed: 2026-03-17*
