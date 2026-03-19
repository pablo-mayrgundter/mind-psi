# Research State

## Project Reference

See: .gpd/PROJECT.md

**Core research question:** Can Maxwell-equation treatment of endogenous EM field of parametric thalamic TRN syncytium (HH cable, Cx36 gap junctions, cortical controllers) produce EM manifold dynamics that holographically encode 3D sensory inputs and reproduce LFP oscillatory signatures correlated with stimulus identity?
**Current focus:** Phase 1 complete (3 open items); deciding Phase 2 entry or gap-closure first

## Current Position

**Current Phase:** 1
**Current Phase Name:** Theory Framework and Single-Cell Validation
**Total Phases:** 4
**Current Plan:** — (all 3 plans complete)
**Total Plans in Phase:** 3
**Status:** Complete with open items (GO-06 FAIL, GO-08/GO-11/GO-12 PENDING)
**Last Activity:** 2026-03-17

**Progress:** [██░░░░░░░░] 25%

## Active Calculations

None active.

## Intermediate Results

### Phase 1 Go/No-Go Table (2026-03-17)

| Criterion | Status | Value | Target |
|---|---|---|---|
| GO-01 dt convergence | **PASS** | 0.035 ms | < 0.1 ms |
| GO-02 spatial convergence | **PASS** | 1.86% | < 5% |
| GO-03 V_LTS threshold | **PASS** | −66.25 mV | [−70, −60] mV |
| GO-04 I_T half-activation | **PASS** | −56.98 mV | [−59, −55] mV |
| GO-05 f_spindle | **PASS** | 10.00 Hz | [7, 14] Hz |
| GO-06 intra-burst frequency | **FAIL** | single AP | [100, 400] Hz |
| GO-07 spike timing (dt proxy) | **PASS** | 0.040 ms | < 0.1 ms |
| GO-08 NEURON waveform RMS | **PENDING** | unavailable | < 0.1 mV |
| GO-09 LFP sign check | **PASS** | phi > 0 | positive |
| GO-10 LFP accuracy | **PASS** | 0.00e+00 | < 1% |
| GO-11 Brian2CUDA spike count | **PENDING** | not installed | exact match |
| GO-12 Brian2CUDA spike timing | **PENDING** | not installed | < 0.1 ms |
| GO-13 gap junction zero-current | **PASS** | 0.000e+00 A | < 1e-15 A |

### Key Phase 1 Results (locked)

- **Quasi-static ratio**: L/λ_EM = 2.98×10⁻⁷ at f=1kHz, L=10mm — confirmed << 1; full Maxwell adds nothing
- **TRN substrate**: Locked (Landisman 2002 Cx36 Cx36 gap junctions, g_j = 0.1–2 nS)
- **Holographic encoding operator**: Φ = U·S·Vᵀ; H = U[:,0]; ρ ≥ 0.70 criterion; 50-replicate Monte Carlo baseline
- **Phase 3 benchmark**: Contreras et al. (1997) J. Neurosci. 17:1179; Fig. 4; 7–14 Hz spindle band
- **HH cable**: Brian2 SpatialNeuron, 10 compartments, 500 µm, I_Na/K/L/T/h/NaP, Q10→37°C (T_ref=24°C)
- **LFP kernel**: φ = (1/4πσ)·Σ I_m,j/|r−rⱼ|; σ=0.33 S/m; float64; validated to machine precision
- **Spindle**: 10 Hz (forced IPSP protocol); I_h dependence confirmed (g_h=0 abolishes)
- **LTS**: V_LTS = −66.25 mV; I_T h-gate recovers fully at −90 mV (tau_h ≈ 87 ms)
- **V_rest**: −72 mV (E_L = −70 mV effect); AP peak: ~27 mV (T_ref_NaK = 24°C)

## Open Questions

- **GO-06**: Multi-spike LTS burst requires I_A (A-type K+ current, Huguenard & Prince 1992) — standard HH K+ delayed rectifier cannot inactivate fast enough; see SUMMARY 01-02 Deviation 3
- **GO-08**: NEURON ModelDB 279 (modeldb.yale.edu) timed out; need to retry or use local mirror
- **GO-11/GO-12**: Brian2CUDA not installed on this machine; need GPU-equipped environment
- Autonomous spindle impossible in isolated single cell; requires TC-RE network (Phase 2 design)
- Bridging from single-cell LFP to collective N=100 syncytium LFP spatial modes (Phase 2)

## Performance Metrics

| Label | Duration | Tasks | Files |
| ----- | -------- | ----- | ----- |
| -     | -        | -     | -     |

## Accumulated Context

### Decisions

- **2026-03-17**: TRN substrate locked (over relay cells) — Landisman 2002 Cx36 exclusively in TRN
- **2026-03-17**: T_ref_NaK = 24°C (mammalian, McCormick & Huguenard 1992) — NOT 6.3°C (HH squid); affects AP peak voltage (~27 mV vs +40 mV target; acceptable)
- **2026-03-17**: tau_m_h formula: exp((V+467)/66.6)/tadj_T for V<−75 mV; 28+exp(−(V+22)/10.5) for V≥−75 mV
- **2026-03-17**: LTS detection: inflection-point method (d²V/dt²=0 while V∈[−80,−55] mV, dV/dt>0) NOT dV/dt>10 mV/ms
- **2026-03-17**: Spindle forcing protocol: 10 Hz periodic IPSP (autonomous spindle requires TC-RE network)
- **2026-03-17**: Phase 2 architecture: Brian2 CPU → save I_m(t) → JAX CPU LFP kernel (fallback B); Brian2CUDA if available

### Active Approximations

- Quasi-static Poisson (never breaks at neural scales — L/λ_EM = 3e-7)
- Homogeneous isotropic tissue σ=0.33 S/m (factor-2 amplitude uncertainty; no skull boundaries)
- Point-source current per compartment at 50 µm length (error < 2% for r > 100 µm)
- Fixed gap junction conductance g_j (no voltage gating; valid for g_j < 1 nS per Landisman 2002)

**Convention Lock:**

- Metric signature: N/A — biophysics project (computational neuroscience, not QFT)
- Fourier convention: Physics asymmetric: f_tilde(omega) = integral f(t) exp(-i*omega*t) dt; inverse: f(t) = integral [d_omega/(2*pi)] f_tilde(omega) exp(+i*omega*t); consistent with scipy.signal and MNE-Python
- Natural units: N/A — biophysics project (computational neuroscience, not QFT)
- Gauge choice: N/A — biophysics project (computational neuroscience, not QFT)
- Regularization scheme: N/A — biophysics project (computational neuroscience, not QFT)
- Renormalization scheme: N/A — biophysics project (computational neuroscience, not QFT)
- Coordinate system: N/A — biophysics project (computational neuroscience, not QFT)
- Spin basis: N/A — biophysics project (computational neuroscience, not QFT)
- State normalization: N/A — biophysics project (computational neuroscience, not QFT)
- Coupling convention: Gap junction: I_gap_i = g_j*(V_i-V_j); outward-positive from cell i; g_j in nS range 0.1-10; K = g_j/(g_j+g_m) dimensionless
- Index positioning: N/A — biophysics project (computational neuroscience, not QFT)
- Time ordering: N/A — biophysics project (computational neuroscience, not QFT)
- Commutation convention: N/A — biophysics project (computational neuroscience, not QFT)
- Levi-Civita sign: N/A — biophysics project (computational neuroscience, not QFT)
- Generator normalization: N/A — biophysics project (computational neuroscience, not QFT)
- Covariant derivative sign: N/A — biophysics project (computational neuroscience, not QFT)
- Gamma matrix convention: N/A — biophysics project (computational neuroscience, not QFT)
- Creation/annihilation order: N/A — biophysics project (computational neuroscience, not QFT)

*Custom conventions:*
- Observable: LFP (local field potential), NOT scalp EEG; amplitude in V/m or uV; compare to LFP literature (Linden et al. 2010)
- Unit System: SI for EM (V/m, A, S/m); neural units (mV, ms, nS, uF/cm2, uA/cm2, um); NO CGS; conversion: 1 uV/cm = 0.1 mV/m
- Hh Current Sign: outward_positive; I_ion = g*(V_m-E_ion); h = fraction NOT inactivated; h_inf(-65mV)=0.596; C_m dV/dt = -I_Na-I_K-I_L-I_T-I_h-I_NaP+I_ext
- Membrane Potential Sign: V_m = V_intracellular - V_extracellular; resting = -65 mV; AP peak = +40 mV; AHP = -75 mV
- Em Approximation: quasi-static Poisson: ∇·(σ∇φ) = -∇·J_imp; σ=0.33 S/m; retardation ratio L/lambda ~ 3e-8 at 1kHz and 1cm; full Maxwell adds nothing at neural scales
- Standing Wave Definition: LFP spatial eigenmode of network dynamics (NOT EM radiation mode); dominant left singular vector U[:,0] of LFP snapshot matrix; spatial period set by network architecture not EM wavelength
- Holographic Fidelity: rho = Pearson(F_original, F_reconstructed) >= 0.70 after 20% cell loss; F = U[:,0] dominant SVD mode; compare to matched-N random distributed code baseline
- Kuramoto Order Parameter: R = (1/N)*|sum_k exp(i*theta_k)|; subcritical R<0.2; supercritical R>0.8; g_j^c = coupling threshold at max dR/dg_j
- Simulation Timestep: dt <= 0.025 ms; never exceed 0.05 ms; Crank-Nicolson for cable; backward/exponential Euler for HH gates; initialize gates to steady-state at V=-65mV; 100ms settling before recording

### Propagated Uncertainties

None yet.

### Pending Todos

- [ ] Resolve GO-06: add I_A current (Huguenard & Prince 1992) to enable multi-spike LTS burst (Phase 1 gap closure or Phase 2 prerequisite)
- [ ] Retry ModelDB 279 download for NEURON cross-validation (GO-08)
- [ ] Install Brian2CUDA on GPU machine for GO-11/GO-12 before Phase 2 production runs

### Blockers/Concerns

- GO-06 FAIL: multi-spike intra-burst frequency requires A-type K+ current; single AP on LTS rebound. Hard-block per experiment design — Phase 2 advancement technically blocked. However, LTS mechanism and spindle frequency (GO-05) ARE confirmed; Phase 2 design can proceed with the understanding that burst structure will be improved.
- GO-08/GO-11/GO-12 PENDING: external dependencies (NEURON, GPU) unavailable on current machine.

## Session Continuity

**Last session:** 2026-03-17
**Stopped at:** Phase 1 Wave 1 complete; 3 plans have SUMMARY.md; 3 open items (GO-06 FAIL, GO-08/GO-11/GO-12 PENDING)
**Resume file:** —
