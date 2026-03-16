# Research Summary

**Project:** Thalamo-Cortical EM Manifold Model
**Domain:** Computational neuroscience / biological electrodynamics
**Researched:** 2026-03-16
**Confidence:** MEDIUM

---

## Unified Notation

The four research files are internally consistent with each other and with the standard literature. One substantive convention conflict was identified and resolved (see below). The following table is binding for all downstream work.

| Symbol | Quantity | Units | Convention Notes |
|--------|----------|-------|-----------------|
| V_m | Membrane potential | mV | V_intracellular - V_extracellular; resting ≈ -65 mV |
| E_Na, E_K, E_L | Ionic reversal potentials | mV | Nernst; E_Na ≈ +55, E_K ≈ -90, E_L ≈ -70 |
| I_Na, I_K, I_L | Ionic membrane currents | µA/cm² | Outward-positive (HH standard); I_Na = g_Na·m³·h·(V_m - E_Na) |
| m, h, n | HH gating variables | dimensionless | h = fraction NOT inactivated; standard HH sign |
| I_T | T-type Ca²⁺ current | µA/cm² | Thalamic literature convention |
| I_h | HCN / "h" current | µA/cm² | Thalamic literature convention |
| I_NaP | Persistent sodium current | µA/cm² | Thalamic literature convention |
| g_T, g_h | Maximal conductances | mS/cm² | g_T ≈ 0.5–2; g_h ≈ 0.05–0.2 |
| λ | Electrotonic space constant | µm | λ = √(d·R_m / (4R_i)); Rall |
| τ_m | Membrane time constant | ms | τ_m = R_m·C_m |
| C_m | Specific membrane capacitance | µF/cm² | C_m ≈ 1 µF/cm² |
| J_imp | Impressed current density | A/m² | Nunez & Srinivasan volume conductor convention |
| φ | Extracellular potential | V (or µV) | Quasi-static; φ = (1/4πσ) ∫ J·∇'(1/|r-r'|) dV' |
| σ | Tissue electrical conductivity | S/m | σ ≈ 0.33 S/m (gray matter) |
| g_j | Gap junction conductance | nS | Per junction; Cx36-dominated; range 0.1–10 nS |
| K | Gap junction coupling coefficient | dimensionless | K = g_j / (g_j + g_membrane); range 0.01–0.5 |
| k_LFP | LFP spatial mode wavenumber | m⁻¹ | Spatial eigenmode of network dynamics field |
| f | Neural oscillation frequency | Hz | IFCN bands: delta 0.5–4, theta 4–8, alpha 8–13, beta 13–30, gamma 30–100 |
| dt | Simulation timestep | ms | dt ≤ 0.025 ms required; never > 0.05 ms |
| N | Syncytium cell count | dimensionless | Parametric: target N = 100 for Phase 1 |

**Notation conflict resolved — "standing waves":** The phrase "standing wave" appears in both the Lehar theory anchors and the EM field literature with two incompatible meanings. In free-space EM theory, a standing wave at 10–80 Hz has wavelength ~3,750–37,500 km — no EM radiation standing wave is possible at thalamic scales. In this project, "standing wave" means a spatial eigenmode of the population-level LFP field (a slow electromechanical network mode, not an EM radiation mode). All downstream documents must use "LFP spatial mode" or "network field mode" when the meaning is the population dynamics pattern, and reserve "electromagnetic" for Maxwell-equation quantities. These are related — the LFP field is computed from Maxwell's equations applied to the neural currents — but the resonance that produces standing patterns is in the network dynamics, not EM radiation propagation.

---

## Executive Summary

This project sits at the intersection of three well-established but previously uncombined frameworks: Hodgkin-Huxley cable neuroscience (mature), Maxwell-equation volume-conductor field modeling (mature), and Lehar harmonic resonance theory (conceptually rich but formally underspecified). The core novelty is extending Hales' (2014) single-cell endogenous EM field treatment to a gap-junction coupled syncytium of N cells, and asking whether the collective LFP-like field organizes into spatial modes that holographically encode 3D sensory inputs. The foundational physics (quasi-static Maxwell + HH cable) is well understood; the novel claim is the collective behavior.

The most important finding from the literature survey is a required reframing of the primary observable. The project's original language refers to "EEG-like signatures," but a thalamic syncytium of N~100 cells cannot produce scalp EEG (which requires ~10,000–100,000 synchronously firing pyramidal neurons over cm² patches). The correct and physically appropriate observable for this system is the **local field potential (LFP)**, recorded 100–1000 µm from the syncytium. LFP from N~100–1,000 coupled cable neurons is computationally tractable, physically meaningful, and empirically well-characterized in thalamic recordings. All downstream phases must target LFP, not scalp EEG. If EEG comparison is later desired, a full forward model projection must be added and only spectral profiles (not amplitudes) compared.

The recommended computational approach is: Brian2+Brian2CUDA for HH cable simulation with gap junctions; JAX-GPU dense Green's function kernel for quasi-static LFP computation at N ≤ 3,000 (O(N²) per timestep, trivial for target scale); FMM library (ExaFMM-t or equivalent) if N > 3,000. Validation proceeds single-cell first (NEURON cross-check), then N=100 pilot, then gap junction parameter sweep, then sensory encoding. The chain of dependencies is strict: single-cell HH fidelity must be established before the EM pipeline is built, and the EM pipeline must be validated before encoding claims are made. There are three critical pitfalls that must be addressed in Phase 1 theory work before any simulation: the EEG-amplitude scale gap (use LFP), the quasi-static validity statement (justify explicitly), and the operational definition of holographic encoding (define the reconstruction operator before Phase 2 begins).

---

## Key Findings

### Computational Approaches

The simulation stack has a clear and well-justified architecture. Brian2 (Stimberg et al. 2019) with the Brian2CUDA GPU extension is the primary neural simulation tool, supporting multi-compartment HH cable models and gap junction electrical synapses natively. NEURON is used only for single-cell validation (superior morphological fidelity, but no GPU ODE integration path). The EM field computation uses the quasi-static volume conductor Green's function (quasi-static is valid at all neural frequencies — retardation correction is ~10⁻¹¹ at f = 1 kHz, L = 10 mm). At N ≤ 3,000, a precomputed dense N_src × N_obs Green's function tensor **G** allows each timestep to reduce to a single GPU matrix-vector multiply φ = **G**·**I_m**, at cost O(N²) per step; for N ≤ 1,000 this fits in < 4 GB VRAM. Above N ≈ 3,000, Fast Multipole Method (ExaFMM-t) is required.

Resource estimates from domain knowledge (not benchmarks; must be validated in Phase 1): N = 100 ≈ 1 s wall time on A100; N = 1,000 ≈ 20–30 s; N = 10,000 ≈ 500–1000 s requiring multi-GPU or HPC. The critical feasibility threshold is N = 100, and Phase 1 must demonstrate tractability before committing to larger sweeps.

The FlyWire/Shiu 2024 scale benchmark (140,000 neurons, 50M synapses) used simplified LIF neurons — not HH cable. HH cable is approximately 4,000× more expensive per simulated second than LIF (combined: slower timestep × more state variables × higher per-step cost). FlyWire validates data pipeline architecture, not HH cable scalability. The realistic feasibility bound for this project is N ~ 1,000–10,000.

**Core stack:**

- Neural dynamics: Brian2 + Brian2CUDA (HH cable, gap junctions, dt = 0.025 ms)
- EM field kernel: JAX-GPU Green's function tensor (N ≤ 3,000) / ExaFMM-t (N > 3,000)
- Validation: NEURON (single-cell), LFPy (LFP kernel cross-check)
- Analysis: MNE-Python / scipy.signal (PSD), SVD or DMD (spatial mode extraction)
- Sensory input: 3D Gaussian Splatting (gsplat) for visual light field; Ambisonics + scipy for spatial audio

### Prior Work Landscape

The foundational biology is HIGH confidence: Hodgkin & Huxley (1952) HH equations; Rall (1969) cable theory; Steriade et al. (1993) thalamo-cortical spindle mechanism; McCormick & Huguenard (1992) thalamic relay cell parameters; Nunez & Srinivasan (2006) quasi-static neural EM. Standard HH with only Na/K channels cannot reproduce thalamic EEG-band oscillations — I_T (T-type Ca²⁺) and I_h (HCN) are mandatory additions for burst firing and spindle generation. This is established physiology, not an approximation.

Gap junctions in thalamus require care. The best-characterized gap junction coupling in thalamus is in the **thalamic reticular nucleus (TRN)** via Cx36 (Landisman et al. 2002: g_j = 0.1–2 nS per junction, coupling coefficient 0.01–0.1, range < 50 µm). Gap junction coupling in excitatory relay cells is much weaker or absent in most specific thalamic nuclei. **The syncytium model may need to be TRN-based rather than relay-cell-based.** This is a major unresolved design decision that must be addressed in Phase 1. Landisman & Connors (2005) show TRN gap junction conductance is dynamically modulated by cortical efferents (mGluR1 pathway → increased coupling; dopamine → decreased coupling), providing a biophysical mechanism for the L2 "topology switching" concept.

The theoretical anchors (Hales 2014, Lehar 2003, McFadden 2002) are MEDIUM confidence from training data and require primary source verification. Hales (2014) computes the endogenous EM field from single-cell / small-population HH cable currents using Maxwell's equations — the natural method for this project. The critical gap is that Hales does not address gap-junction-coupled populations; extending to the N-cell syncytium is the core novel derivation. The paper must be read before Phase 1 planning. Lehar (2003) provides the standing wave / holographic encoding theoretical framework but has no quantitative predictions for spatial frequency, wavelength, or amplitude in brain tissue — these must be derived from first principles. McFadden (2002) CEMI theory argues the endogenous EM field is causally active (not merely epiphenomenal), supporting a bidirectional EM-neural feedback loop; however, the amplitude threshold for EM field influence on firing is contested and must be quantified (ephaptic coupling ~0.1–1 mV — likely sub-threshold for significant spike modulation at LFP field strengths).

**Must reproduce (benchmarks):**
- Single-cell HH spike waveform: < 0.1 mV RMS deviation vs. NEURON at matching parameters
- TRN spindle oscillation 7–14 Hz from I_T/I_h thalamic dynamics (Steriade et al. 1993)
- LFP amplitude in V/m or µV/mm in literature-appropriate range (compare Linden et al. 2010)
- Green's function kernel: < 1% error on point-source test case

**Novel predictions (contributions):**
- Collective LFP spatial mode structure from N coupled HH cable neurons
- Stimulus-specific spatial mode variation as function of 3D sensory input
- Gap junction coupling threshold for coherent oscillation onset
- Holographic recovery after cell loss (if operational definition satisfied)

**Defer (future work):**
- Full bidirectional CEMI feedback (EM field back-modulating HH dynamics) — requires quantifying ephaptic amplitude first
- Manifold dimensionality analysis
- Motor loop closure

### Methods and Tools

The methods ecosystem is well-matched to the project. The quasi-static volume conductor approach (not full-wave Maxwell / FDTD) is universally recommended for neural EM below 10 kHz and reduces the Maxwell computation to a Poisson solve. Spatial mode extraction should use SVD on E-field snapshot matrices (left singular vectors = standing wave patterns, singular values = mode energy) or Dynamic Mode Decomposition (DMD) if transient dynamics and frequency information are needed simultaneously. The "Hales method" used as a project anchor is consistent with the standard volume conductor approach; the specific contribution is the distributed morphological current treatment (not point-dipole approximation). Brian2CUDA multi-compartment cable stability must be validated in Phase 1 before committing as primary tool.

**Major components:**

1. HH cable with I_T, I_h (Brian2; validated vs. NEURON) — L1 neural dynamics
2. Quasi-static Green's function EM kernel (JAX-GPU tensor) — L1 field computation
3. Gap junction Ohmic coupling I_gap = g_j(V_i - V_j) (Brian2 Synapses) — L2 topology
4. SVD / DMD mode extraction (numpy/scipy) — spatial mode analysis
5. LFP PSD computation (scipy.signal Welch; MNE-Python) — validation observable
6. 3DGS visual + Ambisonics audio (gsplat, sounddevice+scipy) — sensory input layer

### Critical Pitfalls

1. **EEG amplitude scale gap — use LFP, not scalp EEG** (C1, C+M5). N~100 thalamic cells cannot produce scalp EEG. The target observable must be LFP (local field potential, 100–1000 µm from syncytium). Report field amplitude in absolute units (V/m or µV/mm). This must be locked in Phase 1 before any simulation. EEG frequency bands can still be used as labeling conventions for identified oscillation frequencies, but amplitude comparison must be against LFP literature, not scalp EEG.

2. **"Standing waves" are network LFP modes, not EM radiation modes** (C3). At neural scales (f = 100 Hz, L = 1 cm), L/λ_EM ~ 3×10⁻⁹. The quasi-static approximation reduces Maxwell to Poisson's equation, and "standing wave patterns" in the field are spatial eigenmodes of the network dynamics — not electromagnetic wave resonances. Include one-line quasi-static justification in every document that uses Maxwell's equations. Confirm quasi-static and full-Maxwell solvers agree (they must to 9 significant figures).

3. **Holographic encoding requires an operational definition before Phase 2** (C2). Lehar's theory is analogical, not mechanistic. Before Phase 2 simulations begin, define: (a) what scalar/vector field quantity F(x,t) constitutes the hologram; (b) the reconstruction operator; (c) the fidelity metric; (d) the baseline (matched-N random distributed code). The standing wave spatial mode structure is the correct operational candidate — stimulus identity decoded from the dominant left singular vector of the E-field snapshot matrix.

4. **Gap junction parameter sweep is mandatory — not optional** (C4). g_j spans two orders of magnitude in the literature (0.1–10 nS), with the synchrony phase transition inside this range. Any result reported at a single g_j is incomplete. Phase 2 must include a sweep across at least one decade of g_j, identifying the coupling regime (subcritical / critical / supercritical) at each value.

5. **Uncoupled-cell control is mandatory** (M2). HH neurons fire at 10–80 Hz by tuning bias current. N uncoupled cells produce LFP power at EEG-band frequencies without any coupling or EM field physics. The decisive test is stimulus-specific SPATIAL MODE VARIATION — not frequency content. Run g_j = 0 control for every stimulus condition.

---

## Approximation Landscape

| Method | Valid Regime | Breaks Down When | Controlled? | Complements |
|--------|-------------|-----------------|-------------|-------------|
| Quasi-static Maxwell (Poisson) | f < 10 kHz, any neural scale | f > 100 MHz (never in this project) | Yes — retardation ratio L/λ | Full-wave FDTD (never needed here) |
| Homogeneous isotropic tissue (σ = 0.33 S/m) | Syncytium-only, no skull/CSF | Cortical projection or EEG comparison | No — factor 2–3 error bound | FEM (FEniCSx, SimNIBS) for tissue boundaries |
| Dense Green's function tensor | N ≤ ~3,000 | N > 3,000 (VRAM wall ~4 GB float32) | Yes — O(N²) per step | FMM (ExaFMM-t) |
| Fast Multipole Method (FMM) | N > 3,000 | Not applicable at thalamic scales | Yes — controlled error | Dense tensor (N ≤ 3,000) |
| HH cable model (I_Na, I_K, I_T, I_h) | Single cell to small population | Does not scale to N > 10,000 without HPC | Yes — full ionic current | Compartmental simplification (future) |
| Ohmic gap junction (linear g_j) | g_j < 1 nS, coupling coeff < 0.1 | Large transjunctional voltages (> 30 mV) | Approximate | Voltage-gated Cx36 model |
| Standard HH (Na/K only) | Squid axon | Thalamic burst / spindle physics | N/A | HH + I_T + I_h (mandatory for thalamus) |
| Point-neuron (LIF or rate model) | Large-N exploration only | Any morphology-dependent EM field claim | No — breaks Hales method | HH cable (required for L1 EM field) |
| Crank-Nicolson cable solver | Passive cable | Active membrane with very fast gates | 2nd order unconditionally stable | Operator splitting (active membrane) |
| SVD mode extraction | Stationary modes | Transient dynamics (e.g., spindle onset) | Yes — energy ranked | DMD (frequency + growth rate) |

**Coverage gaps:** No reliable analytical approximation exists for the synchrony onset in a gap-junction coupled HH network as a function of N, g_j, and cell heterogeneity simultaneously. This transition region requires direct numerical simulation. Similarly, the spatial mode structure of the LFP at intermediate g_j (near the coupling transition) has no analytical prediction — it must be extracted computationally.

---

## Theoretical Connections

### 1. Established: HH cable currents → quasi-static EM field (Hales 2014, Nunez 2006, Linden 2010)

The transmembrane current distribution I(x,t) along the cable morphology is the source term in Poisson's equation: ∇·(σ∇φ) = -∇·J_imp. This is a well-established many-to-one mapping. Cable geometry (orientation, branching) determines whether dendritic contributions to the LFP add coherently or cancel. Linden et al. (2010) show that randomly oriented populations have near-complete LFP cancellation — morphological alignment is required for observable LFP. TRN cells are relatively homogeneous in orientation (lamellar structure), which is favorable.

### 2. Established: Thalamo-cortical spindle mechanism (Steriade 1993)

The I_T / I_h TRN-relay loop produces 7–14 Hz spindles via a well-characterized biophysical mechanism. This is the L1 → LFP oscillation link. Spindle frequency is intrinsic to thalamic circuitry, not just HH firing rate, and requires both currents to be present in the model.

### 3. Conjectured: Gap junction topology (L2) → LFP spatial mode structure

Cortical modulation of TRN gap junctions (Landisman & Connors 2005, mGluR1 pathway) changes the effective network topology of the syncytium, which changes the eigenmodes of the LFP field. This is the mechanistic basis for "topology switching" and "folding the manifold." It has not been demonstrated computationally for LFP modes — it is the central novel claim of L2.

### 4. Conjectured: Stimulus-specific LFP modes → holographic encoding (Lehar 2003)

If 3D sensory inputs drive different TRN activity patterns, which drive different gap junction modulation, which produces different LFP spatial modes, then the mode structure encodes the stimulus. The chain of physical causation is plausible but unverified. The "holographic" property (resistance to cell loss) requires that the mode structure is distributed across many cells, not concentrated in a few — this is a testable structural property of the SVD decomposition.

### 5. Speculative: CEMI bidirectional feedback (McFadden 2002)

If the LFP field amplitude (computed via the Green's function) is large enough to modulate voltage-gated channels (ephaptic coupling ~0.1–1 mV), then the system has L1 → neural → L1 feedback. This would make the EM manifold causally active in shaping the dynamics that generate it. Current estimates suggest the ephaptic amplitude is well below the spike threshold (~20 mV above resting), making this effect likely sub-dominant in the first-order model. Quantify in Phase 1 and treat as a future extension unless Phase 1 numbers suggest otherwise.

### 6. Structural: Neural network synchrony and Kuramoto order parameter

The gap junction coupled HH network has a synchrony phase transition as a function of g_j, analogous to the Kuramoto model for phase oscillators. The critical coupling g_j^c depends on the spread of natural frequencies (cell heterogeneity) and N. This is an established result in coupled oscillator theory and provides a theoretical expectation for where the coherent LFP mode should emerge. The Kuramoto analogy is structural (not exact — HH is not a phase oscillator) but gives useful order-of-magnitude guidance.

---

## Implications for Research Plan

### Phase 1: Theory Framework and Single-Cell Validation

**Rationale:** Three Phase 1 tasks must be completed before any N > 1 simulation is meaningful. First, the theory framework must be locked: quasi-static validity justified, observable defined as LFP (not EEG), holographic encoding operationally defined (reconstruction operator + fidelity metric), and TRN vs. relay cell choice made. Second, the HH cable model with I_T and I_h must be implemented and validated against NEURON at the single-cell level. Third, the EM pipeline (Green's function kernel) must be validated against a point-source analytical solution. All three tasks are prerequisites for Phase 2.

**Delivers:** (a) Theory document specifying operational definitions; (b) Single-cell HH model with I_T, I_h matching Steriade / McCormick & Huguenard parameters; (c) Validated Green's function LFP kernel; (d) Ephaptic coupling amplitude estimate; (e) Decision: TRN-based or relay-cell-based syncytium.

**Validates:** HH spike waveform < 0.1 mV RMS vs. NEURON; LFP kernel < 1% error on point-source test.

**Avoids:** C1 (LFP target locked), C3 (quasi-static stated), C2 (operational definition written), m1 (HH initialization), M1 (ODE stiffness tested), M3 (tissue homogeneity bounded), M5 (observable locked).

**Needs research:** Read Hales (2014) primary source before starting — confirm method details and whether relay or TRN cells are the appropriate syncytium substrate.

### Phase 2: N=100 Syncytium Pilot and Gap Junction Sweep

**Rationale:** With single-cell and EM kernel validated, build the N=100 gap-junction coupled syncytium. The gap junction conductance g_j must be swept across at least one decade (0.1–10 nS). Run both coupled and uncoupled (g_j = 0) controls for every condition. This phase establishes whether coherent LFP oscillations emerge, identifies the synchrony threshold g_j^c, and provides the baseline LFP amplitude in physical units.

**Delivers:** (a) g_j sweep: LFP PSD vs. g_j; (b) Synchrony order parameter vs. g_j (identify sub/critical/supercritical regime); (c) LFP spatial mode structure (SVD) at each g_j; (d) Uncoupled control LFP PSD; (e) Absolute LFP amplitude in V/m compared to literature.

**Uses:** Brian2CUDA + JAX Green's function kernel; scipy.signal Welch PSD; numpy SVD.

**Builds on:** Phase 1 validated components.

**Avoids:** C4 (g_j sweep), M2 (uncoupled control), m2 (fixed g_j documented).

### Phase 3: Sensory Encoding — Stimulus-Specific Spatial Modes

**Rationale:** With a functional syncytium at the operating point identified in Phase 2, drive the model with distinct 3D sensory inputs (visual light field, spatial audio) and test whether the LFP spatial mode structure (dominant SVD modes) is stimulus-specific. This is the primary scientific claim. Spatial discriminability of modes is the correct test — frequency content alone is insufficient (see pitfall M2).

**Delivers:** (a) LFP spatial mode patterns per stimulus (SVD maps); (b) Stimulus discriminability metric (decoding accuracy from mode structure); (c) PSD profiles per stimulus (frequency comparison against known bands); (d) Holographic recovery: fidelity vs. fraction of cells removed, compared to matched-N random code baseline.

**Uses:** 3DGS (gsplat) for visual; Ambisonics for audio; SVD mode extraction; decoding classifier (simple linear, then SVM or logistic regression).

**Builds on:** Phase 2 syncytium at selected g_j; Phase 1 operational definition of holographic encoding.

**Avoids:** C2 (operational definition now implemented), M4 (synchrony/field causality direction stated), M2 (stimulus-specific test, not just frequency).

**Risk:** HIGH — the stimulus-specificity of LFP spatial modes has not been demonstrated for this model. If modes are not stimulus-discriminable, the fundamental encoding claim fails.

### Phase 4: Gap Junction Topology Control (L2) and Cortical Modulation

**Rationale:** After demonstrating stimulus-encoding in Phase 3, introduce dynamic gap junction modulation (L2 layer) to test whether cortical top-down control can reshape the LFP spatial modes. This validates the "folding the manifold" concept. The biophysical mechanism is the Landisman & Connors (2005) mGluR1-mediated modulation. This phase is mechanistically cleaner after the Phase 3 baseline is established.

**Delivers:** (a) LFP mode structure under two or more cortical drive conditions (different g_j topologies); (b) Demonstration that mode reshaping is cortically controllable; (c) Timescale of topology switching.

**Builds on:** Phase 3 encoding demonstrations; Landisman & Connors (2005) modulation mechanism.

### Phase 5: Scaling — N=1,000 and Resource Characterization

**Rationale:** Phase 2-3 use N=100. Phase 5 tests whether the encoding results scale favorably with N, and whether N=1,000 is computationally feasible on available hardware. This also addresses the FMM transition threshold if N approaches 3,000.

**Delivers:** (a) N scaling curve: discriminability vs. N; (b) Holographic recovery fidelity vs. N; (c) Computational cost characterization for project planning.

**Uses:** JAX-GPU dense tensor (N ≤ 3,000); introduce ExaFMM-t if N > 3,000.

**Risk:** MEDIUM — N=1,000 HH cable feasibility is estimated but unvalidated.

---

### Phase Ordering Rationale

The ordering is strictly dictated by physical and computational dependencies:
- Phase 1 must precede everything — three show-stoppers (observable, framework, cell type choice) must be resolved before any multi-cell simulation
- Phase 2 must precede Phase 3 — synchrony transition and operating point must be characterized before stimulus encoding is meaningful
- Phase 3 must precede Phase 4 — the encoding baseline must exist before topology control is tested
- Phase 5 can follow Phase 3 or run in parallel with Phase 4 — N scaling is logistically independent

### Phases Requiring Deep Investigation

- **Phase 1 (TRN vs relay cell design decision):** Read Hales (2014) primary source, Landisman et al. (2002), and additional TRN biology. The relay-vs-TRN question is not settled by the current research files and materially affects all downstream parameters.
- **Phase 3 (stimulus encoding):** No prior work has demonstrated stimulus-specific LFP spatial modes in a simulated thalamic syncytium. This is genuinely novel and may fail.

Phases with established methodology:
- **Phase 1 (HH implementation and validation):** Well-documented procedure; NEURON cross-check is straightforward.
- **Phase 2 (g_j sweep):** Standard parameter sweep; tools and analysis well established.
- **Phase 5 (scaling):** Standard resource characterization.

---

## Critical Claim Verification

Web search was unavailable during this synthesis. The following high-impact claims are flagged for independent verification before Phase 1 planning concludes:

| # | Claim | Source | Action Required | Stakes |
|---|-------|--------|----------------|--------|
| 1 | Quasi-static valid for all neural EM (f << σ/(2πε) ≈ 10⁹ Hz in tissue) | Nunez & Srinivasan 2006 | Verify in textbook Ch. 1; check retardation ratio explicitly | Blocks C3 resolution |
| 2 | TRN gap junctions (Cx36): g_j = 0.1–2 nS; coupling coeff 0.01–0.1 | Landisman et al. 2002 | Read primary paper; confirm is TRN not relay cells | Blocks cell type decision |
| 3 | I_T mandatory for spindle generation in thalamus | Steriade et al. 1993 | Read primary paper; confirm parameter ranges for I_T, I_h | Blocks HH model construction |
| 4 | N~10,000–100,000 required for scalp EEG (not LFP) | Nunez & Srinivasan 2006 | Confirm number estimate; check for thalamic LFP data | Locks observable choice |
| 5 | Hales (2014) uses full Maxwell vs. quasi-static (claimed in project context) | Hales 2014 | Read paper directly; assess whether retarded treatment is justified | Critical — if quasi-static is equivalent, the Hales method reduces to standard LFP |
| 6 | LFP cancellation for randomly oriented populations (near-complete) | Linden et al. 2010 | Read paper; check TRN morphological alignment data | Affects N required for observable LFP |
| 7 | FlyWire/Shiu 2024 used LIF not HH cable | Shiu et al. 2024 | Confirm in paper; critical for scale feasibility framing | Blocks FlyWire as HH scale benchmark |

---

## Cross-Validation Matrix

| | NEURON (single cell) | Analytical Green's function | LFPy | Kuramoto theory | Empirical thalamic LFP |
|--|:--:|:--:|:--:|:--:|:--:|
| Brian2 HH | Spike waveform, < 0.1 mV RMS | — | — | — | Spindle frequency 7–14 Hz |
| JAX Green's function | — | Point-source < 1% error | LFP amplitude check | — | LFP amplitude range |
| SVD modes | — | — | — | — | Stimulus-specific LFP oscillations (Gray & Singer) |
| Synchrony transition | — | — | — | Kuramoto g_j^c estimate | — |

Methods with no cross-validation (high risk): 3DGS sensory input → LFP coupling (no prior benchmark); holographic recovery metric (no empirical comparator identified yet).

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Computational Approaches | MEDIUM | Brian2CUDA multi-compartment stability unvalidated; resource estimates not benchmarked; FMM library availability unconfirmed |
| Prior Work | MEDIUM | Foundational biology (HH, cable, spindles) is HIGH; Hales 2014 and Lehar 2003 specifics are MEDIUM (unread primary sources); gap junction parameters for relay cells LOW |
| Methods | MEDIUM | Quasi-static EM is well established; SVD/DMD mode extraction is standard; 3DGS sensory integration is new and unvalidated in this context |
| Pitfalls | HIGH | The four critical pitfalls (EEG amplitude scale, standing wave terminology, holographic operationalization, g_j sweep) are clearly identified with specific prevention strategies |

**Overall confidence:** MEDIUM

### Gaps to Address

- **Hales (2014) primary source must be read before Phase 1 design.** The project relies on Hales as a method anchor, but the specific computational approach (full retarded vs. quasi-static equivalent) is unconfirmed. Resolution materially affects the EM computation strategy.
- **TRN vs. relay cell design decision is open.** Current literature favors TRN as the gap-junction-coupled substrate. If TRN is chosen, parameters (I_T, g_j, cell morphology) differ from excitatory relay cells. This must be resolved before HH model construction.
- **No EEG benchmark dataset identified.** The empirical comparison for stimulus-specific oscillations (Gray & Singer 1989 is for cortex, not thalamus) needs a specific thalamic LFP dataset. Identify in Phase 1.
- **Brian2CUDA multi-compartment cable stability.** Must be validated in Phase 1 before committing as primary tool. GeNN is the fallback.
- **Lehar quantitative predictions do not exist.** Spatial wavelengths, standing wave frequencies, and amplitudes for the LFP spatial modes must be derived from first principles in Phase 1, not imported from Lehar's phenomenology.

---

## Open Questions

**HIGH priority (block subsequent phases):**

1. Is the syncytium substrate TRN (GABAergic, well-coupled) or excitatory relay cells (weakly/uncoupled)? Answer determines all cell-type parameters. [Blocks Phase 1 HH construction]
2. Does Hales (2014) add anything to the standard quasi-static LFP computation, or is it equivalent? [Blocks Phase 1 EM method choice]
3. What thalamic LFP dataset will serve as the empirical validation comparator for stimulus-specific oscillations? [Blocks Phase 3 validation]
4. Is Brian2CUDA multi-compartment HH cable numerically stable? If not, which fallback (GeNN or custom JAX)? [Blocks Phase 1 implementation]

**MEDIUM priority (must address before Phase 3):**

5. What is the ephaptic feedback amplitude from N=100 syncytium field? Is it above or below 1% of spike threshold? [Determines whether CEMI bidirectional scope is relevant]
6. Does TRN have morphologically aligned cells that support coherent (non-canceling) LFP? [Determines minimum N for observable field]
7. What is the Kuramoto-predicted g_j^c for synchrony onset in a realistic HH TRN network with measured heterogeneity? [Guides Phase 2 parameter range]

**LOW priority (future work):**

8. Can the CEMI bidirectional coupling (McFadden) be incorporated as a second-order correction without destabilizing the simulation?
9. Can the holographic manifold be analyzed with information-geometric tools (e.g., Fisher information metric on mode space)?

---

## Sources

### Primary — HIGH confidence

- Hodgkin & Huxley (1952). J. Physiol. 117, 500–544. DOI: 10.1113/jphysiol.1952.sp004764 — HH equations, spike generation
- Rall (1969). Biophys. J. 9, 1483–1508. DOI: 10.1016/S0006-3495(69)86435-9 — Cable theory, J(x,t) source term
- Steriade, McCormick & Sejnowski (1993). Science 262, 679–685. DOI: 10.1126/science.8235588 — Thalamo-cortical spindle mechanism, I_T/I_h circuit
- Nunez & Srinivasan (2006). Electric Fields of the Brain, 2nd ed. OUP. — Quasi-static neural EM, forward model, EEG generator requirements
- Gray & Singer (1989). PNAS 86, 1698–1702. DOI: 10.1073/pnas.86.5.1698 — Stimulus-specific gamma oscillations; key observable model

### Secondary — MEDIUM confidence (verify primary sources before use in derivations)

- McCormick & Huguenard (1992). J. Neurophysiol. 68(4), 1384–1400. DOI: 10.1152/jn.1992.68.4.1384 — Thalamic relay cell parameter set
- Landisman et al. (2002). J. Neurosci. 22(3), 1002–1009. DOI: 10.1523/JNEUROSCI.22-03-01002.2002 — Cx36 gap junctions in TRN
- Landisman & Connors (2005). Science 310, 1809–1813. DOI: 10.1126/science.1114655 — Cortical modulation of TRN gap junctions
- **Hales, C.G. (2014). J. Integr. Neurosci. 13(2), 313–361. DOI: 10.1142/S021963521440013X [VERIFY]** — PRIMARY METHOD ANCHOR — must read before Phase 1
- Linden et al. (2010). PLoS Comput. Biol. 6(11): e1000972. DOI: 10.1371/journal.pcbi.1000972 [VERIFY] — LFP from cable populations; orientation/synchrony effects
- Stimberg et al. (2019). eLife, DOI: 10.7554/eLife.47314 [VERIFY] — Brian2 simulator
- McFadden (2002a). J. Conscious. Stud. 9(4), 23–50 — CEMI theory; EM field as causal integrator
- McFadden (2002b). J. Conscious. Stud. 9(8), 45–60 — CEMI theory full statement
- Shiu et al. (2024). Nature [DOI TBD — VERIFY] — FlyWire scale benchmark (LIF, not HH cable)

### Tertiary — LOW confidence (theoretical anchors without quantitative predictions; needs primary source verification)

- **Lehar, S. (2003). Behav. Brain Sci. 26(4), 375–408. DOI: 10.1017/S0140525X03000098 [VERIFY]** — PRIMARY THEORY ANCHOR — harmonic resonance, holographic world model; no quantitative predictions for thalamic substrate
- Lehar, S. (2003). The World in Your Head. Lawrence Erlbaum. ISBN: 978-0805838886 — Extended theory exposition
- Pribram (1991). Brain and Perception. Lawrence Erlbaum. ISBN: 978-0898599954 — Holonomic brain theory (theoretical ancestor to Lehar)

---

_Research analysis completed: 2026-03-16_
_Ready for research plan: yes — with Phase 1 prerequisite of reading Hales (2014) and resolving TRN vs. relay cell design decision_

---

```yaml
# --- ROADMAP INPUT (machine-readable, consumed by gpd-roadmapper) ---
synthesis_meta:
  project_title: "Thalamo-Cortical EM Manifold Model"
  synthesis_date: "2026-03-16"
  input_files: [METHODS.md, PRIOR-WORK.md, COMPUTATIONAL.md, PITFALLS.md]
  input_quality:
    METHODS: good
    PRIOR-WORK: good
    COMPUTATIONAL: good
    PITFALLS: good

conventions:
  unit_system: "SI for EM (V/m, T, A/m²); neural units (mV, ms, nS, µm) for dynamics"
  metric_signature: "N/A — quasi-static (Poisson), not wave equation"
  fourier_convention: "physics (e^{-iωt} forward transform)"
  coupling_convention: "Ohmic gap junction: I_gap = g_j·(V_i - V_j); g_j in nS"
  renormalization_scheme: "N/A"
  EM_approximation: "quasi-static (Poisson): ∇·(σ∇φ) = -∇·J_imp; full-wave FDTD not needed"
  standing_wave_definition: "LFP spatial eigenmode of network dynamics — NOT EM radiation mode"
  HH_convention: "outward-positive; h = fraction NOT inactivated; I_Na = g_Na·m³·h·(V_m - E_Na)"
  observable: "LFP (local field potential), not scalp EEG; absolute amplitude in V/m required"

methods_ranked:
  - name: "HH cable model with I_T and I_h (Brian2+Brian2CUDA)"
    regime: "Single cell to N~1,000; thalamic burst/spindle dynamics require I_T and I_h"
    confidence: HIGH
    cost: "O(N·N_compartments) per step; N=100: ~1s wall time (estimated, unvalidated)"
    complements: "NEURON (single-cell validation); GeNN (fallback if Brian2CUDA unstable)"
  - name: "Quasi-static Green's function LFP kernel (JAX-GPU dense tensor)"
    regime: "N ≤ 3,000; homogeneous isotropic medium; valid for all neural frequencies"
    confidence: HIGH
    cost: "O(N²) per timestep; N=1,000: ~4 GB VRAM float32; trivial for N≤1,000"
    complements: "FMM (ExaFMM-t) for N > 3,000"
  - name: "SVD spatial mode extraction"
    regime: "Stationary or slowly varying field patterns; any N"
    confidence: HIGH
    cost: "O(N_obs × T²) for T snapshots; cheap post-processing"
    complements: "DMD (for transient dynamics or frequency-resolved modes)"
  - name: "Gap junction Ohmic coupling (linear g_j) sweep"
    regime: "g_j < 1 nS (coupling coeff < 0.1); fixed conductance"
    confidence: HIGH
    cost: "Included in Brian2 Synapses; minimal overhead"
    complements: "Voltage-gated Cx36 model (if g_j > 1 nS or transjunctional V > 30 mV)"
  - name: "Fast Multipole Method (ExaFMM-t)"
    regime: "N > 3,000; required when dense tensor exceeds VRAM"
    confidence: MEDIUM
    cost: "O(N log N) per step; library availability [VERIFY]"
    complements: "Dense tensor (N ≤ 3,000)"
  - name: "Dynamic Mode Decomposition (DMD)"
    regime: "Transient dynamics; when frequency + growth rate needed alongside spatial structure"
    confidence: MEDIUM
    cost: "O(N_obs × T²); similar to SVD"
    complements: "SVD (energy-ranked modes)"

phase_suggestions:
  - name: "Theory Framework and Single-Cell Validation"
    goal: "Lock all theory definitions; validate HH cable with I_T/I_h against NEURON; validate Green's function LFP kernel; decide TRN vs relay cell."
    methods: ["HH cable model with I_T and I_h (Brian2+Brian2CUDA)", "Quasi-static Green's function LFP kernel (JAX-GPU dense tensor)"]
    depends_on: []
    needs_research: true
    risk: MEDIUM
    pitfalls: ["C1-EEG-amplitude-scale-gap", "C3-quasi-static-vs-wave-confusion", "C2-holographic-operationalization", "M1-HH-ODE-stiffness", "M3-tissue-homogeneity", "M5-EEG-vs-LFP", "m1-HH-initialization", "m3-ephaptic-coupling-scope"]
  - name: "N=100 Syncytium Pilot and Gap Junction Sweep"
    goal: "Build N=100 coupled syncytium; sweep g_j 0.1–10 nS; identify synchrony threshold; characterize LFP spatial modes and absolute amplitude; run uncoupled control."
    methods: ["HH cable model with I_T and I_h (Brian2+Brian2CUDA)", "Quasi-static Green's function LFP kernel (JAX-GPU dense tensor)", "SVD spatial mode extraction", "Gap junction Ohmic coupling (linear g_j) sweep"]
    depends_on: ["Theory Framework and Single-Cell Validation"]
    needs_research: false
    risk: MEDIUM
    pitfalls: ["C4-gap-junction-parameter-uncertainty", "M2-EEG-bands-without-mechanism-control", "m2-fixed-gj-during-spikes"]
  - name: "Stimulus-Specific Spatial Mode Encoding"
    goal: "Drive N=100 syncytium at Phase 2 operating point with distinct 3D sensory stimuli (visual + audio); test stimulus discriminability of LFP spatial modes; measure holographic recovery after cell loss."
    methods: ["SVD spatial mode extraction", "HH cable model with I_T and I_h (Brian2+Brian2CUDA)", "Quasi-static Green's function LFP kernel (JAX-GPU dense tensor)"]
    depends_on: ["N=100 Syncytium Pilot and Gap Junction Sweep"]
    needs_research: false
    risk: HIGH
    pitfalls: ["C2-holographic-operationalization", "M4-synchrony-vs-field-coherence-conflation", "M2-EEG-bands-without-mechanism-control"]
  - name: "Gap Junction Topology Control (L2) and Cortical Modulation"
    goal: "Introduce dynamic gap junction modulation (mGluR1-mediated) to test whether cortical top-down control reshapes LFP spatial modes; validate 'folding the manifold' concept."
    methods: ["Gap junction Ohmic coupling (linear g_j) sweep", "SVD spatial mode extraction", "Dynamic Mode Decomposition (DMD)"]
    depends_on: ["Stimulus-Specific Spatial Mode Encoding"]
    needs_research: false
    risk: MEDIUM
    pitfalls: ["M4-synchrony-vs-field-coherence-conflation"]
  - name: "N=1,000 Scaling and Resource Characterization"
    goal: "Test whether encoding results scale favorably with N; characterize computational cost; determine if FMM transition is needed."
    methods: ["HH cable model with I_T and I_h (Brian2+Brian2CUDA)", "Quasi-static Green's function LFP kernel (JAX-GPU dense tensor)", "Fast Multipole Method (ExaFMM-t)"]
    depends_on: ["Stimulus-Specific Spatial Mode Encoding"]
    needs_research: false
    risk: MEDIUM
    pitfalls: ["C1-EEG-amplitude-scale-gap"]

critical_benchmarks:
  - quantity: "Single-cell HH spike waveform vs. NEURON"
    value: "< 0.1 mV RMS deviation at matching parameters"
    source: "METHODS.md validation table"
    confidence: HIGH
  - quantity: "Thalamo-cortical spindle frequency"
    value: "7–14 Hz"
    source: "Steriade et al. 1993"
    confidence: HIGH
  - quantity: "TRN Cx36 gap junction conductance"
    value: "g_j = 0.1–2 nS per junction; coupling coefficient 0.01–0.1"
    source: "Landisman et al. 2002"
    confidence: MEDIUM
  - quantity: "Green's function LFP kernel error"
    value: "< 1% on point-source test case"
    source: "METHODS.md validation table"
    confidence: HIGH
  - quantity: "HH simulation timestep constraint"
    value: "dt ≤ 0.025 ms (never > 0.05 ms)"
    source: "HH Na+ activation kinetics; standard in Brian2/NEURON literature"
    confidence: HIGH
  - quantity: "Scalp EEG generator requirement (NOT target for this project)"
    value: "~10,000–100,000 synchronously firing neurons over cm² patches"
    source: "Nunez & Srinivasan 2006"
    confidence: HIGH

open_questions:
  - question: "Is the syncytium substrate TRN (well-coupled) or excitatory relay cells (weakly/absent gap junctions)?"
    priority: HIGH
    blocks_phase: "Theory Framework and Single-Cell Validation"
  - question: "Does Hales (2014) add anything to standard quasi-static LFP computation, or is it physically equivalent?"
    priority: HIGH
    blocks_phase: "Theory Framework and Single-Cell Validation"
  - question: "What specific thalamic LFP dataset will be used as empirical validation comparator for stimulus-specific oscillations?"
    priority: HIGH
    blocks_phase: "Stimulus-Specific Spatial Mode Encoding"
  - question: "Is Brian2CUDA multi-compartment HH cable numerically stable? If not, which fallback?"
    priority: HIGH
    blocks_phase: "Theory Framework and Single-Cell Validation"
  - question: "What is the ephaptic feedback amplitude from N=100 syncytium field — is it above 1% of spike threshold?"
    priority: MEDIUM
    blocks_phase: "none"
  - question: "Does TRN have morphologically aligned cells that support coherent (non-canceling) LFP at N=100?"
    priority: MEDIUM
    blocks_phase: "N=100 Syncytium Pilot and Gap Junction Sweep"
  - question: "What is the Kuramoto-predicted g_j^c for synchrony onset in realistic HH TRN network?"
    priority: MEDIUM
    blocks_phase: "none"

contradictions_unresolved:
  - claim_a: "Project context states Hales uses 'full retarded potential treatment' (full Maxwell) for neural EM field"
    claim_b: "Standard literature (Nunez & Srinivasan 2006; Plonsey & Heppner 1967) establishes that quasi-static is exact to 9 significant figures at neural scales (retardation correction ~10^-11); full Maxwell adds nothing"
    source_a: "PRIOR-WORK.md citing Hales 2014 project context"
    source_b: "PITFALLS.md C3; METHODS.md analytical methods table"
    investigation_needed: "Read Hales (2014) primary source directly and assess whether paper claims the retarded treatment is necessary at neural scales, or whether the paper uses full Maxwell for generality while the quasi-static limit holds in practice. This determines whether the 'Maxwell-equation treatment' framing is a meaningful distinction from LFP modeling."
  - claim_a: "TRN gap junction conductance: 0.1–2 nS (Landisman 2002)"
    claim_b: "PITFALLS.md cites g_j range 0.1–10 nS for thalamic neurons generally"
    source_a: "PRIOR-WORK.md, METHODS.md"
    source_b: "PITFALLS.md C4"
    investigation_needed: "Confirm whether the 0.1–10 nS range includes excitatory relay cells (which may have different or absent gap junctions) or is TRN-only. Landisman 2002 is TRN-specific. The sweep range should be set based on the chosen cell type."
```
