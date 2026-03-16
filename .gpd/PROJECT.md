# Thalamo-Cortical EM Manifold Model

## What This Is

This project models the thalamo-cortical system as a three-layer electromagnetic computation architecture. Cortical columns act as computational controllers, driving a parametric thalamic syncytium (N gap-junction coupled cells) whose collective endogenous EM field constitutes a folding electromagnetic manifold. The primary hypothesis is that this manifold encodes a holographic model of the sensed world — via Lehar-style standing wave interference patterns — and that this encoding is detectable as EEG-like oscillatory signatures correlated with stimulus identity.

## Core Research Question

Can a Maxwell-equation treatment of the endogenous EM field of a parametric thalamic syncytium (HH cable neurons, gap junction coupling, cortical column controllers) produce EM manifold dynamics — standing wave patterns and folding transitions — that holographically encode 3D sensory inputs and reproduce EEG-like oscillatory signatures correlated with stimulus identity?

## Scoping Contract Summary

### Contract Coverage

- **claim-main** (EEG signatures): Simulated thalamic EM field PSD shows peaks within known EEG bands AND profiles are statistically distinguishable across 3D sensory stimuli
- **claim-holographic** (standing wave encoding): Standing wave patterns encode distinguishable 3D inputs with >70% recovery fidelity after 20% cell loss
- **False progress to reject**: Oscillations uncorrelated with stimulus identity; 2D sensory shortcuts; qualitative EEG resemblance without frequency band matching

### User Guidance To Preserve

- **User-stated observables**: EEG-like frequency signatures from simulated field equations; standing wave spatial maps per stimulus; holographic recovery after partial cell loss
- **User-stated deliverables**: Working parametric simulation (N tunable); PSD figure per stimulus vs EEG bands; standing wave encoding figure with recovery demonstration
- **Must-have references / prior outputs**: Hales endogenous EM field paper (method anchor); Lehar harmonic resonance theory (theory anchor); FlyWire/Shiu 2024 (scale feasibility benchmark)
- **Stop / rethink conditions**: No oscillatory structure at any stimulus (fundamental failure); EEG signatures uncorrelated with stimulus (encoding failure); N~100 computationally intractable (rethink EM computation strategy)

### Scope Boundaries

**In scope**

- Three-layer thalamo-cortical model: L1 multipole EM field from HH cable currents (Hales method), L2 gap junction topology switching, L3 phase-space meta-optimization
- Cortical column controllers modulating thalamic syncytium via efferent connections (top-down control → cavity boundary conditions)
- 3D sensory input: light field (visual) and spatial audio field (auditory) — genuine spatial 3D fields
- Holographic world model encoding via standing wave interference patterns (Lehar harmonic resonance)
- EEG-like oscillatory signature extraction from simulated population-level EM field dynamics
- Parametric scalability: N tunable up and down for computational feasibility
- Efficient compute kernels for neural EM field physics
- Bulk-boundary correspondence encoding (L2 phenomenon)
- Distributed error-correcting holographic redundancy (L3 phenomenon)

**Out of scope**

- Full motor control loop closure (FlyWire-style neural-to-action) — future growth target
- Manifold dimensionality analysis — future work
- 2D sensory simplification as the primary sensory model (virtual test sources OK inside 3D field)
- Consciousness claims

### Active Anchor Registry

- **Ref-Hales**: Hales, C. — Origins of the brain's endogenous electromagnetic field (2014)
  - Why it matters: Primary method anchor for computing EM field from neural cable current distributions via Maxwell's equations — foundation of L1
  - Carry forward: planning, execution, verification
  - Required action: read, use, cite

- **Ref-Lehar**: Lehar, S. — Harmonic Resonance Theory / The World in Your Head
  - Why it matters: Primary theory anchor — defines standing wave holographic encoding and what "folding the manifold" means
  - Carry forward: planning, execution, verification, writing
  - Required action: read, use, cite

- **Ref-FlyWire**: Shiu et al. — Nature 2024 — FlyWire connectome simulation
  - Why it matters: Scale and feasibility benchmark; target architecture to grow toward
  - Carry forward: planning, execution
  - Required action: read, compare

- **Ref-EEG-empirical**: EEG empirical literature (thalamic spindles 7-14 Hz, alpha 8-12 Hz, gamma 30-80 Hz)
  - Why it matters: Smoking-gun validation target — specific benchmark dataset TBD in Phase 1
  - Carry forward: planning, execution, verification
  - Required action: compare, cite

### Carry-Forward Inputs

- None confirmed yet — prior outputs accumulate from Phase 1 onward

### Skeptical Review

- **Weakest anchor**: Gap between Hales' single-neuron/small-population EM treatment and the syncytium-scale collective field dynamics needed for holographic encoding — new physics must be derived
- **Unvalidated assumptions**: Syncytium large enough to support EEG-frequency standing wave modes; Maxwell+HH gives good macroscopic EM field approximation; gap junction timescales compatible with encoding timescales
- **Competing explanation**: EEG-like signatures could emerge from synaptic currents alone (undermining L1 necessity); holographic encoding may be at synaptic weight level rather than physical EM field
- **Disconfirming observation**: EEG signatures appear but are stimulus-independent; computational cost makes N~100 intractable
- **False progress to reject**: Oscillations without stimulus correlation; qualitative EEG resemblance without frequency matching; 2D sensory shortcuts

### Open Contract Questions

- Decisive EEG benchmark dataset not yet selected — identify specific experimental paper before execution
- Precise gap junction coupling model for thalamic cells TBD (literature search Phase 1)
- Which EEG signatures most diagnostic: spindles, alpha, gamma, or cross-frequency coupling?
- Optimal neuron model complexity: full HH cable vs. simplified compartmental for practical scalability

## Research Questions

### Answered

(None yet — investigate to answer)

### Active

- [ ] Can Hales' Maxwell+cable method be extended to a gap-junction coupled syncytium of N cells to yield a meaningful collective EM field?
- [ ] Does the collective thalamic EM field support standing wave modes at EEG-relevant frequencies (7-80 Hz)?
- [ ] Are the standing wave patterns stimulus-specific when driven by 3D sensory input (light field, spatial audio)?
- [ ] Does gap junction topology (L2) modulate the standing wave mode structure in ways analogous to known thalamic gating?
- [ ] Is the holographic redundancy of the field encoding sufficient for >70% recovery after 20% cell loss?
- [ ] What is the minimum N for which meaningful holographic encoding emerges?

### Out of Scope

- Motor loop closure / action control — requires different subfield (motor control, FlyWire architecture)
- Manifold dimensionality analysis — future work
- Consciousness mechanism claims — beyond the scope of EM field modeling

## Research Context

### Physical System

A parametric thalamic syncytium of N electrically coupled neurons (gap junctions as tunable connectivity), driven by afferent 3D sensory input and modulated by cortical column efferents. Each cell is modeled as a Hodgkin-Huxley cable (full axonal and dendritic morphology), with the surrounding endogenous EM field computed via Maxwell's equations applied to the distributed current distribution. Cortical columns act as active controllers of the syncytium's effective boundary conditions, shaping which resonant modes are accessible.

### Theoretical Framework

Computational neuroscience / biological electrodynamics at the intersection of:
- Classical electrodynamics (Maxwell's equations in biological media)
- Hodgkin-Huxley cable theory (distributed current in neural morphology)
- Nonlinear dynamics (syncytium as coupled oscillator network)
- Holographic field theory (Lehar harmonic resonance, Pribram holonomic theory)
- Information geometry (bulk-boundary correspondence, error-correcting redundancy)

### Key Parameters and Scales

| Parameter | Symbol | Regime | Notes |
|-----------|--------|--------|-------|
| Syncytium size | N | 10–10,000 | Parametric — tunable for compute budget |
| Gap junction conductance | g_j | 0.1–10 nS | Literature range for thalamic cells |
| EEG target frequencies | f | 7–80 Hz | Spindles, alpha, gamma |
| HH cable diameter | d | 0.5–5 μm | Thalamic relay cell morphology |
| 3D visual field resolution | — | TBD | Light field representation |
| Compute kernel target | — | N~100 tractable | Hard stop/rethink if not achievable |

### Known Results

- Hales (2014): Full Maxwell treatment of endogenous EM field from HH cable currents — single-neuron and small-population level
- Lehar: Harmonic resonance theory predicts standing wave spatial periodicity matching Gestalt perceptual organization
- FlyWire/Shiu 2024: Full connectome-scale neural simulation is feasible with simplified neuron models; demonstrates path to large N
- Empirical EEG: Thalamic spindles 7-14 Hz, cortical alpha 8-12 Hz, gamma 30-80 Hz well-characterized

### What Is New

Extending Hales' single-cell endogenous EM field treatment to a gap-junction coupled syncytium and asking whether the collective field can self-organize into Lehar-style standing wave patterns that holographically encode 3D sensory inputs — with EEG-like signatures as the decisive empirical test. No prior work has attempted this three-layer (L1 field / L2 topology / L3 meta) architecture with these specific theoretical anchors.

### Target Venue

To be determined after Phase 1 results — likely: *PLOS Computational Biology*, *Neural Computation*, or *Journal of Computational Neuroscience* depending on whether the main result is primarily computational or theoretical.

### Computational Environment

- Primary language: TBD in Phase 1 (Python with JAX/CUDA or C++/CUDA likely given EM field compute demands)
- Target hardware: GPU-accelerated (CUDA) for EM field kernels; local workstation for prototyping; cluster for full-N sweeps
- Reference codebase: FlyWire connectome simulation architecture (Shiu 2024) as scaling guide

## Notation and Conventions

See `.gpd/CONVENTIONS.md` for all notation and sign conventions.

## Unit System

SI units for EM field quantities (V/m, T, A/m²); natural units where convenient for neural dynamics (mV, ms, nS, μm).

## Requirements

See `.gpd/REQUIREMENTS.md` for the detailed requirements specification.

Key requirement categories: DERV (derivation), CALC (calculation), SIMU (simulation), VALD (validation)

## Key References

- **Ref-Hales** (method anchor): Hales, C. — Origins of the brain's endogenous electromagnetic field — 2014
- **Ref-Lehar** (theory anchor): Lehar, S. — Harmonic Resonance Theory / The World in Your Head
- **Ref-FlyWire** (scale anchor): Shiu et al. — Nature 2024 — FlyWire connectome neural simulation
- **Ref-EEG-empirical** (validation target): EEG empirical literature — benchmark dataset TBD in Phase 1

## Constraints

- **Computational**: N~100 tractable must be achievable — if not, fundamental rethink of EM field computation strategy
- **Sensory modeling**: 3D sensory fields required — no 2D shortcuts in the core model
- **Scalability**: All major components must be parameterized in N for up/down scaling

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| HH cable + Maxwell as L1 method | Hales provides the most rigorous treatment of endogenous EM field from neural morphology | Pending validation of scalability |
| Lehar harmonic resonance as theory anchor | Explicit predictions about standing wave structure and holographic encoding | — Pending |
| EEG signatures as smoking-gun test | Observable from real thalamic/cortical data; not a proxy — directly tests the field dynamics hypothesis | — Pending |
| 3D sensory fields required | Eyes and ears do 3D sensing up-front; 2D shortcuts would invalidate the encoding claims | — Locked |
| FlyWire/Shiu as scale benchmark | Best available demonstration of what connectome-scale neural simulation can achieve | — Pending |

---

_Last updated: 2026-03-16 after initialization_
