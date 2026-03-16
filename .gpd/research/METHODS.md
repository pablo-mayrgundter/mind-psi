---
generated: 2026-03-16
domain: Thalamo-cortical EM manifold modeling
dimension: Methods and Analytical Approaches
confidence: MEDIUM (web search unavailable — versions marked [VERIFY])
---

# Methods Research: Thalamo-Cortical EM Manifold

## 1. Analytical Methods

### EM Field from Neural Current Distributions

| Method | Purpose | Why Recommended |
|--------|---------|-----------------|
| Quasi-static volume conductor (Green's function) | φ from distributed cable currents | Valid for f < 1 kHz; analytically tractable; O(N²) per timestep |
| Multipole expansion (dipole, quadrupole) | Far-field EM approximation | Efficient for long-range field; matches EEG/MEG forward model literature |
| Full Maxwell (FDTD) | Near-field EM with wave propagation | Overkill by ~6 orders of magnitude at neural frequencies — avoid |
| Laplace equation (static) | Extracellular potential from soma current | Valid for quasi-static limit; used in standard LFP modeling |

**Key formula (homogeneous medium, σ ≈ 0.33 S/m brain tissue):**
```
φ(r) = (1/4πσ) ∫ J(r') · ∇'(1/|r-r'|) dV'
```
For cable segments: discretize as line current sources, sum Green's function contributions.

### Hodgkin-Huxley Cable Theory

| Method | Purpose | Notes |
|--------|---------|-------|
| Cable equation (Rall 1967) | Voltage propagation along morphology | Fundamental; d²V/dx² = r_a C_m dV/dt + r_a i_m |
| Compartmental model | Discretize cable into isopotential segments | Standard in NEURON, Brian2 |
| Hines algorithm (1984) | Tridiagonal solve for passive cable | O(N_compartments) per step; numerically stable |
| Operator splitting (active membrane) | Separate passive cable from active channels | Enables Crank-Nicolson for cable + explicit HH gates |

**HH channel equations (Hodgkin & Huxley 1952):**
```
I_Na = g_Na · m³ · h · (V - E_Na)
I_K  = g_K  · n⁴ · (V - E_K)
I_L  = g_L  · (V - E_L)
```
Gate dynamics: dm/dt = α_m(V)(1-m) - β_m(V)m, etc.

### Gap Junction (Electrical Synapse) Modeling

| Method | Purpose | Notes |
|--------|---------|-------|
| Ohmic coupling (linear) | I_gap = g_gap (V_i - V_j) | Standard first approximation; adequate for slow dynamics |
| Voltage-gated conductance | g_gap(V) — connexin-dependent | Cx36 (thalamus dominant) has weak voltage gating; linear usually sufficient |
| Hemichannel model | Asymmetric current flow | Only needed for disease/extreme conditions |

**Cx36 connexin** dominates thalamic gap junctions (especially thalamic reticular nucleus). g_gap ~ 0.1–2 nS per junction (literature range).

### Standing Wave Mode Extraction

| Method | Purpose | Notes |
|--------|---------|-------|
| Spatial FFT / DFT | Frequency domain of field pattern | For uniform spatial grids; fast |
| SVD of E-field time series | Dominant spatial modes | Robust; non-periodic geometries; mode = U[:,k] |
| Dynamic Mode Decomposition (DMD) | Spatiotemporal coherent modes | Extracts frequency + growth rate; good for transient dynamics |
| Proper Orthogonal Decomposition (POD) | Energy-ranked spatial basis | Equivalent to SVD on snapshots |

For the thalamic EM field: stack snapshots as E_field[:, t], SVD → left singular vectors are standing wave patterns; singular values → mode energy.

---

## 2. Numerical Methods

### HH Integration Schemes

| Scheme | Order | Stability | Recommended for |
|--------|-------|-----------|-----------------|
| Exponential Euler | 1st | A-stable for linear | Fast, good for gate variables |
| Crank-Nicolson (cable) | 2nd | Unconditionally stable | Passive cable — use always |
| RK4 | 4th | Conditionally stable | High accuracy single-cell validation |
| GEAR/CVODE | Adaptive | Stiff-stable | Very stiff systems (tight coupling) |

**dt constraint:** ≤ 0.025 ms to resolve HH Na⁺ activation. Never > 0.05 ms.

### Gap Junction Network Integration

- **Implicit coupling:** For all-to-all or high-density gap junctions, use implicit matrix solve to avoid instability. Brian2 handles via `Synapses`.
- **Explicit coupling:** Valid when g_gap << g_membrane (weakly coupled regime). Simpler but may be unstable for large g_gap.
- **Split-step:** Separate intra-cell cable dynamics (stiff) from inter-cell coupling (less stiff).

### EEG/LFP Forward Model

Standard: **kernel-based LFP** (Hagen et al. 2016 — LFPy; Lindén et al. 2014):
```
LFP(r, t) = Σ_i (1/4πσ) · I_i(t) / |r - r_i|
```
Where I_i are transmembrane currents from each cable compartment.

For population-level EEG: sum over all N cells, apply spatial smoothing kernel (distance-dependent).

---

## 3. Computational Tools

### Neural Simulation

| Tool | Strengths | Weaknesses | Role in This Project |
|------|-----------|------------|---------------------|
| **Brian2 + Brian2CUDA** | Python-native, HH cable, gap junctions, GPU | Multi-compartment GPU stability [VERIFY] | **Primary** |
| NEURON | Gold standard HH morphology | No GPU ODE integration | Single-cell validation |
| GeNN | Compiled CUDA, fast | Limited multi-compartment | Fallback GPU option |
| PyNN | Abstract interface | Too abstracted for cable | Not recommended |

### EM Field Computation

| Tool | Method | Role |
|------|--------|------|
| **JAX (GPU)** | Dense Green's function tensor G·I | Primary EM kernel (N ≤ 3,000) |
| **pyfmmlib / ExaFMM-t** [VERIFY] | Fast Multipole Method | Large N (> 3,000) |
| FEniCSx | FEM, heterogeneous media | Only if tissue boundaries needed |
| LFPy | Standard LFP kernel (Hagen 2016) | Validation cross-check |

### EEG Analysis and Comparison

| Tool | Purpose |
|------|---------|
| **MNE-Python** | EEG data loading, PSD computation, time-frequency analysis |
| **scipy.signal** | Welch PSD, spectrogram, coherence |
| **fooof / specparam** | Parametric PSD decomposition (aperiodic + peaks) |
| **tensorpac** | Phase-amplitude coupling (if cross-frequency needed) |

---

## 4. EM Field Theory Anchors in This Domain

### Hales (2014) — Endogenous EM Field from HH Cable Currents

Primary method anchor. Computes the endogenous EM field generated by the full distributed current along neural morphology using Maxwell's equations in biological media. Key contributions:
- Treats axonal and dendritic currents as distributed sources (not point dipoles)
- Shows the endogenous field is physically significant at short range within the tissue
- Establishes the volume conductor approach as the appropriate framework

**Action required:** Read paper directly to confirm method details before Phase 1 planning.

### McFadden — CEMI Theory (Conscious Electromagnetic Information)

Johnjoe McFadden's Conscious Electromagnetic Information theory (McFadden 2002a, 2002b; J Conscious Stud):
- Argues the brain's endogenous EM field is causally active in integrating neural information — not epiphenomenal
- EM field coherence across neural populations is the integrating mechanism
- Directly relevant to L1 of the project: the collective EM field is the computational substrate, not just a byproduct
- Key prediction: EM field manipulations should alter cognitive processing — testable but not yet conclusively demonstrated

### Lehar — Harmonic Resonance Theory

Standing wave interference patterns in neural EM fields encode perceptual world model. Predictions:
- Spatial periodicity of percepts matches standing wave wavelengths
- Gestalt completion follows constructive interference geometry
- Phase relationships encode object identity and spatial relations

### Pribram — Holonomic Brain Theory

Precursor to Lehar; posits that dendritic microprocesses (slow wave potentials, not spikes) create holographic interference patterns. Emphasizes distributed storage and Gabor-wavelet-like encoding.

---

## 5. Validation Techniques

| Test | What It Checks | Pass Condition |
|------|---------------|----------------|
| Single-cell HH vs NEURON | Spike timing, waveform | < 0.1 mV RMS deviation at matching parameters |
| LFP kernel vs analytical | Green's function implementation | < 1% error on point-source test case |
| EEG PSD comparison | Frequency band matching | Peaks within known bands; stimulus-discriminable profiles |
| Standing wave mode recovery | Holographic redundancy | > 70% fidelity after 20% cell loss |
| Synchrony threshold sweep | Gap junction coupling strength | Identify g_gap for which coherent oscillation emerges |
| Limiting case (N=1) | Cable + EM field consistency | Matches single-cell Hales result |

---

## 6. Open Method Questions

- [ ] Confirm Hales (2014) volume conductor vs. alternative EM method
- [ ] Determine if Cx36 gap junction voltage gating matters at thalamic firing rates
- [ ] Choose: SVD vs. DMD for standing wave extraction (DMD preferred if transient dynamics matter)
- [ ] Determine EEG benchmark dataset for PSD comparison (specific paper TBD in Phase 1)
- [ ] Validate Brian2CUDA multi-compartment cable stability before committing to it as primary tool

---

_Generated: 2026-03-16 | Confidence: MEDIUM (web search unavailable; versions marked [VERIFY])_
