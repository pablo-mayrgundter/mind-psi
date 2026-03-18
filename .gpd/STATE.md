# Research State

## Project Reference

See: .gpd/PROJECT.md

**Core research question:** [Not set]
**Current focus:** [Not set]

## Current Position

**Current Phase:** —
**Current Phase Name:** —
**Total Phases:** —
**Current Plan:** —
**Total Plans in Phase:** —
**Status:** —
**Last Activity:** —

**Progress:** [░░░░░░░░░░] 0%

## Active Calculations

None yet.

## Intermediate Results

None yet.

## Open Questions

- How to bridge from Hales small-population EM field treatment to syncytium-scale collective field dynamics — new physics required
- Precise gap junction switching timescales and coupling strengths in the thalamus
- Which EEG signatures are most diagnostic: spindles, alpha, gamma, or cross-frequency coupling
- Optimal neuron model complexity: full HH cable vs simplified compartmental for practical scalability
- Decisive EEG benchmark dataset not yet selected

## Performance Metrics

| Label | Duration | Tasks | Files |
| ----- | -------- | ----- | ----- |
| -     | -        | -     | -     |

## Accumulated Context

### Decisions

None yet.

### Active Approximations

None yet.

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

None yet.

### Blockers/Concerns

None

## Session Continuity

**Last session:** —
**Stopped at:** —
**Resume file:** —
