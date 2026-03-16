# Research State

## Project Reference

See: `.gpd/PROJECT.md` (updated 2026-03-16)

**Machine-readable scoping contract:** `.gpd/state.json` field `project_contract`

**Core research question:** Can a Maxwell-equation treatment of the endogenous EM field of a parametric thalamic syncytium (HH cable neurons, gap junction coupling, cortical column controllers) produce EM manifold dynamics — standing wave patterns and folding transitions — that holographically encode 3D sensory inputs and reproduce EEG-like oscillatory signatures correlated with stimulus identity?

**Note on "EEG-like":** The correct target observable is LFP (local field potential, 100–1000 µm from syncytium, amplitude in V/m or µV/mm). N~100–1,000 thalamic cells cannot produce scalp EEG amplitudes. EEG frequency band labels (spindle 7–14 Hz, alpha 8–13 Hz, gamma 30–100 Hz) remain valid as oscillation frequency targets.

**Current focus:** Phase 1 — Theory Framework and Single-Cell Validation

## Current Position

**Current Phase:** 01
**Current Phase Name:** Theory Framework and Single-Cell Validation
**Total Phases:** 4
**Current Plan:** —
**Total Plans in Phase:** TBD
**Status:** Ready to plan
**Last Activity:** 2026-03-16
**Last Activity Description:** Roadmap created; research literature survey complete; requirements defined; ready for Phase 1 planning

**Progress:** [░░░░░░░░░░] 0%

## Active Calculations

None yet — Phase 1 planning not started.

## Intermediate Results

None yet.

## Open Questions

**HIGH priority (block Phase 1 execution):**

- Is the syncytium substrate TRN (GABAergic, well-coupled via Cx36) or excitatory relay cells (weakly/absent gap junctions)? Answer determines all cell-type parameters. [Blocks DERV-03, SIMU-01]
- Does Hales (2014) add anything to the standard quasi-static LFP computation, or is it effectively equivalent? [Blocks DERV-01, Phase 1 EM method confirmation]
- What specific thalamic LFP dataset will serve as the empirical validation comparator for stimulus-specific oscillations in Phase 3? [Must identify in Phase 1]
- Is Brian2CUDA multi-compartment HH cable numerically stable? If not, which fallback (GeNN or custom JAX)? [Blocks SIMU-01]

**MEDIUM priority (must address before Phase 3):**

- What is the ephaptic feedback amplitude from N=100 syncytium field — above or below 1% of spike threshold (~20 mV)? [Determines whether CEMI bidirectional scope is in-scope or deferred]
- Does TRN have morphologically aligned cells that support coherent (non-canceling) LFP at N=100? [Affects minimum N for observable field]
- What is the Kuramoto-predicted g_j^c for synchrony onset in realistic HH TRN network with measured heterogeneity? [Guides Phase 2 g_j sweep range]

## Performance Metrics

| Label | Duration | Tasks | Files |
| ----- | -------- | ----- | ----- |
| - | - | - | - |

## Accumulated Context

### Decisions

Full log: `.gpd/DECISIONS.md`

**Pre-Phase 1 decisions locked by literature survey:**

- [Pre-Phase 1] Observable locked as LFP, not scalp EEG — N~100–1,000 cells cannot produce scalp EEG amplitudes (4–6 orders of magnitude gap); all amplitude comparisons must use LFP literature (e.g., Linden et al. 2010)
- [Pre-Phase 1] "Standing waves" redefined as LFP spatial eigenmodes of network dynamics — NOT electromagnetic radiation modes; quasi-static Maxwell (Poisson) is the operative approximation; "standing wave" means dominant left singular vector of E-field snapshot matrix
- [Pre-Phase 1] Holographic encoding operational definition deferred to Phase 1 execution — must specify F(x,t), reconstruction operator, and fidelity metric before Phase 2 begins
- [Pre-Phase 1] g_j sweep (0.1–10 nS, >= 10 points) is mandatory in Phase 2 — no result reported at a single g_j value
- [Pre-Phase 1] Uncoupled (g_j=0) control required in Phase 2 for every stimulus condition — decisive test is spatial mode structure, not frequency content alone

### Active Approximations

| Approximation | Validity Range | Controlling Parameter | Current Value | Status |
| --- | --- | --- | --- | --- |
| Quasi-static Maxwell (Poisson) | f < 10 kHz, any neural scale | Retardation ratio L/lambda_EM | L/lambda ~10^-9 at f=1 kHz, L=10 mm | Valid — to confirm in Phase 1 |
| Homogeneous isotropic tissue | Syncytium-only; no skull/CSF | Tissue boundary distance | N/A for LFP within syncytium | Approximate; +/-factor 2-3 uncertainty |
| Dense Green's function tensor | N <= 3,000 | N x VRAM (float32 ~4 GB at N=1,000) | N = 100 (Phase 2), N = 1,000 (Phase 4) | Valid for target N; FMM needed above N~3,000 |
| Ohmic gap junction (linear g_j) | g_j < 1 nS, coupling coeff < 0.1 | Transjunctional voltage | 0.1–10 nS sweep | Approximate at high g_j; nonlinearity risk noted |
| HH cable with I_T, I_h (not LIF) | Single cell to N~1,000 HPC | N, dt | dt <= 0.025 ms required | Valid; cost ~4,000x LIF per simulated second |

**Convention Lock (from SUMMARY.md unified notation):**

- Metric signature: N/A — quasi-static (Poisson), not wave equation
- Fourier convention: physics e^{-i*omega*t} forward transform
- Natural units: not set (SI for EM: V/m, T, A/m²; neural: mV, ms, nS, µm)
- Gauge choice: N/A (quasi-static)
- Regularization: N/A
- Renormalization: N/A
- Coordinate system: not set (3D Cartesian for syncytium geometry)
- Coupling convention: Ohmic gap junction I_gap = g_j*(V_i - V_j); g_j in nS
- HH convention: outward-positive; h = fraction NOT inactivated; I_Na = g_Na*m^3*h*(V_m - E_Na)
- Observable convention: LFP phi = (1/4*pi*sigma) * integral J*grad'(1/|r-r'|) dV'; sigma = 0.33 S/m gray matter
- Timestep constraint: dt <= 0.025 ms (never > 0.05 ms)

### Propagated Uncertainties

| Quantity | Current Value | Uncertainty | Last Updated | Method |
| --- | --- | --- | --- | --- |
| TRN g_j per junction | 0.1–2 nS (Landisman 2002) | Range spans 20x; upper bound TBD | Pre-Phase 1 | Literature |
| LFP field amplitude at N=100 | TBD | Unknown until Phase 2 | — | — |
| Synchrony threshold g_j^c | TBD | Unknown; Kuramoto analogy gives order-of-magnitude | — | — |

### Pending Todos

None yet.

### Blockers / Concerns

- [Pre-Phase 1] TRN vs. relay cell design decision is unresolved — materially affects all cell-type parameters and Phase 1 HH model construction
- [Pre-Phase 1] Specific thalamic LFP empirical benchmark dataset not yet identified — must be resolved in Phase 1 before Phase 3 can be designed
- [Pre-Phase 1] Brian2CUDA multi-compartment stability unvalidated — potential Phase 1 blocker; GeNN is named fallback

## Session Continuity

**Last session:** 2026-03-16
**Stopped at:** Roadmap created; Phase 1 ready to plan
**Resume file:** —
