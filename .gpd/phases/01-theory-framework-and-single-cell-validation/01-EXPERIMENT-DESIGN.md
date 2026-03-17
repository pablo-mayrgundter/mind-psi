# Experiment Design: Phase 1 — Theory Framework and Single-Cell Validation

> **For gpd-executor:** This file contains parameter specifications, convergence criteria, and
> statistical analysis plans. Use these when executing computational tasks in this phase.
> Every tolerance stated below is a hard go/no-go criterion unless explicitly marked advisory.

**Phase:** 01-theory-framework-and-single-cell-validation
**Sub-tasks covered:** 01-02 (HH cable cell), 01-03 (LFP Green's function kernel)
**Research mode:** balanced
**Autonomy:** balanced
**Date authored:** 2026-03-16

---

## Target Quantities

| Quantity | Symbol | Units | Expected Range | Required Accuracy | Validation Source |
|----------|--------|-------|----------------|-------------------|-------------------|
| LTS threshold voltage | V_LTS | mV | −70 to −55 mV | ±5 mV absolute | McCormick & Huguenard 1992 Fig. 1 |
| I_T m-gate half-activation | V_half | mV | −59 to −55 mV | ±2 mV absolute | Huguenard & McCormick 1992 Table 1 |
| Spindle burst recurrence frequency | f_spindle | Hz | 7–14 Hz | in-band | Steriade et al. 1993 |
| Intra-burst instantaneous frequency | f_burst | Hz | 100–400 Hz | in-band | Steriade et al. 1993 |
| Spike waveform RMS vs. NEURON | RMS_waveform | mV | 0 | < 0.1 mV | Phase spec VALD-01 |
| LFP point-source relative error | ε_LFP | dimensionless | 0 | < 0.01 (1%) | Phase spec VALD-02 |
| Brian2CUDA vs. CPU spike time difference | Δt_GPU | ms | 0 | < 0.1 ms | Phase spec SIMU-01 |

---

## Experiment 1: HH Numerical Convergence (dt and Spatial)

### 1A. Timestep (dt) Convergence Study

**Objective:** Confirm that spike timing and subthreshold dynamics are insensitive to dt at the
production value dt = 0.025 ms, and that the method becomes unstable or degrades visibly at
dt = 0.05 ms.

**Rationale for dt range:** HH gating variables have time constants as short as 0.1 ms (sodium
m-gate near threshold). Exponential Euler is first-order; Richardson extrapolation with p = 1 is
appropriate. The reference solution uses dt_ref = 0.005 ms (5× finer than production).

**System setup (identical across all dt values):**
- Model: 10-compartment HH cable (soma only for this test; see Section 1B for spatial)
- Channels: I_Na, I_K, I_L, I_T, I_h, I_NaP with Q10-corrected kinetics at 37°C
- Initial conditions: V = −65 mV; all gates at steady-state (m∞, h∞, n∞ at −65 mV)
- Settling: 100 ms silent (no injection) before recording; discard settling period
- Stimulus: single 5 ms current pulse at t = 120 ms; amplitude 0.5 nA (suprathreshold)
- Simulation duration: 300 ms total (100 ms settle + 200 ms recording)
- Integration: Crank-Nicolson for passive cable operator; exponential Euler for gate ODEs
- Boundary: sealed ends (no-flux) at both cable termini

**dt values to test:**

| Level | dt (ms) | Role |
|-------|---------|------|
| dt_1 | 0.005 | Reference solution (Richardson extrapolation target) |
| dt_2 | 0.010 | Convergence check point |
| dt_3 | 0.025 | Production value (must pass) |
| dt_4 | 0.050 | Expected degradation; documents upper stability boundary |

These are geometrically spaced by factor 2, enabling Richardson extrapolation with p = 1
(first-order exponential Euler).

**Observables and convergence metrics:**

For each dt value, record:

1. **Spike time t_AP:** Time of first action potential peak (V_m maximum). Measure from the
   somatic compartment. Criterion: |t_AP(dt) − t_AP(dt_ref)| < 0.1 ms at dt = 0.025 ms.

2. **AP peak voltage V_peak:** Maximum V_m during the action potential. Criterion: |V_peak(dt) −
   V_peak(dt_ref)| < 0.5 mV at dt = 0.025 ms (advisory; not a go/no-go).

3. **Subthreshold RMS deviation:** For the 20 ms epoch before the AP (t = 100–120 ms), compute
   RMS(V_m(dt) − V_m(dt_ref)) sampled at dt_ref resolution (interpolate coarser traces linearly).
   Criterion: < 0.05 mV at dt = 0.025 ms (advisory).

4. **Richardson extrapolation error estimate:** Using the three converged levels (dt_1, dt_2,
   dt_3), compute the estimated truncation error at dt = 0.025 ms as |t_AP(dt_3) − t_AP(dt_2)|.
   This must be < 0.05 ms.

5. **Energy proxy (stability indicator):** For each dt, compute |V_m(t = 300 ms) − (−65 mV)|.
   At dt = 0.05 ms this may show instability; document whether it does. If V_m oscillates or
   diverges at dt = 0.05 ms, record the failure mode.

**Expected convergence pattern:** Spike timing error should scale as O(dt) from dt_1 to dt_3,
consistent with first-order exponential Euler. If the error scales faster than linear, the
Crank-Nicolson cable solve (second-order) is dominating; that is acceptable. Non-monotonic
behavior (error larger at dt_2 than dt_3) is a red flag requiring investigation before proceeding.

**Red flag:** If spike timing at dt = 0.025 ms deviates from dt_ref by > 0.2 ms, do not proceed
to production. Investigate gate initialization and integration method before continuing.

---

### 1B. Compartment Count (Spatial) Convergence Study

**Objective:** Confirm that the spatial discretization of the cable does not distort spike timing,
LFP spatial profile, or gate variable dynamics at N_comp = 10 compartments.

**Physical basis:** The electrotonic length constant for a thalamic cell soma/dendrite is
λ ≈ 300–500 µm (R_m ~ 20 kΩ·cm², R_i ~ 100 Ω·cm, diameter ~ 10 µm). The cable theory
requirement is compartment length ≤ λ/10 = 30–50 µm. For a 500 µm cable, N_comp = 10 gives
50 µm per compartment — at the boundary of the requirement. N_comp = 20 gives 25 µm — safely
within. This study confirms the boundary case is adequate.

**Cable geometry (identical across N_comp values):**
- Morphology: single straight cable representing soma + proximal dendrite
- Total length: 500 µm
- Diameter: 10 µm (soma-equivalent; uniform for this test)
- Channels: full set (I_Na, I_K, I_L, I_T, I_h, I_NaP) in all compartments
- dt: 0.025 ms (production value; fixed from Experiment 1A result)
- Stimulus: 5 ms current step at t = 120 ms, 0.5 nA injected at compartment 0 (proximal end)
- Simulation duration: 300 ms
- LFP observation points: 5 points at distances [100, 200, 500, 1000, 2000] µm from cable midpoint,
  perpendicular to the cable axis

**Compartment counts to test:**

| Level | N_comp | Compartment length (µm) | Role |
|-------|--------|-------------------------|------|
| N_1 | 5 | 100 | Under-resolved (expected to fail) |
| N_2 | 10 | 50 | Production minimum (must pass) |
| N_3 | 20 | 25 | Reference solution |
| N_4 | 40 | 12.5 | Over-resolved check (confirms N_3 is converged) |

**Observables and convergence metrics:**

1. **Spike timing at compartment 0:** |t_AP(N) − t_AP(N_4)| < 0.1 ms at N = 10.

2. **LFP spatial profile RMS:** At each of the 5 observation distances, compute:
   ε_LFP(N) = |φ(r, N) − φ(r, N_4)| / |φ(r, N_4)|
   Criterion: ε_LFP(N = 10) < 0.05 (5%) at all 5 distances.

3. **Gate variable profiles:** At peak AP time, compute the spatial profile of m_T(x) along the
   cable for N_2, N_3, N_4. Compute RMS difference normalized by range. Criterion: < 2% at N = 10
   (advisory; confirms no spatial resonance artifacts).

4. **Richardson estimate for LFP:** Using N_2, N_3, N_4, compute the estimated discretization
   error in φ at r = 500 µm as |φ(N_3) − φ(N_2)| / |φ(N_3)|. This should be < 0.02 (2%).

**Expected behavior:** LFP profiles should converge from below as N increases (coarser
discretization underestimates the LFP morphology-dependence). The N_1 = 5 case is expected to
show > 10% LFP error, confirming that the convergence study is sensitive. If N_1 passes (< 5%
error), the spatial convergence criterion is too loose and should be tightened.

**Output:** A table reporting ε_LFP at all 5 distances for all 4 compartment counts, plus the
Richardson estimate. This table is the permanent spatial convergence record for Phase 2.

---

## Experiment 2: LTS Threshold Measurement Protocol

### Objective

Measure the membrane potential at which the I_T-mediated low-threshold spike (LTS) is triggered
upon release from hyperpolarization, and confirm it lies within −65 ± 5 mV.

### Physical Basis

The LTS arises because I_T inactivation gate h_T recovers (de-inactivates) during hyperpolarization.
The half-inactivation voltage is h_T_∞ = 0.5 at V ≈ −80 mV. When the cell is released from
holding at −90 mV, h_T recovers to near 1.0. Upon rebound, I_T activates (m_T rises as V
crosses −57 mV), generating a regenerative inward Ca²⁺ current that produces a broad
depolarizing envelope. Superimposed on this envelope, I_Na/I_K generate fast action potentials
(the burst crown).

The LTS threshold is the lowest voltage at which this rebound depolarization becomes regenerative.
It depends jointly on h_T deinactivation (requires sufficiently deep hyperpolarization and
sufficient holding time) and the m_T activation curve.

### Protocol

**Step 1: Steady-state initialization.**

Initialize the model to V = −65 mV with all gates at their −65 mV steady-state values. Allow
100 ms settling with no external current.

**Step 2: Hyperpolarizing holding step.**

Inject a constant holding current I_hold to bring V_m to −90 mV (confirmed by checking
V_m(t = 600 ms) is within ±1 mV of −90 mV). Hold for 500 ms. The 500 ms holding time ensures
h_T reaches ≥ 95% of its steady-state de-inactivated value (given tau_h_T ~ 50–100 ms at
−90 mV after Q10 correction).

To find I_hold: use a bisection search in the range [−0.5, −0.1] nA to achieve V_m = −90 ± 0.5 mV
at t = 600 ms. Document the required I_hold for the parameter record.

**Step 3: Release from hyperpolarization.**

At t = 600 ms, set I_hold = 0. Do not inject any additional current. Record V_m for 200 ms
post-release.

**Step 4: LTS threshold measurement.**

The LTS threshold V_LTS is defined as the most negative V_m at which dV/dt first crosses
+10 mV/ms during the rebound depolarization (this marks the onset of the regenerative I_T
upswing, not an action potential — action potentials have dV/dt > 100 mV/ms).

Measurement procedure:
- Compute dV_m/dt numerically (central differences at dt = 0.025 ms)
- Find the first time t* where dV/dt > 10 mV/ms AND V_m < −40 mV (to exclude AP contributions)
- V_LTS = V_m(t*) is the threshold voltage

**Criterion:** −70 mV ≤ V_LTS ≤ −60 mV (i.e., −65 ± 5 mV).

**Step 5: Holding-depth sensitivity scan.**

Repeat the protocol with 5 holding voltages to confirm that V_LTS is not sensitive to the precise
holding depth and that a deeper holding is indeed required for LTS to appear:

| Hold voltage | Expected outcome |
|-------------|-----------------|
| −75 mV | LTS may appear (borderline de-inactivation) |
| −80 mV | LTS appears; threshold should be similar to −90 mV hold |
| −85 mV | LTS appears; use as main measurement point if available |
| −90 mV | LTS appears; primary measurement point |
| −95 mV | LTS appears; confirms insensitivity to deeper hyperpolarization |

For each hold depth, report: whether LTS appeared (yes/no), V_LTS if yes, burst spike count.

**Step 6: Holding duration sensitivity scan.**

At the −90 mV holding voltage, repeat with holding durations of [100, 200, 500, 1000] ms.

- At 100 ms: h_T is only partially de-inactivated (~80%); LTS may be smaller or absent.
- At 500 ms and beyond: h_T should be fully de-inactivated; results should be stable.

Report: LTS present/absent, V_LTS, and peak LTS amplitude for each duration. The main result
must use the 500 ms duration.

**Step 7: Confirm burst crown.**

Within the LTS envelope (between V_LTS and return to baseline), count the number of fast action
potentials. Their instantaneous frequency (inverse of inter-spike interval) should be 100–400 Hz.
Report the mean and range of instantaneous frequencies within the burst.

**Output artifacts:**
- V_m(t) trace for the primary protocol (−90 mV hold, 500 ms)
- dV/dt(t) trace showing the threshold crossing
- Table of V_LTS and burst spike count vs. holding voltage and vs. holding duration
- Value of I_hold required to maintain −90 mV

---

## Experiment 3: Spindle Oscillation Protocol

### Objective

Demonstrate spontaneous spindle-frequency (7–14 Hz) oscillations driven by the I_T/I_h loop in
the single-cell model, and measure the spindle recurrence frequency.

### Physical Basis

In the isolated single-cell context, spindle-frequency oscillations emerge from the interaction
between I_T (fast, burst-generating) and I_h (slow, hyperpolarization-activated). After each
LTS burst, I_h activates (it responds to hyperpolarization and is slow to deactivate), producing
a slow inward current that de-activates as V_m depolarizes, creating a negative-feedback loop
with a natural period of ~70–140 ms (7–14 Hz). In the network context, spindles involve TRN-relay
interactions; in the single-cell context, the same I_T/I_h loop produces intrinsic "bursting"
with spindle-like period.

Note: The single-cell intrinsic oscillation is a model validation of the mechanism, not a
full network spindle. Full network spindles require multiple cells and coupling (Phase 2).

### Protocol

**Step 1: Bias current injection.**

Apply a sustained depolarizing bias current I_bias to keep the cell in a sub-threshold but
excitable state (V_m ≈ −65 to −60 mV at rest). This replaces the tonic excitatory input that
relay cells receive from cortex in vivo. Use I_bias = 0.05 nA as the starting value; adjust
as needed so that the cell is near threshold but does not fire tonically.

Scan I_bias in [0.02, 0.05, 0.08, 0.12] nA. For each value, record whether the cell: (a) silent,
(b) tonic firing, or (c) burst mode with spindle-like recurrence. Identify the I_bias range
producing burst mode.

**Step 2: Simulation duration.**

I_h has time constants of 100–500 ms. To observe at least 5 complete spindle cycles at 7 Hz
(period = 143 ms), the simulation must run for at least 5 × 143 = 715 ms after settling.

Run duration: 2000 ms total (100 ms settling + 1900 ms recording at the I_bias value from Step 1
that produces burst mode). This is long enough to observe 7–14 full spindle cycles.

**Step 3: Burst detection.**

Define a burst as any epoch where V_m > −40 mV for at least 2 ms (i.e., at least one action
potential is present). Detect bursts using a threshold-crossing algorithm:

1. Smooth V_m(t) with a 2 ms boxcar filter (to merge intra-burst spikes into a single burst event)
2. Apply threshold at −40 mV
3. Record burst onset times {t_burst,1, t_burst,2, ...}
4. Spindle period T_sp = mean inter-burst interval = mean(diff(t_burst))
5. Spindle frequency f_sp = 1 / T_sp

**Criterion:** 7 Hz ≤ f_sp ≤ 14 Hz.

**Step 4: Spindle frequency power spectrum.**

Compute the power spectral density of V_m(t) using scipy.signal.welch with:
- Window: Hann
- Segment length: 512 samples at dt = 0.025 ms → 12.8 ms window
- Overlap: 50%
- Frequency resolution: ≈ 78 Hz (too coarse for 7–14 Hz; use longer segments)

Revised parameters for spindle band:
- Segment length: 4096 samples → 102.4 ms
- Overlap: 75%
- Frequency resolution: ≈ 9.8 Hz — marginal; use 8192 samples (204.8 ms) for 4.9 Hz resolution

At 4.9 Hz resolution, the 7–14 Hz band spans ~1.5 frequency bins. This is the minimum acceptable
resolution. Report:
- Peak frequency in 3–20 Hz band
- Peak power spectral density value (µV²/Hz)
- Ratio of peak power to mean background power in 20–50 Hz band (should exceed 5:1)

**Step 5: Sensitivity to I_h maximal conductance.**

Vary g_h_max at [0, 50%, 100%, 150%] of the nominal value. Expected outcomes:
- g_h = 0: No spindle oscillation (I_h absent); cell produces irregular bursts only
- g_h at 50%: Spindle period longer (lower frequency; I_h de-activation is slower when weaker)
- g_h at 100%: Target 7–14 Hz
- g_h at 150%: Spindle period shorter (higher frequency)

This scan validates that the oscillation mechanism is correctly I_h-dependent.

**Red flag:** If spindle frequency > 20 Hz, I_h is too fast (Q10 overcorrection) or g_h_max is
too high. If spindle frequency < 5 Hz or absent, I_h may be missing or tau_h is too long.

**Output artifacts:**
- V_m(t) trace for 2000 ms showing at least 5 burst episodes
- Burst onset times and inter-burst intervals
- PSD of V_m, annotated with spindle peak frequency
- Table of f_sp vs. I_bias and vs. g_h_max

---

## Experiment 4: Brian2 vs. NEURON Cross-Validation Protocol

### Objective

Confirm that the Brian2 SpatialNeuron implementation and the NEURON reference model produce
spike times agreeing to < 0.1 ms RMS and waveforms agreeing to < 0.1 mV RMS, for identical
parameter sets and identical stimuli.

### Reference Model

NEURON reference: ModelDB accession 279 (McCormick & Huguenard 1992 relay cell). Download
directly from ModelDB; do not retype the HOC or MOD files. This is the authoritative parameter
source for this cross-validation.

**Before running:** Confirm ModelDB 279 includes: I_Na, I_K, I_L, I_T, I_h, I_NaP. If I_NaP
is absent in ModelDB 279, use the McCormick & Huguenard (1992) paper Appendix to add it to the
NEURON model, and document the modification.

### Parameter Matching Protocol

All parameters in the Brian2 model must be set to exactly the values in the NEURON ModelDB 279
files. Record the following parameter values from the MOD files before Brian2 implementation:

| Parameter | Source (file/line in ModelDB 279) | Value used in Brian2 |
|-----------|-----------------------------------|----------------------|
| g_Na_max | itc_naf.mod or equivalent | (fill in) |
| g_K_max | itc_kdr.mod or equivalent | (fill in) |
| g_L | itc_pas.mod or equivalent | (fill in) |
| g_T_max | itc_it.mod or equivalent | (fill in) |
| g_h_max | itc_ih.mod or equivalent | (fill in) |
| g_NaP_max | itc_inap.mod or equivalent | (fill in) |
| E_Na | (file) | (fill in) |
| E_K | (file) | (fill in) |
| E_Ca | (file) | (fill in) |
| E_L | (file) | (fill in) |
| E_h | (file) | (fill in) |
| C_m | (file) | (fill in) |
| R_a | (file) | (fill in) |
| T_ref | (file) | (fill in) |
| Q10_T | (file) | (fill in) |
| Cable length | (file) | (fill in) |
| Cable diameter | (file) | (fill in) |
| N_compartments | (NEURON nseg) | (fill in) |

This parameter record is a required artifact of Phase 1.

### Stimulus Protocol

The identical stimulus is applied to both models:

- Simulation duration: 500 ms
- Settling: 100 ms (no current; confirm resting V_m = −65 ± 1 mV in both models at t = 100 ms)
- Protocol A (AP waveform test): At t = 100 ms, apply a 5 ms, 1.0 nA suprathreshold current step
  to the soma. Record V_m at the soma for the full 500 ms.
- Protocol B (LTS test): At t = 100 ms, apply −0.3 nA for 500 ms (hyperpolarization to ~ −90 mV),
  then remove at t = 600 ms and record rebound LTS. (Use total duration 800 ms for this protocol.)
- Protocol C (tonic bias test, optional): Apply 0.05 nA sustained; run 2000 ms; record burst pattern.

### Comparison Metrics

**For Protocol A (AP waveform):**

1. **Spike time agreement:** Extract the time of the first AP peak from each model. Compute
   |t_AP,Brian2 − t_AP,NEURON|. Criterion: < 0.1 ms.

2. **Waveform RMS deviation:** Interpolate both V_m traces to a common 0.005 ms grid (using
   linear interpolation). Compute the waveform RMS over the 50 ms epoch centered on the AP:
   RMS = sqrt(mean((V_Brian2(t) − V_NEURON(t))^2))
   Criterion: < 0.1 mV.

3. **AP peak voltage:** |V_peak,Brian2 − V_peak,NEURON| < 1.0 mV (advisory).

4. **AP half-width:** |HW_Brian2 − HW_NEURON| < 0.05 ms (advisory). Half-width defined as AP
   duration at (V_peak + V_threshold)/2.

**For Protocol B (LTS test):**

5. **LTS threshold voltage:** |V_LTS,Brian2 − V_LTS,NEURON| < 2 mV. This tests whether the I_T
   gating is correctly implemented in both models.

6. **Burst spike count:** Both models must produce the same number of spikes in the LTS burst ±1.

**Data management:** Save all V_m(t) traces from both models to disk in a common format
(e.g., numpy .npz or HDF5 with keys "t_ms", "V_Brian2_mV", "V_NEURON_mV"). These are the
permanent validation artifacts.

### Tolerances and Go/No-Go

| Metric | Criterion | Go/No-Go |
|--------|-----------|----------|
| Spike time agreement (Protocol A) | < 0.1 ms | Go/No-Go |
| Waveform RMS (Protocol A) | < 0.1 mV | Go/No-Go |
| LTS threshold agreement (Protocol B) | < 2 mV | Go/No-Go |
| Burst spike count agreement | ±1 spike | Advisory |

If spike timing deviates by 0.1–0.5 ms: first check unit conversions (nA vs. µA, ms vs. s,
mV vs. V) before concluding there is a physics discrepancy. A 10× factor error in conductance
units is the most common cause. If waveform RMS passes but peak voltage deviates > 2 mV:
check E_Na and E_Ca reversal potentials.

---

## Experiment 5: LFP Green's Function Point-Source Test

### Objective

Validate that the JAX-GPU LFP kernel φ(r) = (1/4πσ) Σ_j I_m,j / |r − r_j| produces the
correct analytical solution for a single point source to < 1% relative error at three test
distances, using float64 precision.

### Analytical Reference

For a single point current source of amplitude I at the origin in a homogeneous isotropic medium
of conductivity σ:

φ(r) = I / (4π σ r)

With σ = 0.33 S/m and I = 1 nA = 1 × 10⁻⁹ A:

| r | φ_analytical |
|---|-------------|
| r = 0.1 mm = 100 µm | φ = 1e-9 / (4π × 0.33 × 0.0001) = **2.41 µV** |
| r = 1.0 mm | φ = 1e-9 / (4π × 0.33 × 0.001) = **0.241 µV** |
| r = 10.0 mm | φ = 1e-9 / (4π × 0.33 × 0.010) = **0.0241 µV** |

These values are derived from first principles and must be reproduced to < 1% by the kernel.

**Numerical pre-check:** Before running the kernel, verify by hand (or in a one-line Python
calculation) that these three values are correct. They should be memorized as the primary
calibration values for this project.

### Test Setup

**Single-source geometry:**
- Source position: r_src = (0, 0, 0) m
- Source current: I_src = 1 × 10⁻⁹ A (1 nA, outward-positive convention)
- Observation points: 3 points along the x-axis:
  - r_obs,1 = (0.0001, 0, 0) m → r = 100 µm
  - r_obs,2 = (0.001, 0, 0) m → r = 1 mm
  - r_obs,3 = (0.010, 0, 0) m → r = 10 mm
- σ = 0.33 S/m (exactly as locked in CONVENTIONS.md)
- Precision: float64 (JAX default is float32; must explicitly pass dtype=jnp.float64 and
  enable 64-bit mode with jax.config.update("jax_enable_x64", True))

**Kernel call:**
```python
import jax
jax.config.update("jax_enable_x64", True)
import jax.numpy as jnp

sigma = 0.33  # S/m
I_src = jnp.array([1e-9])  # A
r_src = jnp.array([[0.0, 0.0, 0.0]])  # m
r_obs = jnp.array([[1e-4, 0.0, 0.0],
                   [1e-3, 0.0, 0.0],
                   [1e-2, 0.0, 0.0]])  # m

# Distance tensor: shape (N_obs, N_src)
diffs = r_obs[:, None, :] - r_src[None, :, :]  # (N_obs, N_src, 3)
r_dist = jnp.linalg.norm(diffs, axis=-1)       # (N_obs, N_src)

# Apply minimum distance cutoff for singularity handling (not needed here; r_min = 1e-6 m)
r_dist_safe = jnp.maximum(r_dist, 1e-6)

# Green's function tensor G_ij = 1 / (4 pi sigma r_ij)
G = 1.0 / (4.0 * jnp.pi * sigma * r_dist_safe)  # (N_obs, N_src)

# LFP at observation points
phi = G @ I_src  # (N_obs,); units: V
phi_uV = phi * 1e6  # convert to µV
```

**Expected output:**
```
phi_uV = [2.41, 0.241, 0.0241]  # µV, at r = [100 µm, 1 mm, 10 mm]
```

### Convergence and Precision Metrics

**Primary metric:** Relative error at each observation point:
ε(r_obs,k) = |φ_kernel(r_obs,k) − φ_analytical(r_obs,k)| / |φ_analytical(r_obs,k)|

Criterion: ε < 0.01 (1%) at all three distances.

**Float32 vs. float64 comparison:**

Run the same computation in float32 and float64. The float32 result should agree with float64
to better than 1e-5 relative error at r ≥ 100 µm (float32 has ~7 significant digits; at
r = 100 µm, φ ≈ 2.4 µV — well above float32 precision floor). Document the float32 result and
confirm it also passes the 1% criterion at the three test distances.

**Sign convention check:**

Outward-positive I_src = +1 nA must produce φ > 0 at all observation points (positive source
in a conductive medium produces positive potential at all distances in free space). If φ < 0
for positive I_src, there is a sign error in the kernel. This is a hard go/no-go.

**Singularity guard check:**

Add a fourth test: place an observation point at r_obs,4 = (1e-6, 0, 0) m (r = 1 µm, exactly
at the singularity cutoff). The kernel should return a finite value (not NaN or Inf), and it
should be approximately I/(4πσ × 1e-6) ≈ 241 V (very large but finite). This tests the guard.

**Cross-validation with LFPy:**

If LFPy is available, run the same geometry through LFPy's PointSourcePotential class and compare
to the JAX result. Criterion: agree to < 0.1% (the two implementations should be numerically
identical up to floating-point rounding). Document whether LFPy was available and used.

### Multi-Source Superposition Test

**Objective:** Verify linearity of the kernel (two sources of equal and opposite current produce
φ = 0 at a far-field point midway between them on the perpendicular bisector, for a balanced
dipole).

Setup:
- Source 1: I = +1 nA at r_src,1 = (0, 0, −0.001) m (1 mm below origin)
- Source 2: I = −1 nA at r_src,2 = (0, 0, +0.001) m (1 mm above origin)
- Observation at far field: r_obs = (0.010, 0, 0) m (10 mm from midpoint, perpendicular)

By dipole superposition, φ should be very small (dipole cancellation, ~1000× smaller than
single-source). Compute the exact analytical value:
φ_dipole = I/(4πσ) × [1/|r_obs − r_src,1| − 1/|r_obs − r_src,2|]

With r_obs = (10, 0, 0) mm, r_src,1 = (0, 0, −1) mm, r_src,2 = (0, 0, +1) mm:
|r_obs − r_src,1| = sqrt(100 + 1) mm = 10.05 mm
|r_obs − r_src,2| = sqrt(100 + 1) mm = 10.05 mm (by symmetry → exact cancellation)

So φ_dipole = 0 exactly. The kernel should return < 10⁻¹² V (numerical noise only). Report the
actual value; it should be < 1e-15 V (machine epsilon × φ_single_source).

### Output Artifacts

- Printed table: ε(r_obs,k) for float64 and float32 at all three primary distances
- Sign convention verification result (pass/fail)
- Singularity guard result (finite value at r = 1 µm)
- LFPy cross-check result (if available)
- Dipole cancellation result (≈ 0 at perpendicular bisector)

---

## Experiment 6: Brian2CUDA GPU Stability Test

### Objective

Confirm that Brian2CUDA correctly runs the multicompartment SpatialNeuron + gap junction model
on GPU, producing spike times within 0.1 ms of the CPU (Brian2) reference for a 1-second
simulation at dt = 0.025 ms.

### Test System Design

**Rationale for N = 2:** Pairwise gap junction interaction is the minimal non-trivial case
requiring inter-cell communication. This tests: (a) SpatialNeuron cable solve on GPU,
(b) gap junction Synapses object on GPU, and (c) spike detection consistency between GPU and CPU.

**Cell model:**
- Two identical HH cable cells (10 compartments each, identical geometry and parameters)
- Same parameters as Experiment 1A/1B
- Gap junction: Synapses object coupling soma compartment of cell 0 to soma compartment of cell 1
  with g_j = 1 nS (within Landisman 2002 range of 0.1–2 nS)
- Gap junction current in outward-positive convention: I_gap,i = g_j × (V_soma,i − V_soma,j)
  added to the right-hand side of cell i's somatic compartment equation

**Stimulus:**
- Cell 0: suprathreshold current pulse 0.5 nA for 5 ms at t = 200 ms
- Cell 1: no direct current injection; should respond only via gap junction coupling
- No additional current to either cell after t = 205 ms

**Run parameters:**
- Simulation duration: 1000 ms (1 second)
- dt = 0.025 ms (40,000 timesteps)
- CPU run: Brian2 with default Python runtime (no device set)
- GPU run: Brian2CUDA with `set_device('cuda_standalone', directory='output_cuda')`
- Both runs from identical initial conditions (same seed; both cells at V = −65 mV steady-state)

### Setup Verification Steps

Before running the full 1-second test, run three quick setup checks (< 1 minute each):

**Check A: Zero-gap-junction baseline.**
Run both cells for 100 ms with g_j = 0 (no coupling). Cell 0 should produce exactly the same
spike pattern as a single isolated cell. Cell 1 should remain at resting potential. Both CPU and
GPU should agree to < 0.01 ms spike time difference. This confirms the gap junction Synapses
object adds zero current when g_j = 0.

**Check B: Equal-voltage zero-current verification.**
Initialize both cells to identical V_m and gate values. Run for 10 ms with no current injection.
Record the gap junction current I_gap,0 = g_j × (V_soma,0 − V_soma,1). It should be < 10⁻¹⁵ A
(numerical zero) at all timesteps. This confirms the gap junction convention (Pitfall 5 from
RESEARCH.md).

**Check C: Coupling coefficient verification.**
Inject a step of 0.1 nA into cell 0 only; allow to reach steady state (100 ms). Measure
V_soma,0 and V_soma,1. The coupling coefficient κ = (V_soma,1 − V_rest) / (V_soma,0 − V_rest)
should equal g_j / (g_j + g_soma) where g_soma is the total membrane conductance of the somatic
compartment. For g_j = 1 nS and g_soma ~ 10 nS (from R_m and area), κ ≈ 0.09 (8–10%). Report
both the measured κ and the theoretical value.

### Spike Comparison Protocol (1-second run)

**Spike detection:** A spike is recorded in a cell when V_m at the somatic compartment crosses
0 mV from below. Record spike times to 0.001 ms precision.

**Comparison metrics:**

1. **Spike count agreement:** Both CPU and GPU runs must produce the same number of spikes in each
   cell. Criterion: spike count must be identical. If spike counts differ, this is a GPU
   instability indicator and the test fails.

2. **Spike time agreement:** For each matched spike pair (nth spike in CPU matched to nth spike in
   GPU), compute |t_n,CPU − t_n,GPU|. Report the maximum and RMS of all differences.
   Criterion: max |Δt_n| < 0.1 ms, RMS < 0.05 ms.

3. **Subthreshold voltage agreement:** At 10 evenly-spaced time points between spikes, compare
   V_soma from CPU and GPU. Criterion: max |V_CPU − V_GPU| < 0.5 mV (advisory; numerical
   differences between CPU double and GPU float may cause small differences in subthreshold V).

4. **Gap junction current agreement:** At 10 evenly-spaced time points, compare I_gap from CPU
   and GPU. Criterion: max |I_gap,CPU − I_gap,GPU| / I_gap,peak < 0.01 (1% of peak gap current).

**Performance benchmark (optional, advisory):**
Record wall-clock time for 1-second simulation on CPU vs. GPU. At N = 2 cells, GPU is likely
slower than CPU due to overhead. Report the crossover point if possible (what N causes GPU to
be faster than CPU?). This informs Phase 2 resource planning.

### Failure Recovery

If GPU and CPU spike times differ by > 0.1 ms:
1. First check: Does the GPU simulation produce NaN or Inf in any state variable? If so, reduce
   dt to 0.010 ms and repeat (possible GPU floating-point divergence).
2. Check: Are the gap junction currents the same on CPU and GPU? If not, the Synapses ordering
   (gap current before vs. after cable solve) may differ between CPU and GPU code generation.
3. If step 2 fails: disable the gap junction (g_j = 0) and retest. If CPU/GPU now agree,
   the issue is in the gap junction implementation only.
4. If all checks fail: document the failure mode; fall back to CPU-only for Phase 1 validation.
   This does NOT block Phase 1 (which is single-cell). Brian2CUDA must be resolved before Phase 2.

**Output artifacts:**
- Spike time tables for both cells, both CPU and GPU
- Coupling coefficient κ from Check C
- Wall-clock time for 1-second run on CPU and GPU
- Any failure mode documentation

---

## Error Budget

### Dominant Error Sources by Experiment

| Experiment | Dominant Error Source | Type | Estimated Magnitude | Controlled By | Blocking? |
|------------|----------------------|------|---------------------|---------------|-----------|
| 1A (dt convergence) | Exponential Euler O(dt) truncation on gate variables | Systematic | ~0.05 ms at dt = 0.025 ms | Use dt = 0.025 ms; confirmed by Richardson | No (< 0.1 ms criterion) |
| 1A (dt convergence) | Forward coupling of cable-to-gate at each step | Systematic | Absorbed into O(dt) term | Operator splitting; exponential Euler | No |
| 1B (spatial convergence) | Compartment discretization of current density | Systematic | < 5% LFP at N = 10 | Confirmed by N_4 = 40 reference | No |
| 1B (spatial convergence) | Point-source approximation per compartment | Systematic | < 1% for 50 µm compartments | λ/10 criterion satisfied | No |
| 2 (LTS threshold) | Q10 temperature correction error | Systematic | ±2 mV LTS shift if Q10 wrong | Verify T_ref from primary paper | Potentially blocking |
| 2 (LTS threshold) | Holding duration (insufficient h_T de-inactivation) | Protocol | < 2 mV if hold < 300 ms | Use 500 ms hold (> 5× tau_h) | No |
| 3 (spindle) | I_h time constant Q10 error | Systematic | ±2 Hz spindle shift if tau_h wrong | Verify Q10 for I_h separately | No (within 7–14 Hz band) |
| 3 (spindle) | Stimulus I_bias not at optimal value | Protocol | ±3 Hz | Scan 4 I_bias values | No |
| 4 (Brian2 vs. NEURON) | Parameter mismatch (units, Q10) | Systematic | Up to 1 ms spike offset if wrong | Parameter table (fill before coding) | Blocking if > 0.1 ms |
| 4 (Brian2 vs. NEURON) | Different numerical integration methods | Systematic | ~0.02–0.05 ms | Both using exponential Euler | No |
| 5 (LFP kernel) | Float64 arithmetic error | Numerical | < 1e-14 relative | Float64 enabled in JAX | No |
| 5 (LFP kernel) | Singularity cutoff (1 µm) | Systematic | < 0.01% at r ≥ 100 µm | r_min = 1 µm; test at 3 distances | No |
| 6 (Brian2CUDA) | GPU float32 vs. CPU float64 | Numerical | < 0.5 mV subthreshold | Advisory only; spikes are discrete | Advisory |
| 6 (Brian2CUDA) | Gap junction current ordering (CPU vs. GPU) | Algorithmic | Could cause > 0.1 ms | Check B in setup verification | Blocking if present |

### Systematic vs. Statistical Error

All experiments in Phase 1 are deterministic (no stochastic dynamics). There are no statistical
errors. All errors are systematic (algorithmic, numerical, or parameter-matching errors).

**Error hierarchy:**
1. Parameter errors (wrong Q10, wrong reversal potential, wrong unit conversion): largest,
   can invalidate all results. Prevented by the parameter table in Experiment 4 and the
   explicit Q10 formulas below.
2. Numerical integration errors: moderate, bounded by convergence studies in Experiment 1A/1B.
3. Floating-point precision: negligible for float64; verified in Experiment 5.

### Q10 Correction Reference Values

These values must be used in all implementations. Any deviation is a parameter error.

| Current | Q10 | T_ref (°C) | T_target (°C) | tadj = Q10^(ΔT/10) |
|---------|-----|-----------|--------------|---------------------|
| I_T gating (m_T, h_T) | 2.5 | 24 | 37 | 2.5^1.3 ≈ 3.40 |
| I_h gating (m_h) | 2.5 | 24 | 37 | 2.5^1.3 ≈ 3.40 |
| I_Na, I_K (standard HH) | 3.0 | 6.3 | 37 | 3.0^3.07 ≈ 28.0 |

Note: The Na/K Q10 and T_ref come from the original Hodgkin-Huxley (1952) paper (squid axon at
6.3°C). If the NEURON ModelDB 279 uses a different T_ref for Na/K, use the ModelDB values for
consistency with Experiment 4 (cross-validation).

**Application formula:**
tau_corrected = tau_measured / tadj
rate_corrected = rate_measured × tadj

---

## Computational Cost Estimate

All Phase 1 simulations involve a single cell (N_comp = 10–40) for durations of 300–2000 ms.
These are trivially fast on any modern CPU. No GPU is required for Phase 1 HH simulations
(GPU testing is isolated to Experiment 6).

| Experiment | N_runs | Sim duration | Est. wall time/run | Total |
|------------|--------|--------------|--------------------|----|
| 1A (dt convergence) | 4 | 300 ms | < 1 s | < 5 s |
| 1B (spatial convergence) | 4 | 300 ms | < 2 s | < 10 s |
| 2 (LTS threshold, including scans) | ~15 | 700–900 ms | < 3 s | < 1 min |
| 3 (spindle, including I_bias scan) | ~8 | 2000 ms | < 5 s | < 1 min |
| 4 (Brian2 vs. NEURON, 3 protocols) | 6 (3 protocols × 2 simulators) | 500–2000 ms | < 10 s | < 2 min |
| 5 (LFP kernel) | 1 | instantaneous | < 1 s | < 5 s |
| 6 (Brian2CUDA, 1-second run) | 2 (CPU + GPU) | 1000 ms | 10–120 s (GPU) | < 5 min |
| **Total Phase 1** | | | | **< 15 min wall clock** |

The total compute budget is negligible. There is no cost constraint on Phase 1. Efficiency
concerns do not apply. All experiments should be run to completion without shortcuts.

---

## Execution Order and Dependencies

```
Stage 0: Prerequisites (before any simulation)
  ├── Download ModelDB accession 279 (NEURON MOD files)
  ├── Download ModelDB accession 3670 (Destexhe 1994, optional reference)
  ├── Fill in parameter table in Experiment 4 from ModelDB 279 MOD files
  ├── Verify JAX float64 mode: jax.config.update("jax_enable_x64", True)
  ├── Verify Brian2CUDA installation: python -c "import brian2cuda; print('OK')"
  └── Verify NEURON installation: python -c "from neuron import h; print('OK')"

Stage 1: Numerical convergence (Experiments 1A and 1B)
  ├── Run Experiment 1A (dt convergence): 4 Brian2 runs; produce spike timing table
  ├── Confirm dt = 0.025 ms passes 0.1 ms spike timing criterion
  ├── Run Experiment 1B (spatial convergence): 4 Brian2 runs; produce LFP error table
  ├── Confirm N_comp = 10 passes 5% LFP criterion
  └── CHECKPOINT: Parameters dt and N_comp locked for all downstream experiments

Stage 2: Single-cell physiology (Experiments 2 and 3)
  ├── Run Experiment 2 (LTS threshold): hyperpolarization protocol + scans
  ├── Confirm V_LTS in [-70, -60] mV
  ├── Run Experiment 3 (spindle oscillation): bias scan + 2000 ms production run
  ├── Confirm f_spindle in [7, 14] Hz
  └── CHECKPOINT: Physiology validated against McCormick & Huguenard 1992 benchmarks

Stage 3: Cross-validation (Experiment 4)
  ├── Depends on: Stage 0 (parameter table complete), Stage 1 (dt locked at 0.025 ms)
  ├── Run NEURON model (ModelDB 279) for all 3 protocols
  ├── Run Brian2 model (same parameters) for all 3 protocols
  ├── Compare spike times and waveforms
  └── CHECKPOINT: Brian2 implementation confirmed correct

Stage 4: EM kernel (Experiment 5)
  ├── Independent of Stages 1-3 (no dependency on HH model)
  ├── Run JAX point-source test (< 5 minutes)
  ├── Run dipole cancellation test
  ├── (Optional) Run LFPy cross-check
  └── CHECKPOINT: LFP kernel confirmed correct

Stage 5: GPU stability (Experiment 6)
  ├── Depends on: Stage 1 (dt locked), Stage 3 (gap junction parameters verified)
  ├── Run Setup Checks A, B, C (< 10 minutes total)
  ├── Run 1-second CPU vs. GPU comparison
  └── CHECKPOINT: Brian2CUDA confirmed stable for Phase 2 use

All stages complete → Go/No-Go review for Phase 2 advancement
```

---

## Go/No-Go Criteria for Phase 2 Advancement

Phase 2 requires a confirmed, validated single-cell model and LFP kernel. The following criteria
must ALL be met before Phase 2 begins.

| ID | Criterion | Measurement | Tolerance | Blocking? |
|----|-----------|-------------|-----------|-----------|
| GO-01 | dt convergence | Spike timing at dt=0.025 ms vs dt=0.005 ms | < 0.1 ms | Hard block |
| GO-02 | Spatial convergence | LFP RMS error at N_comp=10 vs N_comp=40 | < 5% at all 5 distances | Hard block |
| GO-03 | LTS threshold | V_LTS measured by hyperpolarization protocol | −70 to −60 mV | Hard block |
| GO-04 | I_T half-activation | m_T∞ = 0.5 at V_half from gate curve | −59 to −55 mV | Hard block |
| GO-05 | Spindle frequency | f_spindle from burst detection on 2000 ms trace | 7–14 Hz | Hard block |
| GO-06 | Intra-burst frequency | Instantaneous frequency within LTS burst | 100–400 Hz | Hard block |
| GO-07 | Brian2 vs. NEURON spike timing | |t_AP,Brian2 − t_AP,NEURON| | < 0.1 ms | Hard block |
| GO-08 | Brian2 vs. NEURON waveform | RMS(V_Brian2 − V_NEURON) over AP epoch | < 0.1 mV | Hard block |
| GO-09 | LFP kernel sign convention | Positive I_src produces positive φ | φ > 0 | Hard block |
| GO-10 | LFP kernel accuracy | ε(r) at 100 µm, 1 mm, 10 mm | < 1% at all 3 | Hard block |
| GO-11 | Brian2CUDA spike count | GPU spike count = CPU spike count | Exact match | Hard block |
| GO-12 | Brian2CUDA spike timing | max |t_n,CPU − t_n,GPU| | < 0.1 ms | Hard block |
| GO-13 | Gap junction zero-current | I_gap at equal voltages | < 10⁻¹⁵ A | Hard block |
| ADV-01 | Spindle frequency PSD | Peak in 3–20 Hz band exceeds background by | > 5:1 | Advisory |
| ADV-02 | AP peak voltage agreement | |V_peak,Brian2 − V_peak,NEURON| | < 1.0 mV | Advisory |
| ADV-03 | Brian2CUDA subthreshold V | |V_CPU − V_GPU| between spikes | < 0.5 mV | Advisory |

**Advisory criteria** that fail do not block Phase 2 but must be documented in SUMMARY.md with
an explanation. Hard block criteria that fail halt Phase 2 until resolved.

**Escalation to /gpd:debug:** If any hard-block criterion fails and the root cause cannot be
identified within 3 diagnostic attempts using the failure-recovery guidance in each experiment
section above, escalate with the following symptom report:

- Expected: [criterion target value and physical meaning]
- Actual: [observed value]
- Reproduction: [exact parameter set, simulation tool, and seed]
- Parameter sensitivity: [does the failure improve/worsen with any single parameter change?]
- What was tried: [list recovery attempts and outcomes]
- Relevant files: [paths to V_m traces, kernel output arrays, spike time tables]

---

## Suggested Task Breakdown (for planner)

| Task | Type | Dependencies | Estimated Complexity |
|------|------|-------------|----------------------|
| Prerequisites: Download ModelDB 279; fill parameter table | setup | none | small |
| Experiment 1A: dt convergence study (4 Brian2 runs) | validate | prerequisites | small |
| Experiment 1B: spatial convergence study (4 Brian2 runs) | validate | Exp 1A | small |
| Experiment 2: LTS threshold protocol (15 runs including scans) | validate | Exp 1B | medium |
| Experiment 3: Spindle oscillation protocol (8 runs + PSD) | validate | Exp 2 | medium |
| Experiment 4: Brian2 vs. NEURON cross-validation (6 runs) | validate | Exp 3, prerequisites | medium |
| Experiment 5: LFP kernel point-source test | validate | prerequisites | small |
| Experiment 6: Brian2CUDA GPU stability test | validate | Exp 4 | medium |
| Final go/no-go table: compile all 13 criteria | analysis | all experiments | small |

---

## Conventions Used

All numerical values in this document use:
- Voltage: mV (V_m, E_ion, V_LTS, V_half)
- Current: nA (I_src, I_hold, I_bias, I_gap)
- Conductance: nS (g_j, g_soma)
- Time: ms (t, dt, tau, T_period)
- Distance: µm in morphology and biology; m in EM kernel (SI for Poisson equation)
- LFP potential: µV (φ) for display; V for kernel computation
- Tissue conductivity: S/m (σ = 0.33)
- Current convention: outward-positive per CONVENTIONS.md
- h gate: fraction NOT inactivated per CONVENTIONS.md
- Temperature: all kinetics at 37°C after Q10 correction per CONVENTIONS.md

---

*Experiment design authored by gpd-experiment-designer for Phase 01.*
*Consumed by gpd-executor for implementation in sub-tasks 01-02 and 01-03.*
