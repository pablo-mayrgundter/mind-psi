# Requirements: Thalamo-Cortical EM Manifold Model

**Defined:** 2026-03-16
**Core Research Question:** Can a Maxwell-equation treatment of the endogenous EM field of a parametric thalamic syncytium (HH cable neurons, gap junction coupling, cortical column controllers) produce EM manifold dynamics that holographically encode 3D sensory inputs and reproduce EEG-like oscillatory signatures correlated with stimulus identity?

## Primary Requirements

### Derivations

- [ ] **DERV-01**: Establish quasi-static LFP field framework — derive Poisson equation from Maxwell for the syncytium geometry; confirm quasi-static validity; define "standing wave" operationally as LFP spatial eigenmode (NOT EM radiation mode)
- [ ] **DERV-02**: Write operational definition of holographic encoding — specify field quantity F(x,t), reconstruction operator, and fidelity metric (≥70% recovery after 20% cell loss constitutes holographic redundancy)
- [ ] **DERV-03**: Derive thalamic relay cell HH model with I_T, I_h, I_NaP (McCormick & Huguenard 1992 parameters, temperature-corrected to 37°C); justify inclusion of each additional current
- [ ] **DERV-04**: Derive cortical column controller coupling law — how efferent cortical signals modulate thalamic membrane dynamics and gap junction conductance (mGluR1 pathway, Landisman & Connors 2005)

### Simulations

- [ ] **SIMU-01**: Implement single HH cable cell (I_T, I_h, I_NaP, multi-compartment; ≥10 compartments) in Brian2; validate against Steriade/McCormick benchmarks before any population work
- [ ] **SIMU-02**: Implement N-cell thalamic syncytium (Brian2+Brian2CUDA) with parametric gap junction topology (g_j tunable); compute LFP field via quasi-static Green's function kernel; N parameterically scalable from 10 to 1,000+
- [ ] **SIMU-03**: Implement 3D light field (visual) and spatial audio field (auditory) input interfaces — genuine 3D spatial fields; virtual point sources acceptable as test stimuli inside the field but must not reduce the sensory model to 2D
- [ ] **SIMU-04**: Implement cortical column controller interface — efferent modulation of syncytium boundary conditions (membrane drive + gap junction conductance modulation via mGluR1-like pathway)

### Calculations

- [ ] **CALC-01**: Gap junction parameter sweep — g_j from 0.1 to 10 nS at ≥10 points; identify synchrony onset threshold; characterize LFP spatial mode structure as a function of coupling regime (sub/critical/super)
- [ ] **CALC-02**: N scaling test at N = 100, 500, 1,000 — establish minimum N for LFP spatial mode emergence and holographic encoding; confirm N~100 is tractable (hard stop if not)
- [ ] **CALC-03**: LFP PSD extraction under ≥2 distinct 3D sensory stimuli (one visual light field, one spatial audio field) — measure stimulus-specific spectral profiles in thalamic LFP bands (spindle 7–14 Hz, alpha 8–13 Hz, gamma 30–100 Hz)

### Validations

- [ ] **VALD-01**: Validate single HH cell against McCormick & Huguenard (1992) benchmarks: LTS threshold ~–65 mV from hyperpolarized base; spindle frequency 7–14 Hz; burst structure 3–8 spikes at 100–400 Hz intra-burst; I_T half-activation ≈ –57 mV
- [ ] **VALD-02**: Validate LFP Green's function kernel against analytical point-source solution in homogeneous medium — relative error < 1%
- [ ] **VALD-03**: Uncoupled control simulation (g_j = 0, same stimulus) — confirm that coupled syncytium shows stimulus-specific LFP spatial mode variation BEYOND uncoupled baseline (decisive: spatial mode structure must differ, not just frequency content)
- [ ] **VALD-04**: Holographic recovery test — encode test 3D sensory stimulus as LFP spatial mode; remove 20% of syncytium cells; reconstruct encoded pattern from remaining field; fidelity ≥ 70% constitutes pass

## Follow-up Requirements

### Extended Analysis

- **EXT-01**: Full motor control loop closure (FlyWire-style neural-to-action) — future milestone
- **EXT-02**: Manifold dimensionality analysis — formal measurement of the dimension of the holographic encoding manifold
- **EXT-03**: Forward model from thalamic LFP to simulated scalp EEG — maps local field to far-field observable
- **EXT-04**: N > 10,000 scaling with FMM EM kernel — requires multi-GPU / HPC resources
- **EXT-05**: CEMI bidirectional feedback — add ephaptic back-action of EM field on neural firing; test whether it amplifies or disrupts holographic encoding

### Meta-Optimization (L3)

- **META-01**: Phase-space configuration optimization — systematic search over syncytium connectivity and cortical controller parameters for optimal holographic encoding properties
- **META-02**: Gap junction topology learning rule — derive update rule for g_j configuration based on encoding fidelity gradient

## Out of Scope

| Topic | Reason |
|-------|--------|
| Scalp EEG amplitude matching | N~100–1,000 thalamic cells physically cannot produce scalp EEG amplitudes (4–6 order of magnitude gap); target is LFP |
| Consciousness mechanism claims | Beyond EM field modeling scope; CEMI is an interpretive framework, not a derivation target |
| Full Drosophila connectome replication | FlyWire is a scale/architecture benchmark, not a direct implementation target for Phase 1 |
| 2D sensory field representations | The core hypothesis requires genuine 3D sensing; 2D shortcuts invalidate the light-field and spatial-audio encoding claims |
| Post-hoc EEG frequency tuning | Tuning parameters to match EEG bands without mechanistic basis is a forbidden proxy |

## Accuracy and Validation Criteria

| Requirement | Accuracy Target | Validation Method |
|-------------|----------------|-------------------|
| VALD-01 | LTS threshold ±5 mV; spindle freq ±1 Hz; burst 3–8 spikes | Direct comparison to McCormick & Huguenard (1992) Table 1/Fig. 4 |
| VALD-02 | Relative LFP error < 1% vs. analytical | Point-source test case at 3 distances; compare Green's function sum vs. 1/(4πσr) |
| VALD-03 | Spatial mode correlation coupled vs uncoupled < 0.5 for different stimuli | Pearson correlation of LFP spatial maps; t-test across stimulus pairs |
| VALD-04 | Recovery fidelity ≥ 70% after 20% cell loss | Correlation between original and reconstructed spatial mode |
| CALC-03 | PSD peaks within ±2 Hz of target band centers; p < 0.05 stimulus discriminability | Welch PSD; permutation test for LFP profile discrimination |

## Contract Coverage

| Requirement | Decisive Output / Deliverable | Anchor / Benchmark | Prior Inputs | False Progress To Reject |
|-------------|------------------------------|-------------------|--------------|--------------------------|
| SIMU-02 + CALC-03 | `deliv-eeg-fig`: LFP PSD per stimulus | Ref-Hales (method); Ref-EEG-empirical (band targets) | VALD-01 output (validated HH cell) | Oscillations present but uncorrelated with stimulus identity |
| SIMU-02 + VALD-04 | `deliv-encoding-fig`: standing wave maps + recovery | Ref-Lehar (encoding theory) | VALD-02 output (validated LFP kernel) | Recovery that matches random distributed code baseline |
| VALD-03 | Control figure: coupled vs uncoupled LFP spatial modes | Ref-EEG-empirical | SIMU-02 output | EEG bands match in uncoupled control → result is trivial |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| DERV-01 | Phase 1: Theory Framework and Single-Cell Validation | Pending |
| DERV-02 | Phase 1: Theory Framework and Single-Cell Validation | Pending |
| DERV-03 | Phase 1: Theory Framework and Single-Cell Validation | Pending |
| DERV-04 | Phase 1: Theory Framework and Single-Cell Validation | Pending |
| SIMU-01 | Phase 1: Theory Framework and Single-Cell Validation | Pending |
| VALD-01 | Phase 1: Theory Framework and Single-Cell Validation | Pending |
| VALD-02 | Phase 1: Theory Framework and Single-Cell Validation | Pending |
| SIMU-02 | Phase 2: N=100 Syncytium Pilot and Gap Junction Sweep | Pending |
| CALC-01 | Phase 2: N=100 Syncytium Pilot and Gap Junction Sweep | Pending |
| VALD-03 | Phase 2: N=100 Syncytium Pilot and Gap Junction Sweep | Pending |
| SIMU-03 | Phase 3: Stimulus-Specific Spatial Mode Encoding | Pending |
| SIMU-04 | Phase 3: Stimulus-Specific Spatial Mode Encoding | Pending |
| CALC-03 | Phase 3: Stimulus-Specific Spatial Mode Encoding | Pending |
| VALD-04 | Phase 3: Stimulus-Specific Spatial Mode Encoding | Pending |
| CALC-02 | Phase 4: N=1,000 Scaling Test | Pending |

**Coverage:**
- Primary requirements: 14 total
- Mapped to phases: 14
- Unmapped: 0

---

_Requirements defined: 2026-03-16_
_Last updated: 2026-03-16 after initial definition_
