# Roadmap: Thalamo-Cortical EM Manifold Model

## Overview

This project derives and simulates a three-layer thalamo-cortical electromagnetic computation architecture in which a parametric thalamic syncytium of N gap-junction-coupled HH cable neurons produces a collective local field potential (LFP) whose spatial mode structure holographically encodes 3D sensory inputs. The research path runs from locking the theoretical framework and validating single-cell physics (Phase 1), through building the N=100 coupled syncytium and mapping the gap junction parameter space (Phase 2), to testing stimulus-specific LFP spatial mode encoding and holographic recovery with cortical modulation (Phase 3), and finally characterizing N=1,000 scaling (Phase 4). The decisive empirical tests are: LFP PSD peaks within known thalamic oscillation bands that are statistically distinguishable across 3D sensory stimuli, and holographic recovery fidelity above 70% after 20% cell loss.

## Contract Overview

| Contract Item | Advanced By Phase(s) | Status |
| ------------- | -------------------- | ------ |
| claim-main (EEG-band LFP signatures, stimulus-correlated) | Phase 2, Phase 3 | Planned |
| claim-holographic (standing wave / LFP spatial mode encoding, >70% recovery) | Phase 3 | Planned |
| obs-eeg (LFP PSD per stimulus vs. thalamic oscillation bands) | Phase 3 | Planned |
| obs-field (standing wave / LFP spatial mode maps per stimulus) | Phase 2, Phase 3 | Planned |
| obs-encoding (holographic sensory encoding with cell-loss recovery) | Phase 3 | Planned |
| deliv-sim (parametric simulation: N HH cable cells, gap junctions, LFP kernel, 3D sensory input) | Phase 1, Phase 2, Phase 3 | Planned |
| deliv-eeg-fig (LFP PSD per stimulus overlaid on thalamic oscillation band references) | Phase 3 | Planned |
| deliv-encoding-fig (LFP spatial mode maps per stimulus + holographic recovery) | Phase 3 | Planned |
| Ref-Hales (method anchor: EM field from neural cable currents) | Phase 1 (read + implement) | Planned |
| Ref-Lehar (theory anchor: harmonic resonance, holographic encoding) | Phase 1 (operationalize) | Planned |
| Ref-FlyWire (scale benchmark: architecture reference) | Phase 4 (compare) | Planned |
| Ref-EEG-empirical (benchmark: thalamic oscillation bands) | Phase 1 (identify dataset), Phase 3 (compare) | Planned |
| fp-oscillation (forbidden: oscillations uncorrelated with stimulus) | Phase 2, Phase 3 | Guard |
| fp-2d (forbidden: 2D sensory model as primary representation) | Phase 3 | Guard |
| fp-qualitative (forbidden: qualitative EEG resemblance without frequency-band match) | Phase 3 | Guard |

## Phase Dependencies

| Phase | Depends On | Enables | Critical Path? |
|-------|-----------|---------|:-:|
| 1 - Theory Framework and Single-Cell Validation | — | 2 | Yes |
| 2 - N=100 Syncytium Pilot and Gap Junction Sweep | 1 | 3, 4 | Yes |
| 3 - Stimulus-Specific Spatial Mode Encoding | 2 | — | Yes |
| 4 - N=1,000 Scaling Test | 2 | — | No (parallel with 3) |

**Critical path:** 1 -> 2 -> 3 (3 sequential phases, minimum duration)
**Parallelizable:** Phase 4 can run concurrently with Phase 3 once Phase 2 is complete

## Phases

- [ ] **Phase 1: Theory Framework and Single-Cell Validation** — Lock observable (LFP not scalp EEG), justify quasi-static approximation, operationally define holographic encoding, decide TRN vs. relay cell substrate, implement and validate single HH cable cell with I_T and I_h, validate LFP Green's function kernel
- [ ] **Phase 2: N=100 Syncytium Pilot and Gap Junction Sweep** — Build N=100 gap-junction-coupled syncytium; sweep g_j 0.1–10 nS at >=10 points; identify synchrony threshold g_j^c; extract LFP spatial modes; run uncoupled (g_j=0) control for every condition
- [ ] **Phase 3: Stimulus-Specific Spatial Mode Encoding** — Drive syncytium at Phase 2 operating point with distinct 3D sensory inputs (visual light field + spatial audio); test stimulus discriminability of LFP spatial modes; measure holographic recovery after 20% cell loss; implement cortical column controller interface for L2 topology modulation
- [ ] **Phase 4: N=1,000 Scaling Test** — Confirm N=1,000 is computationally tractable; characterize how LFP mode quality and holographic recovery fidelity scale with N; benchmark against FlyWire architecture model

## Phase Details

---

### Phase 1: Theory Framework and Single-Cell Validation

**Goal:** All theoretical foundations are locked and validated before any multi-cell simulation begins: the target observable is defined as LFP (not scalp EEG), quasi-static validity is confirmed, holographic encoding is operationally defined with a concrete reconstruction operator and fidelity metric, the syncytium cell type is chosen (TRN vs. relay), the HH cable model with I_T and I_h matches published thalamic cell parameters, and the LFP Green's function kernel agrees with the analytical point-source solution to <1%.

**Depends on:** Nothing (entry point)

**Requirements:** DERV-01, DERV-02, DERV-03, DERV-04, SIMU-01, VALD-01, VALD-02

**Contract Coverage:**
- Advances: deliv-sim (single-cell component); Ref-Hales (read, confirm quasi-static method); Ref-Lehar (read, extract operationalizable encoding claim); Ref-EEG-empirical (identify specific thalamic LFP dataset for Phase 3 comparison)
- Deliverables: theory framework document (LFP observable, quasi-static justification, encoding operator, cell type decision); single-cell HH code validated vs. NEURON; validated LFP kernel
- Anchor coverage: Ref-Hales must be read before implementation; Ref-Lehar must be operationalized (reconstruction operator written out) before Phase 2 begins; Ref-EEG-empirical specific dataset identified
- Forbidden proxies: Using Hales framing without confirming quasi-static equivalence; proceeding with "standing wave" terminology without resolving the LFP-spatial-mode vs. EM-radiation-mode distinction; leaving holographic encoding undefined at Phase 1 exit

**Success Criteria** (what must be TRUE when Phase 1 completes):

1. The target observable is locked as LFP (local field potential, 100–1000 µm from syncytium, amplitude in V/m or µV/mm); scalp EEG amplitude comparison is explicitly ruled out for N~100–1,000; a specific thalamic LFP empirical dataset is identified for Phase 3 validation
2. The quasi-static approximation is justified in writing: retardation ratio L/lambda_EM << 10^-9 at f <= 1 kHz for L = 10 mm; Poisson equation ∇·(σ∇φ) = -∇·J_imp is the operative field equation; the phrase "Maxwell-equation treatment" is confirmed consistent with quasi-static (consistent with Hales 2014 primary source)
3. Holographic encoding is operationally defined: the field quantity F(x,t) is specified (LFP spatial mode = dominant left singular vector of E-field snapshot matrix), the reconstruction operator is written out, and the fidelity metric is quantified (Pearson correlation between original and reconstructed spatial mode >= 0.70 after 20% cell loss, compared to matched-N random distributed code baseline)
4. Single HH cable cell (>= 10 compartments, I_Na, I_K, I_T, I_h, I_NaP; Brian2+Brian2CUDA) validated against McCormick & Huguenard (1992): LTS threshold within ±5 mV of ~-65 mV from hyperpolarized baseline; spindle frequency 7–14 Hz; intra-burst rate 100–400 Hz; I_T half-activation within ±2 mV of -57 mV; spike waveform < 0.1 mV RMS deviation vs. NEURON at matching parameters
5. LFP Green's function kernel (JAX-GPU dense tensor; quasi-static; homogeneous σ = 0.33 S/m) produces < 1% relative error at 3 distinct distances on point-source analytical test case (comparison: φ_computed vs. 1/(4πσr)); Brian2CUDA multi-compartment HH cable numerical stability confirmed (dt <= 0.025 ms; or fallback tool selected with justification)

**Backtracking triggers:**
- If Brian2CUDA multi-compartment cable is numerically unstable at required dt: switch to GeNN or custom JAX HH; document before Phase 2
- If Hales (2014) primary source indicates a computational approach incompatible with standard quasi-static LFP: re-examine EM method choice before Phase 2
- If TRN gap junction parameters are found incompatible with target oscillation regime: reopen cell type decision

**Plans:** 3 plans

Plans:

- [ ] 01-01-PLAN.md -- Write THEORY-FRAMEWORK.md: read Hales 2014, lock LFP observable, confirm quasi-static, define holographic encoding SVD operator, decide TRN substrate, identify Contreras 1997 benchmark
- [ ] 01-02-PLAN.md -- Implement and validate Brian2 HH cable cell (I_T, I_h, I_NaP, >= 10 compartments); convergence study; LTS threshold; spindle oscillation; cross-validate vs. NEURON ModelDB 279
- [ ] 01-03-PLAN.md -- Implement JAX-GPU LFP Green's function kernel; point-source accuracy test; LFPy cross-check; Brian2CUDA GPU stability test; compile Phase 1 go/no-go table

---

### Phase 2: N=100 Syncytium Pilot and Gap Junction Sweep

**Goal:** A working N=100 gap-junction-coupled thalamic syncytium is built, the LFP Green's function pipeline is integrated, and the g_j parameter space (0.1–10 nS at >= 10 points) is fully characterized: the synchrony onset threshold g_j^c is identified, LFP spatial modes are extracted via SVD at each coupling regime, absolute LFP amplitude is compared to thalamic literature values, and an uncoupled (g_j = 0) control confirms that coupled-syncytium LFP spatial structure is not a trivial consequence of single-cell oscillatory properties.

**Depends on:** Phase 1 (validated HH cell + LFP kernel)

**Requirements:** SIMU-02, CALC-01, VALD-03

**Contract Coverage:**
- Advances: claim-main (establishes LFP oscillatory behavior with coupling); obs-field (first LFP spatial mode maps); deliv-sim (N=100 coupled syncytium with parametric g_j)
- Deliverables: g_j sweep figure: LFP PSD and synchrony order parameter vs. g_j; SVD spatial mode plots at sub/critical/supercritical g_j; uncoupled control LFP PSD; absolute LFP amplitude in V/m
- Anchor coverage: Ref-Hales (method in use); Ref-EEG-empirical (compare identified LFP oscillation bands against spindle frequency at coupled operation point)
- Forbidden proxies: fp-oscillation (confirming oscillations exist is NOT sufficient — spatial mode structure and synchrony threshold must be characterized); reporting results at a single g_j value without sweep; skipping the uncoupled g_j=0 control

**Success Criteria** (what must be TRUE when Phase 2 completes):

1. Synchrony onset threshold g_j^c is identified: the Kuramoto-like order parameter R(g_j) transitions from R < 0.2 (subcritical) to R > 0.8 (supercritical) within the swept range 0.1–10 nS; the critical coupling value g_j^c is reported with ±1 nS precision
2. LFP spatial modes (SVD of E-field snapshot matrix) are computed at >= 3 representative g_j values spanning subcritical / critical / supercritical regimes; singular value spectra reported; dominant spatial mode patterns plotted with physical units
3. Uncoupled control (g_j = 0, same stimulus) produces LFP spatial mode maps that differ from the coupled result: Pearson correlation between coupled and uncoupled dominant SVD modes < 0.5 for a test stimulus condition (VALD-03 pass criterion)
4. Absolute LFP field amplitude at the operating point is reported in V/m and compared to empirical thalamic LFP amplitude literature (e.g., Linden et al. 2010); order-of-magnitude agreement required; large discrepancy triggers re-examination of cell count or morphological alignment assumptions
5. The operating point (g_j value near or above g_j^c) is identified for use in Phase 3; LFP oscillation frequencies at operating point are within thalamic spindle/alpha/gamma bands (7–100 Hz)

**Backtracking triggers:**
- If no synchrony transition is observed across the full 0.1–10 nS sweep: re-examine cell model parameters, cell count, or heterogeneity — do not proceed to Phase 3 until coherent LFP modes are obtained
- If N=100 wall-clock time exceeds feasibility threshold (> 1 hour per simulated second): rethink EM field computation strategy before committing to Phase 3

**Plans:** TBD (estimated 2–3 plans)

Plans:

- [ ] 02-01: Build N=100 syncytium in Brian2CUDA with parametric gap junction topology; integrate JAX-GPU LFP kernel; run g_j sweep (>= 10 points, 0.1–10 nS); compute synchrony order parameter and LFP PSD at each point
- [ ] 02-02: Extract LFP spatial modes via SVD at sub/critical/supercritical operating points; run uncoupled (g_j=0) control; validate coupled vs. uncoupled mode difference (VALD-03); report absolute LFP amplitude

---

### Phase 3: Stimulus-Specific Spatial Mode Encoding

**Goal:** The N=100 syncytium at the Phase 2 operating point is driven by distinct 3D sensory inputs (at minimum: one visual light field stimulus and one spatial audio stimulus, with genuine 3D spatial field representation); LFP spatial mode structure (SVD dominant modes) is demonstrably stimulus-discriminable with above-chance decoding accuracy; holographic redundancy is confirmed (>= 70% fidelity recovery after 20% cell loss compared to random code baseline); and the cortical column controller interface (mGluR1-mediated gap junction modulation) is implemented and demonstrated to reshape LFP spatial modes.

**Depends on:** Phase 2 (operating point established, uncoupled control complete, LFP pipeline validated)

**Requirements:** SIMU-03, SIMU-04, CALC-03, VALD-04

**Contract Coverage:**
- Advances: claim-main (stimulus-correlated LFP PSD — decisive test); claim-holographic (standing wave / LFP spatial mode encoding with recovery — decisive test); obs-eeg (LFP PSD per stimulus vs. thalamic oscillation bands); obs-field (LFP spatial mode maps per stimulus); obs-encoding (holographic encoding with cell-loss recovery); deliv-eeg-fig; deliv-encoding-fig; deliv-sim (3D sensory input interface + controller interface components)
- Deliverables: LFP PSD figure per stimulus (>= 2 stimuli) overlaid with thalamic oscillation band references; LFP spatial mode map figure per stimulus; holographic recovery fidelity curve vs. fraction of cells removed; decoding accuracy (stimulus classification from mode structure); cortical modulation demonstration (mode reshape under two controller conditions)
- Anchor coverage: Ref-Lehar (encoding structure validated against harmonic resonance theory predictions); Ref-EEG-empirical (PSD peaks compared quantitatively to thalamic oscillation bands from identified dataset); Ref-Hales (EM field computation method in use)
- Forbidden proxies: fp-oscillation (LFP bands must differ between stimuli — identical PSD across stimuli is a fail); fp-2d (sensory inputs must be genuine 3D spatial fields — 2D simplification invalidates obs-encoding); fp-qualitative (PSD comparison must be quantitative with frequency-band matching and p < 0.05 stimulus discriminability, not visual resemblance)

**Success Criteria** (what must be TRUE when Phase 3 completes):

1. LFP PSD is computed under >= 2 distinct 3D sensory stimuli (one visual light field, one spatial audio field, genuine 3D spatial representations); PSD peaks fall within known thalamic oscillation bands (spindle 7–14 Hz, alpha 8–13 Hz, or gamma 30–100 Hz); PSD profiles are statistically distinguishable across stimuli (permutation test p < 0.05 on LFP spectral profiles); this is test-eeg pass condition for claim-main
2. LFP spatial mode (dominant left singular vector of E-field snapshot matrix) is stimulus-specific: linear decoder trained on mode structure achieves above-chance stimulus classification accuracy (>= 70% for a two-class comparison); mode correlation across stimuli < 0.5 (Pearson); this goes beyond frequency content alone (Mode discriminability > uncoupled control baseline, VALD-03-analog)
3. Holographic recovery test (VALD-04) passes: test 3D sensory stimulus encoded as LFP spatial mode; 20% of syncytium cells removed; reconstruction from remaining field achieves Pearson correlation >= 0.70 with original mode, compared to matched-N random distributed code baseline
4. Cortical column controller interface (DERV-04 / SIMU-04: mGluR1-mediated g_j modulation) is implemented and produces measurable LFP spatial mode change under >= 2 distinct controller drive conditions; mode change Pearson correlation difference > 0.2 between controller conditions
5. No forbidden proxy accepted: oscillation-only result without stimulus-correlation (fp-oscillation) triggers investigation and replanning; 2D sensory simplification (fp-2d) is never used as the primary encoding test

**Backtracking triggers:**
- Stop condition: if LFP signatures exist but show no correlation with stimulus identity across >= 3 independent stimulus pairs — the holographic encoding hypothesis requires revision; block Phase 4 until re-examination
- Stop condition: if no oscillatory structure appears at any 3D stimulus — fundamental failure; rethink cell density, coupling, or sensory driving mechanism before continuing
- If holographic recovery fidelity is < 70% at N=100: before accepting failure, test whether N=500 recovers fidelity (pull from Phase 4); fidelity below baseline after N correction constitutes failure of claim-holographic

**Plans:** TBD (estimated 3–4 plans)

Plans:

- [ ] 03-01: Implement 3D visual light field (gsplat) and spatial audio (Ambisonics) input interfaces; couple to syncytium afferent drive; validate 3D field representation (not 2D)
- [ ] 03-02: Implement cortical column controller interface (SIMU-04/DERV-04: mGluR1-like g_j modulation); run LFP spatial mode extraction under >= 2 stimuli and >= 2 controller conditions; compute CALC-03 PSD per stimulus
- [ ] 03-03: Run VALD-04 holographic recovery test; compute decoding accuracy from mode structure; produce deliv-eeg-fig and deliv-encoding-fig

---

### Phase 4: N=1,000 Scaling Test

**Goal:** Computational tractability at N=1,000 is established, and the quality of LFP spatial mode encoding (stimulus discriminability, holographic recovery fidelity) is characterized as a function of N from 100 to 1,000; the FlyWire/Shiu 2024 architecture is compared as a scale benchmark; and the FMM transition threshold is identified if N approaches 3,000.

**Depends on:** Phase 2 (operating point established); Phase 3 results inform what scaling metrics to track

**Requirements:** CALC-02

**Contract Coverage:**
- Advances: deliv-sim (N scaling confirmed); Ref-FlyWire (scale comparison performed)
- Deliverables: N scaling curve: stimulus discriminability and holographic recovery fidelity vs. N (N = 100, 500, 1,000); wall-clock time per simulated second vs. N; minimum N for LFP spatial mode emergence (if different from N=100); FMM transition assessment if N > 3,000 is needed
- Anchor coverage: Ref-FlyWire (read and compare architecture; note LIF-vs-HH cost difference ~4,000x); Ref-Hales (confirm EM kernel scales correctly with N)
- Forbidden proxies: declaring N=100 scaling results as general without testing N >= 500; comparing to FlyWire HH scale without noting LIF vs. HH difference

**Success Criteria** (what must be TRUE when Phase 4 completes):

1. N=1,000 simulation runs to completion within a feasibility threshold (wall-clock time per simulated second <= 120 s on available GPU hardware); N=100 timing is reported as baseline; if N=1,000 is intractable, a hard stop is triggered and an alternative compute strategy is documented before any further scaling claim
2. LFP spatial mode emergence is characterized vs. N: minimum N_min for coherent spatial modes (synchrony order parameter R > 0.5) is identified; holographic recovery fidelity >= 70% is tested at N = 100, 500, 1,000 — trend (improving, flat, or degrading) reported
3. Stimulus discriminability (linear decoding accuracy from LFP spatial modes) is characterized vs. N = 100, 500, 1,000; the N-scaling trend is reported; if fidelity improves significantly with N, this strengthens claim-holographic
4. FlyWire/Shiu 2024 architecture is compared as scale benchmark: HH cable vs. LIF cost difference quantified (~4,000x estimated); realistic feasibility bound for this project stated (N ~ 1,000–10,000 given HH cable cost); ExaFMM-t FMM library assessed if N > 3,000 is needed

**Backtracking triggers:**
- Hard stop / rethink condition: if N=100 wall-clock time in Phase 2 already exceeds 1 s/simulated-second on available hardware — stop and rethink EM computation strategy before Phase 4 begins (may require simplified compartmental model or GPU-optimized custom HH)

**Plans:** TBD (estimated 2 plans)

Plans:

- [ ] 04-01: Run N = 100, 500, 1,000 syncytium simulations at Phase 2 operating point; measure wall-clock time per simulated second; assess FMM need
- [ ] 04-02: Characterize encoding quality (LFP mode emergence, stimulus discriminability, holographic recovery fidelity) as function of N; compare to FlyWire scale benchmark; report CALC-02 results

---

## Risk Register

| Phase | Top Risk | Probability | Impact | Mitigation |
|-------|---------|:-:|:-:|-----------|
| 1 | Brian2CUDA multi-compartment HH cable numerically unstable | MEDIUM | HIGH | Fallback: GeNN or custom JAX HH; validate at Phase 1 exit before committing |
| 1 | Hales 2014 uses incompatible EM method | LOW | MEDIUM | Read primary source early in Phase 1; quasi-static is almost certainly equivalent |
| 2 | No synchrony transition across full g_j sweep | LOW | HIGH | Cell model parameters and heterogeneity revisit; backtracking trigger at Phase 2 midpoint |
| 2 | N=100 computationally intractable | LOW | HIGH | Hard stop; rethink EM computation strategy; dense tensor cost O(N^2) is predictable |
| 3 | LFP spatial modes not stimulus-specific | HIGH | HIGH | This is the central novel claim; if modes are stimulus-independent, encoding hypothesis fails; must distinguish from uncoupled control |
| 3 | Holographic recovery < 70% at N=100 | MEDIUM | MEDIUM | Test N=500 before declaring failure; fidelity may emerge at larger N |
| 4 | N=1,000 HH cable computationally intractable | MEDIUM | MEDIUM | Feasibility estimated; fallback: compartmental simplification at N > 300 |

---

## Progress

**Execution Order:**
Phases 1 -> 2 -> 3 (critical path); Phase 4 parallelizable with Phase 3 after Phase 2 complete

| Phase | Plans Complete | Status | Completed |
| --- | --- | --- | --- |
| 1. Theory Framework and Single-Cell Validation | 0/TBD | Not started | - |
| 2. N=100 Syncytium Pilot and Gap Junction Sweep | 0/TBD | Not started | - |
| 3. Stimulus-Specific Spatial Mode Encoding | 0/TBD | Not started | - |
| 4. N=1,000 Scaling Test | 0/TBD | Not started | - |
