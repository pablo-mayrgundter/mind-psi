# Conventions Ledger

**Project:** Thalamo-Cortical EM Manifold Model
**Created:** 2026-03-16
**Last updated:** 2026-03-16 (Phase 0 — initialization)

> This file is append-only for convention entries. When a convention changes, add a new
> entry with the updated value and mark the old entry as superseded. Never delete entries.
>
> **Single source of truth** for all notation, sign conventions, unit choices, and
> normalization decisions across all project phases. Every equation written in this project
> must be consistent with every entry below. The consistency checker enforces this
> mechanically at every phase boundary.

---

## Summary of Locked Conventions (Quick Reference)

| Category | Convention | Locked In |
|----------|-----------|-----------|
| Primary observable | LFP (local field potential), NOT scalp EEG | Phase 0 |
| Unit system | SI for EM; neural units (mV, ms, nS, µm) for dynamics | Phase 0 |
| Membrane potential sign | V_m = V_intracellular - V_extracellular; resting = -65 mV | Phase 0 |
| HH current sign | Outward-positive; I_ion = g_ion · (gates) · (V_m - E_ion) | Phase 0 |
| HH h variable | h = fraction NOT inactivated; h_∞ ≈ 0.60 at V = -65 mV | Phase 0 |
| Gap junction current | I_gap = g_j · (V_i - V_j); outward-positive from cell i | Phase 0 |
| EM approximation | Quasi-static Poisson: ∇·(σ∇φ) = -∇·J_imp | Phase 0 |
| LFP potential | φ(r) = (1/4πσ) ∫ J(r')·∇'(1/|r-r'|) dV'; σ = 0.33 S/m | Phase 0 |
| Tissue conductivity | σ = 0.33 S/m (homogeneous isotropic gray matter) | Phase 0 |
| "Standing wave" | LFP spatial eigenmode of network dynamics; NOT EM radiation | Phase 0 |
| SVD convention | E_field[:, t] snapshot matrix → U·Σ·V^T; dominant mode = U[:,0] | Phase 0 |
| Fourier (temporal) | f_tilde(ω) = ∫ f(t) e^{-iωt} dt; f(t) = ∫ dω/(2π) f_tilde(ω) e^{+iωt} | Phase 0 |
| Holographic fidelity | ρ = Pearson(F_original, F_reconstructed); pass threshold ≥ 0.70 | Phase 0 |
| Kuramoto order parameter | R = |Σ_k exp(iθ_k)| / N | Phase 0 |
| Simulation timestep | dt ≤ 0.025 ms; never > 0.05 ms | Phase 0 |
| EEG band standard | IFCN: delta 0.5–4, theta 4–8, alpha 8–13, beta 13–30, gamma 30–100 Hz | Phase 0 |

---

## 1. Observable and Field Quantities

### Primary Observable: LFP, Not Scalp EEG

| Field | Value |
|-------|-------|
| **Convention** | The primary comparison observable for the thalamic syncytium model is the **local field potential (LFP)** — the extracellular potential recorded 100–1000 µm from the syncytium. NOT scalp EEG. |
| **Introduced** | Phase 0 |
| **Rationale** | N~100–1000 thalamic cells cannot produce scalp EEG (which requires ~10,000–100,000 synchronously firing pyramidal neurons over cm² patches; Nunez & Srinivasan 2006). LFP is the physically appropriate observable. Scalp EEG comparison is out of scope unless a full forward model (layered sphere) is added; even then, only spectral profiles (not amplitudes) may be compared. |
| **Dependencies** | All amplitude reporting; field comparison against empirical data; PSD comparison targets |
| **Test value** | Field amplitude reported in V/m or µV/mm. Single HH cable neuron: ~0.1–1 mV/mm near-field. LFP from N=100 coupled cells: report absolute value in µV/mm and compare to Linden et al. (2010). |

### Extracellular Potential Symbol and Sign

| Field | Value |
|-------|-------|
| **Convention** | φ(r, t) denotes the extracellular electric potential in volts (V) or microvolts (µV). φ is defined as the quasi-static solution to Poisson's equation in the tissue volume conductor. Positive φ corresponds to net positive charge accumulation in the vicinity of r. |
| **Introduced** | Phase 0 |
| **Rationale** | Standard in computational neuroscience (Nunez & Srinivasan 2006; Hales 2014; Linden et al. 2010). |
| **Dependencies** | LFP forward model formula; electrode recording sign; dipole source orientation |
| **Test value** | Single point current source of magnitude I_0 (A) at origin in uniform medium: φ(r) = I_0 / (4πσ|r|). At r = 1 mm, I_0 = 1 nA, σ = 0.33 S/m: φ = 1×10⁻⁹ / (4π × 0.33 × 10⁻³) ≈ 0.24 µV |

---

## 2. Neural / Membrane Electrophysiology

### Membrane Potential Definition

| Field | Value |
|-------|-------|
| **Convention** | V_m = V_intracellular − V_extracellular. Resting membrane potential ≈ −65 mV. Threshold for action potential: V_m ≈ −55 mV (approximately +10 mV above rest). Peak of action potential: V_m ≈ +40 mV. |
| **Introduced** | Phase 0 |
| **Rationale** | Standard electrophysiology convention. The opposite sign (V_extracellular − V_intracellular) appears in some older texts (Gaussian CGS) and would flip the sign of every ionic current driving force. |
| **Dependencies** | Every ionic current expression (V_m − E_ion); HH gate kinetics α(V), β(V); threshold conditions |
| **Test value** | At rest: V_m = −65 mV → driving force for Na⁺ is V_m − E_Na = −65 − (+55) = −120 mV (inward, consistent with negative I_Na if outward-positive sign is used — see current sign convention). |

### Ionic Current Sign Convention (Outward-Positive)

| Field | Value |
|-------|-------|
| **Convention** | All transmembrane currents are **outward-positive** (from intracellular to extracellular). Outward current is positive; inward current is negative. Inward Na⁺ during action potential upstroke gives negative I_Na. The membrane voltage equation is: C_m dV_m/dt = −I_Na − I_K − I_L − I_T − I_h − I_NaP + I_ext where all I_ion terms carry the outward-positive sign convention and I_ext is an applied external current (positive = depolarizing). |
| **Introduced** | Phase 0 |
| **Rationale** | Hodgkin & Huxley (1952) outward-positive convention; maintained throughout computational neuroscience. Some texts use inward-positive — this project prohibits that usage. |
| **Dependencies** | All HH current equations; cable equation source term; LFP forward model (transmembrane current → extracellular source); Brian2 implementation sign |
| **Test value** | During AP upstroke: I_Na < 0 (inward Na⁺), I_K > 0 (outward K⁺ during repolarization). At peak depolarization (V_m ≈ +40 mV): I_K is near maximal outward. |

### Hodgkin-Huxley Current Equations

| Field | Value |
|-------|-------|
| **Convention** | Standard HH equations with outward-positive current and V_m = V_in − V_ex: I_Na = g_Na · m³ · h · (V_m − E_Na); I_K = g_K · n⁴ · (V_m − E_K); I_L = g_L · (V_m − E_L). Gate variables: m (Na activation), h (Na inactivation; see below), n (K activation). Gate dynamics: dx/dt = α_x(V_m)(1 − x) − β_x(V_m) x for x ∈ {m, h, n}. |
| **Introduced** | Phase 0 |
| **Rationale** | Hodgkin & Huxley (1952) original equations; universally adopted in computational neuroscience. |
| **Dependencies** | All HH simulations; single-cell validation vs. NEURON; Brian2 implementation; reversal potentials E_Na, E_K, E_L |
| **Test value** | Resting state: m_∞ ≈ 0.053, h_∞ ≈ 0.596, n_∞ ≈ 0.318 at V_m = −65 mV. Spike threshold ≈ −55 mV. Peak Na current (inward, negative): I_Na_min < −200 µA/cm² during AP. |

### HH Inactivation Variable h (Fraction NOT Inactivated)

| Field | Value |
|-------|-------|
| **Convention** | h is the fraction of Na⁺ channels that are **NOT inactivated** (i.e., available for opening). At rest: h ≈ 0.6. During Na channel inactivation (after AP peak): h → 0. Convention: I_Na = g_Na · m³ · **h** · (V_m − E_Na), where h = 1 means fully de-inactivated (available), h = 0 means fully inactivated (unavailable). |
| **Introduced** | Phase 0 |
| **Rationale** | This is the original Hodgkin & Huxley (1952) convention. Some texts define h_inactive = 1 − h_HH, which produces opposite dynamics. Use of the non-standard sign would cause h → 0 at rest and h → 1 after inactivation, reversing the available-channel interpretation. This project locks the HH standard. |
| **Dependencies** | I_Na formula; recovery from inactivation (hyperpolarization increases h); thalamic burst mechanism (h recovery in I_T, see below) |
| **Test value** | Starting at V_m = −65 mV: h_∞(−65) ≈ 0.596. After sustained depolarization to −40 mV: h_∞(−40) ≈ 0.05 (nearly fully inactivated). Verify in Brian2: plot h(t) during single AP — h should dip from ~0.6 to near 0, then recover. |

### Reversal Potential Parameters

| Field | Value |
|-------|-------|
| **Convention** | Standard reference values (Nernst equilibrium potentials; may be adjusted in specific models but must be stated): E_Na ≈ +55 mV; E_K ≈ −90 mV; E_L ≈ −70 mV (leak, phenomenological); E_Ca ≈ +120 mV (T-type Ca²⁺ for I_T); E_h ≈ −43 mV (HCN mixed cation current). All in mV; computed as V_m = V_in − V_ex. |
| **Introduced** | Phase 0 |
| **Rationale** | McCormick & Huguenard (1992) thalamic relay cell parameters; standard for thalamo-cortical models. E_h ≈ −43 mV is the empirical HCN reversal potential (mixed Na⁺/K⁺; mildly depolarizing relative to K⁺ resting). |
| **Dependencies** | All ionic current driving forces; Nernst equation; temperature corrections |
| **Test value** | At rest (V_m = −65 mV): driving force (V_m − E_h) = −65 − (−43) = −22 mV → I_h inward (negative in outward-positive convention) when m_h > 0. This is consistent with I_h providing depolarizing drive after hyperpolarization. |

### Maximal Conductance Parameters

| Field | Value |
|-------|-------|
| **Convention** | Maximal conductances (g_bar) in mS/cm². Standard starting values from McCormick & Huguenard (1992), temperature-corrected to 37°C (Q10 method, see Temperature section below): g_Na ≈ 100 mS/cm²; g_K ≈ 80 mS/cm²; g_L ≈ 0.05 mS/cm²; g_T ≈ 0.5–2 mS/cm²; g_h ≈ 0.05–0.2 mS/cm²; g_NaP ≈ 0.02–0.1 mS/cm². These are per-compartment areal conductances, NOT per-cell conductances. |
| **Introduced** | Phase 0 |
| **Rationale** | McCormick & Huguenard (1992) standard parameterization. All values must be verified against their Table 1 / primary source before Phase 1 implementation. |
| **Dependencies** | HH current equations; compartment area calculation; Q10 temperature correction |
| **Test value** | Resting conductance ≈ g_L ≈ 0.05 mS/cm². Input resistance ≈ 1/g_L = 20 MΩ·cm². Spike threshold consistent with VALD-01 (LTS threshold ~−65 mV from hyperpolarized base). |

### Specific Membrane Parameters

| Field | Value |
|-------|-------|
| **Convention** | C_m = 1 µF/cm² (specific membrane capacitance; standard biological value). R_m = 1/g_L (specific membrane resistance in Ω·cm²). R_i = axial resistivity ≈ 100–200 Ω·cm (intracellular; literature range for neural axons/dendrites). |
| **Introduced** | Phase 0 |
| **Rationale** | Koch & Segev (1998) cable theory standard; C_m = 1 µF/cm² is a near-universal biological membrane value. |
| **Dependencies** | Membrane time constant τ_m = R_m · C_m; cable space constant λ; compartment coupling; LFP source term |
| **Test value** | τ_m = R_m · C_m = (1/0.05 × 10⁻³ Ω·m²) × (1 × 10⁻² F/m²) ≈ 20 ms. This is the passive decay timescale below threshold. |

---

## 3. Thalamic-Specific Currents

### T-type Calcium Current (I_T)

| Field | Value |
|-------|-------|
| **Convention** | I_T = g_T · m_T² · h_T · (V_m − E_Ca). m_T: activation gate (increases with depolarization; half-activation ≈ −57 mV). h_T: inactivation gate (h_T = fraction NOT inactivated; same HH convention as Na h variable; half-inactivation ≈ −83 mV). E_Ca ≈ +120 mV. Threshold for low-threshold spike (LTS) onset: ≈ −65 mV from a hyperpolarized holding potential. |
| **Introduced** | Phase 0 |
| **Rationale** | Steriade, McCormick & Sejnowski (1993); McCormick & Huguenard (1992). I_T is mandatory for thalamic burst firing and spindle generation — cannot be omitted. Standard HH inactivation convention (h_T = NOT inactivated). |
| **Dependencies** | LTS threshold validation (VALD-01); burst firing mechanism; I_T half-activation test value |
| **Test value** | LTS threshold ≈ −65 mV from hyperpolarized base (holding at −85 to −90 mV). Burst structure: 3–8 spikes at 100–400 Hz intra-burst (Steriade et al. 1993). I_T half-activation: m_T∞(V = −57 mV) = 0.5. |

### HCN Current (I_h)

| Field | Value |
|-------|-------|
| **Convention** | I_h = g_h · m_h · (V_m − E_h). m_h: activation gate. I_h activates on **hyperpolarization** (m_h increases when V_m decreases below approximately −75 mV). E_h ≈ −43 mV (mixed Na⁺/K⁺ cation current; depolarizing relative to resting K⁺ reversal). At rest (V_m = −65 mV), I_h provides a small depolarizing drive that participates in the spindle rhythm reset. |
| **Introduced** | Phase 0 |
| **Rationale** | McCormick & Huguenard (1992); Steriade et al. (1993). I_h is mandatory for spindle generation — it provides the rebound depolarization that terminates each silent phase of the spindle oscillation. CAUTION: the activation gate m_h increases on hyperpolarization, which is the OPPOSITE polarity from most other gates. Brian2 implementation must reflect this. |
| **Dependencies** | Spindle frequency (7–14 Hz); g_h parameter sweep; tau_h (slow, 50–500 ms); VALD-01 benchmark |
| **Test value** | m_h∞(−65 mV) ≈ 0.1–0.3 (partial activation at rest). m_h∞(−90 mV) ≈ 0.7–0.9 (near full activation at hyperpolarized potential). I_h at V_m = −80 mV, m_h = 0.5: I_h = g_h · 0.5 · (−80 − (−43)) = g_h · 0.5 · (−37) → inward (negative, consistent with depolarizing pull). |

### Persistent Sodium Current (I_NaP)

| Field | Value |
|-------|-------|
| **Convention** | I_NaP = g_NaP · m_NaP · (V_m − E_Na). m_NaP: steady-state activation gate (no separate inactivation variable in the standard minimal model). Half-activation ≈ −50 to −55 mV. Kinetics are much faster than HH Na (millisecond range); effectively instantaneous on spindle timescales. E_Na = +55 mV. |
| **Introduced** | Phase 0 |
| **Rationale** | I_NaP enhances sub-threshold oscillatory activity and assists in burst generation. Included per McCormick & Huguenard (1992) extended model and DERV-03 requirement. Outward-positive convention applies. |
| **Dependencies** | HH cable model (Brian2); sub-threshold resonance; spindle amplitude |
| **Test value** | g_NaP small compared to g_Na (g_NaP ≈ 0.02–0.1 mS/cm² vs. g_Na ≈ 100 mS/cm²). I_NaP provides ~1–5 nA depolarizing current at sub-threshold voltages. Verify that removing I_NaP reduces burst amplitude but does not eliminate spindles. |

### Temperature Correction (Q10 Method)

| Field | Value |
|-------|-------|
| **Convention** | All HH rate constants (α, β) are temperature-corrected from reference temperature T_ref to physiological temperature T = 37°C using the Q10 rule: k(T) = k(T_ref) · Q10^((T − T_ref)/10). McCormick & Huguenard (1992) parameters are specified at T_ref = 36°C. For Na and K HH: Q10 ≈ 3. For I_T and I_h: Q10 ≈ 2.5. Apply correction factor φ_T = Q10^((T−T_ref)/10) to all α and β rate constants. |
| **Introduced** | Phase 0 |
| **Rationale** | Neural dynamics are temperature-sensitive. Uncorrected room-temperature HH parameters give wrong firing rates and thresholds. McCormick & Huguenard (1992) report 36°C; most thalamic recordings are at 36–37°C. |
| **Dependencies** | All HH gate kinetics; Brian2 model equations; VALD-01 comparison to McCormick & Huguenard benchmarks |
| **Test value** | At T_ref = 36°C, Q10 = 3: correction factor for 37°C = 3^(1/10) ≈ 1.116. Spindle frequency should match 7–14 Hz after correction. |

---

## 4. Cable Theory and Compartmental Modeling

### Cable Equation Convention

| Field | Value |
|-------|-------|
| **Convention** | Rall (1969) cable equation in dimensional form: C_m ∂V_m/∂t = (d/(4R_i)) ∂²V_m/∂x² − i_m(V_m, t) + i_ext, where d is cable diameter (µm), R_i is axial resistivity (Ω·cm), i_m is transmembrane current density (µA/cm²; outward-positive), i_ext is applied current density. Electrotonic space constant: λ = √(d·R_m/(4R_i)). Membrane time constant: τ_m = R_m·C_m. |
| **Introduced** | Phase 0 |
| **Rationale** | Rall (1969) standard formulation. Consistent with Brian2 multi-compartment implementation using isopotential segments connected by axial resistances. |
| **Dependencies** | Compartment length constraint (< λ/10); axial current calculation (source term for LFP); dt constraint (active cable stiffness) |
| **Test value** | For d = 2 µm, R_i = 150 Ω·cm, R_m = 20,000 Ω·cm²: λ = √(2 × 10⁻⁴ cm × 20,000 / (4 × 150)) ≈ 258 µm. τ_m = 20,000 × 10⁻⁶ F/cm² = 20 ms. Compartment length < λ/10 ≈ 26 µm. |

### Compartment Length Constraint

| Field | Value |
|-------|-------|
| **Convention** | Each cable compartment length Δx must satisfy Δx < λ_electrotonic / 10 ≈ 25–50 µm for accurate voltage propagation. Minimum ≥ 10 compartments per HH cable cell (SIMU-01 requirement). |
| **Introduced** | Phase 0 |
| **Rationale** | Accuracy criterion for compartmental cable approximation. Violating this produces incorrect propagation velocities and voltage attenuation. |
| **Dependencies** | Brian2 model construction; LFP source term discretization; simulation memory |
| **Test value** | For λ ≈ 250 µm: Δx_max ≈ 25 µm → minimum 10 compartments for 250 µm cable. |

---

## 5. Gap Junction Coupling

### Gap Junction Current Convention

| Field | Value |
|-------|-------|
| **Convention** | I_gap,i = g_j · (V_i − V_j), where i is the source cell and j is the coupled partner. This current is outward-positive from cell i (consistent with the ionic current sign convention). The same current enters cell j: I_gap,j = −I_gap,i = g_j · (V_j − V_i). The membrane equation for cell i gains the term −I_gap,i / C_m on the right-hand side (note: outward gap current is treated like a positive ionic current, so it subtracts from dV/dt). |
| **Introduced** | Phase 0 |
| **Rationale** | Standard Ohmic gap junction model (linear conductance). Consistent with HH outward-positive current convention. Brian2 Synapses implementation: the gap junction is symmetric — net current flows from higher to lower V_m. |
| **Dependencies** | Brian2 Synapses code; synchrony analysis; Kuramoto coupling analogy |
| **Test value** | If V_i = −50 mV, V_j = −70 mV, g_j = 1 nS: I_gap,i = 1 × 10⁻⁹ S × 20 × 10⁻³ V = 20 pA (outward from cell i). Cell i loses charge (hyperpolarized toward j); cell j gains charge (depolarized toward i). Net effect: cells couple toward a common potential. |

### Gap Junction Conductance Units

| Field | Value |
|-------|-------|
| **Convention** | g_j in **nS (nanosiemens) per junction**. Literature range for thalamic reticular nucleus (TRN) Cx36 gap junctions: 0.1–2 nS per junction (Landisman et al. 2002). Wider sweep range for parameter exploration: 0.1–10 nS (CALC-01). The coupling coefficient K = g_j / (g_j + g_m) is dimensionless (range 0.01–0.5), where g_m is the total membrane conductance of the cell (at rest). |
| **Introduced** | Phase 0 |
| **Rationale** | SI-derived unit (nS is standard in patch-clamp physiology). Coupling coefficient K provides a normalized measure independent of cell size. TRN Cx36 dominates thalamic gap junctions (Landisman et al. 2002). |
| **Dependencies** | CALC-01 parameter sweep; synchrony threshold identification; VALD-03 uncoupled control (g_j = 0) |
| **Test value** | At g_j = 1 nS and g_m ≈ 10 nS (typical HH cell resting conductance): K = 1/(1+10) ≈ 0.09 (weak coupling; subcritical regime expected). At g_j = 5 nS: K ≈ 0.33 (stronger coupling; supercritical regime likely). |

### Gap Junction Model Validity

| Field | Value |
|-------|-------|
| **Convention** | The linear Ohmic model I_gap = g_j · (V_i − V_j) with fixed g_j is valid when: (1) g_j < 1 nS per junction, OR (2) coupling coefficient K < 0.1, OR (3) transjunctional voltage |V_i − V_j| < 30 mV. When any of these thresholds is exceeded, the Cx36 voltage-dependent gating becomes non-negligible and must be documented as a model limitation. |
| **Introduced** | Phase 0 |
| **Rationale** | Cx36 shows weak voltage-gating at transjunctional voltages > 30 mV (Srinivas et al. 1999; Teubner et al. 2000). During action potentials, transjunctional voltage may briefly exceed this. The linear approximation is documented as an assumption with explicit bounds. |
| **Dependencies** | m2 pitfall mitigation; Phase 2 parameter sweep; any result reported at g_j > 1 nS |
| **Test value** | Check: in Phase 2 sweep at g_j = 5 nS, verify that peak transjunctional voltage during AP does not exceed 30 mV. If it does, document as a limitation. |

---

## 6. EM Field and Volume Conductor

### Quasi-Static Approximation (Mandatory Justification)

| Field | Value |
|-------|-------|
| **Convention** | The EM field is computed in the **quasi-static limit**: Maxwell's equations reduce to Poisson's equation ∇·(σ∇φ) = −∇·J_imp, where J_imp is the impressed (neuronal) current density. The retarded (full-wave) treatment adds nothing: at f = 1 kHz and L = 10 mm, the retardation ratio L/λ_EM = fL/c ≈ (10³ Hz × 10⁻² m) / (3 × 10⁸ m/s) ≈ 3.3 × 10⁻⁸ ≪ 1. The quasi-static and full-wave Maxwell solutions agree to 9 significant figures for all neural signals (Plonsey & Heppner 1967; Nunez & Srinivasan 2006). Every document that uses Maxwell's equations must include this one-line justification. |
| **Introduced** | Phase 0 |
| **Rationale** | C3 pitfall prevention. Failure to state quasi-static validity leads to confusion between "EM wave standing modes" (irrelevant at neural scales) and "LFP spatial eigenmodes" (the correct concept). The retardation correction is ~10⁻⁸ to 10⁻¹¹ for neural frequencies and brain dimensions — entirely negligible. |
| **Dependencies** | All LFP forward model derivations; DERV-01 theory document; every code module that solves the EM problem |
| **Test value** | At f_max = 10 kHz, L = 1 cm: L/λ = 10⁴ × 10⁻² / (3 × 10⁸) ≈ 3.3 × 10⁻⁷. Still ≪ 1. Quasi-static remains valid over entire project frequency range (0.5–10,000 Hz). |

### Poisson Equation and LFP Forward Model

| Field | Value |
|-------|-------|
| **Convention** | Governing equation (quasi-static, homogeneous isotropic medium): ∇·(σ∇φ) = −∇·J_imp, which in homogeneous σ reduces to σ∇²φ = −∇·J_imp. For the transmembrane current I_m at position r' (units: A/m³ for volume current density, or A for a point/line source), the solution is: φ(r) = (1/(4πσ)) ∫ J(r')·∇'(1/|r − r'|) dV'. For a discrete set of cable compartment transmembrane currents I_m,k (in A) at positions r_k: φ(r) = (1/(4πσ)) Σ_k I_m,k / |r − r_k|. |
| **Introduced** | Phase 0 |
| **Rationale** | Standard volume conductor LFP forward model (Nunez & Srinivasan 2006; Hales 2014; Linden et al. 2010). The formula is exact for homogeneous isotropic σ; approximate for heterogeneous tissue (error bound 2–3×, see M3 pitfall). |
| **Dependencies** | JAX-GPU Green's function kernel implementation; VALD-02 validation; LFP amplitude units; sign of source term |
| **Test value** | Point current source I_0 = 1 nA at origin, σ = 0.33 S/m, r = 1 mm: φ = 10⁻⁹ / (4π × 0.33 × 10⁻³) ≈ 0.241 µV. VALD-02 target: < 1% error between kernel sum and this analytical value. |

### Tissue Conductivity

| Field | Value |
|-------|-------|
| **Convention** | σ = 0.33 S/m (homogeneous, isotropic gray matter at 37°C; real part of complex conductivity at neural frequencies). Valid for syncytium-only LFP computation within thalamic tissue. Error bound vs. real heterogeneous tissue: factor 2–3 in field amplitude at boundaries. This approximation is documented as M3 mitigation; no correction needed for Phase 1–3 syncytium-only work. |
| **Introduced** | Phase 0 |
| **Rationale** | Nunez & Srinivasan (2006) standard value for cortical/thalamic gray matter. Homogeneous isotropic approximation valid within a single tissue compartment; breaks down at skull/CSF boundaries (not relevant until EXT-03 forward model). |
| **Dependencies** | All LFP amplitude calculations; Green's function tensor G_ij; VALD-02 test; amplitude comparison to literature |
| **Test value** | LFP amplitude at 100 µm from a single HH cell: using σ = 0.33 S/m and I_m ≈ 1 nA → φ ≈ 2.4 µV at 100 µm. Order-of-magnitude consistent with Linden et al. (2010) single-cell LFP. |

### Green's Function Tensor

| Field | Value |
|-------|-------|
| **Convention** | For N source compartments at positions {r_k} and N_obs observation points at positions {r_i}, define the Green's function tensor **G** (N_obs × N_src matrix): G_ij = 1 / (4πσ|r_i − r_j|). The LFP vector φ (length N_obs) is then: φ = **G** · I_m, where I_m (length N_src) is the vector of transmembrane currents in amperes. This matrix-vector product is the primary per-timestep EM computation. For N_obs = N_src = N: cost is O(N²) per timestep. G is precomputed and constant (static geometry). |
| **Introduced** | Phase 0 |
| **Rationale** | JAX-GPU implementation strategy (METHODS.md). Precomputing G amortizes the geometry cost. The single matrix-vector multiply φ = G·I_m is the bottleneck operation, executed at every simulation timestep. |
| **Dependencies** | JAX-GPU kernel code; VALD-02 validation; FMM transition threshold (N > 3000); VRAM estimate |
| **Test value** | For N = 100, G is 100×100 float32 matrix ≈ 40 kB. For N = 1000: 4 MB. For N = 3000: 36 MB. For N > 3000: switch to ExaFMM-t. VALD-02: |φ_kernel − φ_analytical| / |φ_analytical| < 0.01. |

### LFP Field Amplitude Units

| Field | Value |
|-------|-------|
| **Convention** | Report LFP amplitudes in **µV or µV/mm** for local field; use **V/m or mV/m** for electric field (gradient of φ). Do NOT report normalized amplitudes only — always include absolute SI units. Conversion: 1 µV/mm = 1 mV/m = 10⁻³ V/m. Do NOT use CGS units (Gaussian system). Convert: 1 µV/cm = 0.1 mV/m (note the factor-of-10 trap). |
| **Introduced** | Phase 0 |
| **Rationale** | C1 pitfall prevention. Normalized-only reporting prevents comparison to empirical LFP literature. CGS units create factor-of-10 errors. |
| **Dependencies** | All simulation output; figure axis labels; comparison to Linden et al. (2010); PSD normalization |
| **Test value** | Single pyramidal neuron LFP at 100 µm (literature): ~1–10 µV. TRN syncytium (N=100) LFP at 200 µm: estimated ~10–100 µV (scale with √N from coherent sources). Report and compare. |

---

## 7. "Standing Waves" and Spatial Mode Terminology

### Standing Wave Definition (Critical Terminology Lock)

| Field | Value |
|-------|-------|
| **Convention** | In this project, **"standing wave"** means a **spatial eigenmode of the population-level LFP field** — a pattern of constructive/destructive interference in the network's collective extracellular potential driven by the network dynamics. It does NOT mean an electromagnetic radiation standing wave (which would require L ≈ λ_EM / 2 = c/(2f) ≈ 1,500 km at 100 Hz — impossible at thalamic scales). Preferred precise terminology: **"LFP spatial mode"** or **"network field mode"** when referring to the population dynamics pattern. Reserve "electromagnetic" for quantities derived directly from Maxwell's equations (φ, J_imp, E-field). |
| **Introduced** | Phase 0 |
| **Rationale** | SUMMARY.md "Notation conflict resolved — standing waves." C3 pitfall prevention. The two meanings are physically incompatible. Using "EM standing wave" without this distinction implies resonance lengths of thousands of kilometers, which would correctly be rejected by any referee. The LFP spatial mode IS computed from Maxwell's equations applied to neural currents (Poisson equation), so the EM treatment is real — but the resonance is in the neural dynamics, not in EM wave propagation. |
| **Dependencies** | DERV-01 theory framework; all figures; paper language; LEHAR anchor interpretation |
| **Test value** | Quasi-static check: L/λ_EM at 100 Hz, L = 1 cm: (100 × 10⁻²)/(3 × 10⁸) = 3.3 × 10⁻⁹. Standing wave wavelengths of concern are the SPATIAL PERIODS of the LFP mode structure, which are set by the inter-cell spacing and network architecture (µm–mm scale), not by EM wavelengths. |

### SVD Spatial Mode Extraction Convention

| Field | Value |
|-------|-------|
| **Convention** | To extract LFP spatial modes: stack field snapshots as the matrix M (shape: N_obs × T_snapshots), where column t is the field vector at time t. Compute SVD: M = U · Σ · V^T. Left singular vectors U[:,k] are the spatial mode patterns (ordered by decreasing singular value σ_k). Right singular vectors V[:,k] are the temporal coefficients. Dominant mode = U[:,0] (first left singular vector, corresponding to largest σ_0). Mode energy = σ_k². |
| **Introduced** | Phase 0 |
| **Rationale** | SUMMARY.md and METHODS.md standard. SVD (= POD) gives energy-ranked spatial basis. The dominant mode U[:,0] is the standing wave pattern with the most energy. For stimulus encoding: the dominant mode is the primary descriptor of the LFP spatial structure. |
| **Dependencies** | Phase 2 spatial mode extraction; Phase 3 stimulus discriminability; VALD-04 holographic recovery; numpy.linalg.svd or torch.linalg.svd convention |
| **Test value** | For a single coherent oscillation across all cells (full synchrony): U[:,0] should be a uniform spatial pattern (all same sign and similar magnitude). For two-mode case: U[:,0] and U[:,1] should be approximately orthogonal spatial patterns. |

---

## 8. Holographic Encoding Operational Definition

### Field Quantity F(x, t) for Holographic Encoding

| Field | Value |
|-------|-------|
| **Convention** | The "hologram" is operationally defined as the **dominant LFP spatial mode** at steady state under a given sensory stimulus: F(stimulus) = U[:,0], the first left singular vector of the LFP snapshot matrix M computed during the steady-state response window. F is a real-valued vector of length N_obs (one component per observation point). |
| **Introduced** | Phase 0 |
| **Rationale** | DERV-02 requirement; SUMMARY.md operational definition. Lehar's harmonic resonance theory is analogical — this operational definition grounds it in a measurable LFP quantity. The dominant SVD mode captures the maximum-energy spatial structure of the field response. |
| **Dependencies** | VALD-04 holographic recovery test; Phase 3 stimulus discriminability; reconstruction operator (below) |
| **Test value** | F has unit norm (singular vector normalization). Two distinct stimuli should produce F vectors with low mutual correlation: Pearson(F_stim1, F_stim2) < 0.5 for discriminable stimuli (VALD-03 criterion). |

### Holographic Recovery Fidelity Metric

| Field | Value |
|-------|-------|
| **Convention** | After removing a fraction p of syncytium cells at random, recompute the LFP spatial mode F_reconstructed from the remaining (1−p)N cells. Fidelity: ρ = Pearson(F_original, F_reconstructed) = (F_original · F_reconstructed) / (|F_original| × |F_reconstructed|). Pass threshold: ρ ≥ 0.70 after p = 20% cell loss (DERV-02; VALD-04). |
| **Introduced** | Phase 0 |
| **Rationale** | Pearson correlation is scale-invariant and captures spatial pattern similarity without being sensitive to amplitude scaling after cell loss. The 0.70 threshold corresponds to ~49% shared variance — a meaningful level of structural preservation. |
| **Dependencies** | VALD-04 implementation; baseline comparison (random distributed code; same N and same p); Phase 3 holographic recovery figure |
| **Test value** | Baseline: run same cell-removal test on a matched-N random distributed code (non-holographic comparator). Random code should show ρ < 0.70 at p = 20% loss. LFP spatial mode should show ρ ≥ 0.70. This comparison distinguishes holographic redundancy from ordinary distributed coding. |

---

## 9. Synchrony and Collective Dynamics

### Kuramoto Order Parameter

| Field | Value |
|-------|-------|
| **Convention** | R(t) = (1/N) |Σ_{k=1}^{N} exp(i θ_k(t))|, where θ_k(t) is the instantaneous phase of cell k's membrane potential oscillation (extracted via Hilbert transform or peak-detection). R ∈ [0, 1]. Coupling regimes: R < 0.2 → subcritical (asynchronous); 0.2 ≤ R ≤ 0.8 → near-critical (partial synchrony); R > 0.8 → supercritical (near-synchronous). The critical coupling threshold g_j^c is defined as the g_j value at which ∂R/∂g_j is maximal (steepest rise). |
| **Introduced** | Phase 0 |
| **Rationale** | SUMMARY.md; standard Kuramoto model analogy for the gap-junction-coupled HH network. R quantifies coherence on a scale independent of N. The HH network is not exactly a Kuramoto model, but R provides a useful phenomenological synchrony metric. |
| **Dependencies** | CALC-01 g_j sweep; synchrony threshold identification; Phase 2 results |
| **Test value** | Uncoupled cells (g_j = 0, all cells driven independently with noise): R ~ 1/√N ≈ 0.1 for N = 100 (incoherent superposition). Fully synchronized (all cells phase-locked): R → 1. Transition region of primary interest: R sweeps 0.1 → 0.9 as g_j increases from 0.1 to 10 nS. |

---

## 10. Fourier and Spectral Conventions

### Temporal Fourier Convention

| Field | Value |
|-------|-------|
| **Convention** | Forward transform: f_tilde(ω) = ∫_{−∞}^{+∞} f(t) e^{−iωt} dt. Inverse transform: f(t) = ∫_{−∞}^{+∞} [dω/(2π)] f_tilde(ω) e^{+iωt}. Delta function normalization: δ(t) = ∫ [dω/(2π)] e^{+iωt}. This is the physics convention (asymmetric, e^{−iωt} forward). |
| **Introduced** | Phase 0 |
| **Rationale** | Standard physics convention; consistent with scipy.signal and MNE-Python implementations. The minus sign in the forward transform means positive-frequency components have positive ω. |
| **Dependencies** | PSD computation (scipy.signal.welch outputs positive frequencies); LFP spectral analysis; cross-coherence; all frequency-domain expressions |
| **Test value** | FT[δ(t)] = 1 (constant spectrum). FT[e^{−iω_0 t}] = 2π δ(ω − ω_0). FT[cos(ω_0 t)] = π[δ(ω − ω_0) + δ(ω + ω_0)]. scipy.signal.welch returns one-sided PSD in units of V²/Hz; multiply by 2 for one-sided from two-sided. |

### EEG/LFP Frequency Band Definitions (IFCN Standard)

| Field | Value |
|-------|-------|
| **Convention** | IFCN standard oscillatory bands (apply to LFP PSD for identification of oscillatory activity): delta: 0.5–4 Hz; theta: 4–8 Hz; alpha: 8–13 Hz; beta: 13–30 Hz; gamma: 30–100 Hz. Thalamic spindle band: 7–14 Hz (overlaps theta–alpha; distinct oscillatory mechanism via I_T/I_h). Gamma high: 80–200 Hz (high-gamma; usually above thalamic spindle range). Intra-burst HH firing: 100–400 Hz (individual spike timing within burst; NOT the spindle envelope frequency). |
| **Introduced** | Phase 0 |
| **Rationale** | IFCN (International Federation of Clinical Neurophysiology) standard (Noachtar et al. 1999). Consistent labeling prevents ambiguity when comparing LFP PSD profiles across conditions and with empirical data. |
| **Dependencies** | CALC-03 PSD analysis; VALD-01 spindle frequency check; Phase 3 stimulus-specific LFP profiles; all spectral figures |
| **Test value** | Spindle frequency validation (VALD-01): Welch PSD of single HH cell (or uncoupled N=100) should show a peak in the 7–14 Hz range when I_T and I_h are active and cell is driven appropriately. Peak within ±2 Hz of target band centers (CALC-03 criterion). |

### PSD Normalization

| Field | Value |
|-------|-------|
| **Convention** | Power spectral density computed via scipy.signal.welch with: window = 'hann'; nperseg = at least 4× the period of the lowest frequency of interest (e.g., for 7 Hz spindles: nperseg ≥ 4 × (1/7 Hz) × (1/dt) = 4 × 143 ms / 0.025 ms ≈ 22,857 samples); scaling = 'density' (output in V²/Hz for LFP in volts; µV²/Hz for LFP in µV). Report as one-sided PSD (positive frequencies only). |
| **Introduced** | Phase 0 |
| **Rationale** | Welch periodogram is the standard for neural LFP PSD estimation (MNE-Python; scipy). Hann window reduces spectral leakage. Reporting in density (V²/Hz) allows comparison independent of recording duration. |
| **Dependencies** | CALC-03; all PSD figures; stimulus discriminability analysis |
| **Test value** | White noise LFP with variance σ² V² and bandwidth B Hz: PSD ≈ σ²/B V²/Hz. Verify: integrate Welch PSD over [0, f_max] recovers total signal variance. |

---

## 11. Unit System

### Primary Unit System

| Field | Value |
|-------|-------|
| **Convention** | **SI units throughout**. No CGS (Gaussian) units. Neural dynamics use the following SI-derived units for readability: membrane potential in mV; time in ms; conductance in nS or mS/cm²; capacitance in µF/cm²; current in pA or nA or µA/cm²; length in µm; frequency in Hz. EM field quantities use strict SI: φ in V or µV; E = −∇φ in V/m or mV/m; current density J in A/m²; conductivity σ in S/m. Conversion: 1 µV/cm = 0.1 mV/m = 10⁻⁴ V/m. |
| **Introduced** | Phase 0 |
| **Rationale** | SI prevents the factor-of-4π errors that arise with Gaussian CGS for electrostatics. All modern Python libraries (Brian2, scipy, numpy) use SI. Explicit unit conversion factors must be applied when importing parameters from older literature (some pre-1990 HH papers use CGS). |
| **Dependencies** | All simulation code; Brian2 equation definitions; LFP amplitude calculation; parameter table |
| **Test value** | Check: σ = 0.33 S/m = 0.33 A/(V·m). G_ij = 1/(4πσ|r_i−r_j|) in units of 1/(S/m × m) = Ω = V/A. φ = G·I_m in units of Ω × A = V. ✓ |

### Neural Simulation Units in Brian2

| Field | Value |
|-------|-------|
| **Convention** | Brian2 uses SI internally. Model equations must be written with explicit units. Standard Brian2 unit definitions for this project: V_m in mV (Brian2 `mV`); g in mS/cm² (Brian2 `msiemens/cm**2`); C_m = 1 µF/cm² (Brian2 `ufarad/cm**2`); time constants in ms (Brian2 `ms`); I in µA/cm² (Brian2 `uA/cm**2`); length in µm (Brian2 `um`). Axial resistance R_a in Ω·cm (Brian2 `ohm*cm`). |
| **Introduced** | Phase 0 |
| **Rationale** | Brian2 enforces dimensional consistency at compile time. Explicit unit attachment prevents silent unit errors. |
| **Dependencies** | All Brian2 model code; SIMU-01 implementation |
| **Test value** | Brian2 dimensional check: C_m * dV/dt = I must be dimensionally consistent: (µF/cm²) × (mV/ms) = (µF/cm²) × (V/s) = µA/cm². ✓ |

---

## 12. Simulation Numerical Parameters

### Simulation Timestep

| Field | Value |
|-------|-------|
| **Convention** | dt ≤ 0.025 ms (25 µs) for all HH cable simulations. Never exceed dt = 0.05 ms. This constraint is set by the fast Na⁺ activation kinetics of HH (time constant τ_m ≈ 0.1 ms at spike threshold). Crank-Nicolson cable discretization (unconditionally stable for passive cable) plus backward Euler (or exponential Euler) for active gate variables. |
| **Introduced** | Phase 0 |
| **Rationale** | M1 pitfall prevention. Standard timestep for HH simulations in Brian2 (Stimberg et al. 2019). Violating this causes timestep-induced spike timing errors and can produce artificial oscillations. |
| **Dependencies** | All Brian2 simulations; Phase 1 stability validation; runtime estimates (N=100: dt=0.025 ms, T=1 s → 40,000 steps) |
| **Test value** | Validation: run single HH cell at dt = 0.025 ms and dt = 0.005 ms; spike times should agree to < 0.1 ms (M1 criterion from PITFALLS.md). |

### HH Gating Variable Initialization

| Field | Value |
|-------|-------|
| **Convention** | Initialize all HH gate variables to their steady-state values at V_m = −65 mV (resting potential): m_∞(−65) ≈ 0.053; h_∞(−65) ≈ 0.596; n_∞(−65) ≈ 0.318; m_T_∞(−65) ≈ variable (check McCormick & Huguenard); h_T_∞(−65) ≈ variable; m_h_∞(−65) ≈ 0.1–0.3. Allow 100 ms settling time before recording begins. Do NOT initialize to 0 (produces transient artifacts). |
| **Introduced** | Phase 0 |
| **Rationale** | m1 pitfall prevention. Non-equilibrium initialization produces spurious transient spikes in the first 10–100 ms of simulation. |
| **Dependencies** | All simulations; single-cell validation; Phase 2 syncytium |
| **Test value** | After 100 ms settling at V_m = −65 mV with zero external current: dV/dt ≈ 0, all gate variables stable at their ∞ values. Net ionic current ≈ 0. |

### Sampling Rate and Anti-Aliasing

| Field | Value |
|-------|-------|
| **Convention** | The simulation generates LFP at the HH timestep (dt = 0.025 ms → 40 kHz). For PSD analysis of LFP: downsample to ≥ 2 kHz (anti-alias filter at 1 kHz before downsampling) or use LFP snapshots recorded at the full 40 kHz (preferable for up to 1 kHz analysis). For gamma-band (30–100 Hz) analysis: 1 kHz sampling is adequate. For intra-burst HH analysis (100–400 Hz): use full 40 kHz. |
| **Introduced** | Phase 0 |
| **Rationale** | Nyquist theorem; anti-aliasing requirement from PITFALLS.md numerical traps section. |
| **Dependencies** | LFP recording code; PSD analysis; SVD snapshot matrix construction |
| **Test value** | At 2 kHz (downsampled): can resolve up to 1 kHz. Spindle band (7–14 Hz) and gamma (30–100 Hz) are well within range. At 40 kHz: full HH spike waveform resolved. |

---

## 13. Sensory Input Conventions

### 3D Sensory Field Convention

| Field | Value |
|-------|-------|
| **Convention** | Sensory inputs are genuine **3D spatial fields**: (1) Visual: 3D light field (radiance field / plenoptic function) represented as a 3D Gaussian Splatting (3DGS) model (gsplat library); (2) Auditory: spatial audio in Ambisonics format (first-order Ambisonics minimum: 4 channels W, X, Y, Z). The sensory field drives the thalamic syncytium via afferent currents I_aff_i(t) at cell i, proportional to the field value at the spatial location corresponding to that cell's receptive field position. |
| **Introduced** | Phase 0 |
| **Rationale** | REQUIREMENTS.md SIMU-03; PROJECT.md scope boundary ("2D sensory simplification" is forbidden). Virtual point sources are acceptable as test stimuli but must exist within a 3D field representation. |
| **Dependencies** | SIMU-03; Phase 3 stimulus encoding; stimulus discriminability; sensory coordinate system |
| **Test value** | Two distinct stimuli (e.g., point source at position A vs. position B in 3D space) must produce measurably different afferent current patterns I_aff_i(t) across the syncytium. If all cells receive identical afferent drive regardless of stimulus location, the 3D encoding is broken. |

---

## 14. Reference Convention Maps

### Hodgkin & Huxley (1952)

| Field | Value |
|-------|-------|
| **Reference convention** | Original HH (1952) uses: outward-positive currents; V_m = V_in − V_ex; h = fraction NOT inactivated; E_Na ≈ +55 mV, E_K ≈ −12 mV (squid axon — different from thalamic values). Rate constants α, β given for 6.3°C (squid giant axon temperature). |
| **Project conversion** | Use HH current and gate conventions directly. Adjust reversal potentials and maximal conductances to McCormick & Huguenard (1992) thalamic values. Apply Q10 temperature correction to 37°C. |

### McCormick & Huguenard (1992)

| Field | Value |
|-------|-------|
| **Reference convention** | J. Neurophysiol. 68(4):1384–1400. Outward-positive currents. Parameters for thalamic relay cells at 36°C. Separate equations for I_T, I_h. Conductances in mS/cm². |
| **Project conversion** | Direct parameter import. Apply Q10 correction from 36°C to 37°C (factor ≈ 1.1× for most rate constants). Verify all Table 1 parameters before Phase 1 implementation (status: VERIFY — primary source not yet read). |

### Nunez & Srinivasan (2006)

| Field | Value |
|-------|-------|
| **Reference convention** | "Electric Fields of the Brain" 2nd ed. OUP. Uses σ in S/m; volume conductor formula φ = (1/4πσ) ∫ J·∇'(1/|r-r'|) dV'; impressed current convention J_imp. Quasi-static throughout. |
| **Project conversion** | Direct formula import. No conversion needed. All notation is consistent with project conventions. |

### Landisman et al. (2002)

| Field | Value |
|-------|-------|
| **Reference convention** | J. Neurosci. 22(3):1002–1009. TRN Cx36 gap junctions. g_j = 0.1–2 nS per junction (junctional conductance, not coupling coefficient K). Coupling coefficient K = 0.01–0.1 defined as DC steady-state ratio. |
| **Project conversion** | g_j units match (nS). Note: Landisman 2002 reports TRN cells specifically. If relay cells are used instead of TRN, g_j values may differ significantly. Status: VERIFY — TRN vs. relay cell design decision is open (Phase 1 required reading). |

### Lehar (2003) — Theory Anchor

| Field | Value |
|-------|-------|
| **Reference convention** | Behav. Brain Sci. 26(4):375–408. Purely qualitative/analogical; no quantitative predictions for LFP amplitude, spatial frequency, or wavelength at thalamic scales. Uses "standing wave" and "holographic" in an analogical sense. |
| **Project conversion** | Lehar's "standing waves" = project's "LFP spatial modes" (operational definition established above). Lehar's "holographic encoding" = Pearson fidelity ρ ≥ 0.70 after 20% cell loss (operational definition established above). Do NOT import numerical claims from Lehar — derive all quantities from first principles (HH + Poisson). |

---

## 15. Dimensional Consistency Verification

### Dimension Map (SI)

| Quantity | SI Unit | Neural Shorthand |
|----------|---------|-----------------|
| Membrane potential V_m | V | mV |
| Ionic current density | A/m² or A/cm² | µA/cm² |
| Conductance density | S/m² or S/cm² | mS/cm² |
| Capacitance density | F/m² or F/cm² | µF/cm² |
| Time | s | ms |
| Frequency | Hz = s⁻¹ | Hz |
| Length | m | µm |
| Conductivity σ | S/m = A/(V·m) | S/m |
| Extracellular potential φ | V | µV |
| Current (per cell) | A | nA or pA |
| Green's function G_ij | Ω = V/A | V/A |
| Resistivity R_i | Ω·m | Ω·cm |

### Cross-Convention Dimensional Check

| Check | Expression | Dimensions | Result |
|-------|-----------|-----------|--------|
| HH cable equation | C_m ∂V/∂t = I | (F/m²)(V/s) = A/m² | ✓ |
| Gap junction current | I_gap = g_j(V_i - V_j) | S × V = A | ✓ |
| LFP formula | φ = I/(4πσr) | A/(S/m × m) = A/(A/V) = V | ✓ |
| Green's function product | φ = G·I_m | (V/A) × A = V | ✓ |
| Kuramoto order parameter | R = (1/N)|Σ exp(iθ)| | dimensionless | ✓ |
| Pearson fidelity | ρ = (F·F')/(|F||F'|) | dimensionless | ✓ |

---

## 16. Convention Change Log

> No convention changes recorded yet. All conventions established at project initialization (Phase 0, 2026-03-16).

| Change ID | Convention | Old Value | New Value | Changed In | Reason | Conversion |
|-----------|-----------|-----------|-----------|------------|--------|------------|
| — | — | — | — | — | — | — |

---

## 17. Cross-Convention Compatibility Notes

| Convention A | Convention B | Interaction | Factor/Sign | Example |
|-------------|-------------|-------------|-------------|---------|
| Outward-positive I_ion | Membrane equation C_m dV/dt = ... | Ionic currents subtract from dV/dt (outward = depolarizing → increases V via positive driving force on inward current) | −I_Na − I_K − I_L in RHS of dV/dt | During AP: I_Na < 0 → −I_Na > 0 → depolarizes. ✓ |
| V_m = V_in − V_ex | Gap junction I_gap = g_j(V_i − V_j) | Current flows from high to low V_m; outward-positive from cell i if V_i > V_j | Correct: current leaves higher-V cell, enters lower-V cell | If sign flipped, gap junctions would be destabilizing instead of coupling |
| φ = I/(4πσr) | Outward-positive I_m | Outward current = positive source in extracellular medium → positive φ near outward current site | Sign: φ ∝ +I for outward-positive convention | Soma during AP repolarization (I_K outward): positive φ near soma |
| Quasi-static Poisson | Temporal Fourier e^{−iωt} | Quasi-static means no retardation → no e^{iωr/c} phase factor in spatial part | No cross-convention issue; each applies in its domain | Quasi-static = instantaneous propagation; Fourier applied to time series analysis only |
| Kuramoto R | HH phase θ_k | Phase θ_k extracted from V_m oscillation via Hilbert transform; convention: θ_k = 0 at peak of V_m (AP peak) | Phase of complex analytic signal of V_m | Ensure consistent phase extraction method across all cells |
| SVD convention M = UΣV^T | Holographic fidelity ρ = Pearson(F, F') | F = U[:,0] (unit norm, real); Pearson ρ = F·F' (inner product since unit norm) | ρ ∈ [−1, 1]; ρ = 1 → identical; ρ = 0 → orthogonal | Sign ambiguity in U[:,0] (SVD eigenvectors defined up to sign): fix sign convention by requiring majority of components positive |

---

## 18. Machine-Readable Convention Tests

```yaml
# Parseable by consistency checker for automated validation

convention_tests:

  observable_lock:
    observable: "LFP"
    units: "uV or V/m"
    NOT: "scalp_EEG"
    test: "Report field amplitude as φ in uV; compare to LFPy/Linden2010 literature range"

  membrane_potential_sign:
    convention: "V_m = V_intracellular - V_extracellular"
    resting_value_mV: -65
    test: "At rest: V_m = -65 mV. Spike peak: V_m ≈ +40 mV. Afterhyperpolarization: V_m ≈ -75 mV."

  outward_positive_current:
    convention: "outward_positive"
    I_Na_direction: "inward_during_AP_upstroke"
    I_Na_sign_during_AP: "negative"
    test: "I_Na = g_Na * m^3 * h * (V_m - E_Na) at V_m = -65 mV: (-65 - 55) = -120 mV driving force -> I_Na < 0 (inward). ✓"

  HH_h_variable:
    h_meaning: "fraction_NOT_inactivated"
    h_at_rest: 0.596
    h_fully_inactivated: 0.0
    test: "h_inf(-65 mV) ≈ 0.596; h_inf(-40 mV) ≈ 0.05. Verify Brian2: h dips during AP and recovers on hyperpolarization."

  gap_junction_current:
    formula: "I_gap_i = g_j * (V_i - V_j)"
    convention: "outward_positive_from_cell_i"
    sign_test: "V_i > V_j -> I_gap_i > 0 (outward from i, inward to j). ✓"
    test: "g_j=1nS, V_i=-50mV, V_j=-70mV: I_gap = 1e-9 * 20e-3 = 20e-12 A = 20 pA outward from i."

  quasi_static_validity:
    approximation: "Poisson equation (quasi-static Maxwell)"
    retardation_ratio: "L/lambda_EM << 1"
    test: "f=1kHz, L=1cm: L/lambda = (1e3 * 1e-2)/(3e8) = 3.3e-8 << 1. ✓"
    required_in: "every_document_using_Maxwell"

  LFP_forward_model:
    formula: "phi(r) = (1/(4*pi*sigma)) * sum_k I_m_k / |r - r_k|"
    sigma_SI: 0.33
    sigma_units: "S/m"
    test: "I_0=1nA, r=1mm: phi = 1e-9/(4*pi*0.33*1e-3) = 0.241 uV. VALD-02: error < 1%."

  standing_wave_terminology:
    meaning: "LFP_spatial_eigenmode_of_network_dynamics"
    NOT: "EM_radiation_standing_wave"
    test: "L/lambda_EM at 100 Hz, 1 cm = 3.3e-9. EM radiation standing wave impossible. LFP mode spatial period set by network architecture."

  SVD_convention:
    matrix_layout: "M[observations, time_snapshots]"
    decomposition: "M = U * Sigma * V^T"
    dominant_mode: "U[:, 0]"
    test: "Full synchrony -> U[:,0] approximately uniform. numpy.linalg.svd(M, full_matrices=False)[0][:,0]."

  holographic_fidelity:
    metric: "Pearson_correlation"
    formula: "rho = dot(F_orig, F_recon) / (norm(F_orig) * norm(F_recon))"
    pass_threshold: 0.70
    cell_loss_fraction: 0.20
    baseline: "matched_N_random_distributed_code"
    test: "rho >= 0.70 after 20% cell removal. Random code baseline should give rho < 0.70."

  Kuramoto_order_parameter:
    formula: "R = (1/N) * abs(sum_k exp(i*theta_k))"
    subcritical_threshold: 0.2
    supercritical_threshold: 0.8
    test: "Uncoupled N=100 with independent noise: R ~ 1/sqrt(N) = 0.1. Full synchrony: R -> 1."

  fourier_convention:
    forward: "f_tilde(omega) = integral f(t) exp(-i*omega*t) dt"
    inverse: "f(t) = integral [d_omega/(2*pi)] f_tilde(omega) exp(+i*omega*t)"
    test: "FT[delta(t)] = 1. FT[exp(-i*omega_0*t)] = 2*pi*delta(omega - omega_0)."

  simulation_timestep:
    dt_max_ms: 0.025
    dt_never_exceed_ms: 0.05
    test: "Run HH cell at dt=0.025ms and dt=0.005ms; spike times agree to < 0.1 ms."

  unit_system:
    EM_units: "SI (V, A, S/m, V/m)"
    neural_units: "mV, ms, nS, uF/cm2, uA/cm2, um"
    forbidden: "CGS_Gaussian"
    test: "phi = I/(4*pi*sigma*r): units A/(S/m * m) = A/(A/(V*m) * m) = V. ✓"

  EEG_bands_IFCN:
    delta_Hz: [0.5, 4]
    theta_Hz: [4, 8]
    alpha_Hz: [8, 13]
    beta_Hz: [13, 30]
    gamma_Hz: [30, 100]
    spindle_Hz: [7, 14]
    test: "Welch PSD of I_T/I_h HH cell should show peak in [7,14] Hz range. VALD-01 criterion."
```

---

## 19. Pitfall-Convention Mapping

> Links each convention to the specific pitfall it prevents. Use this as a checklist before starting each phase.

| Pitfall ID | Description | Convention(s) That Prevent It |
|-----------|-------------|-------------------------------|
| C1 | EEG amplitude scale gap | Observable lock (LFP, §1); field amplitude units (§6) |
| C2 | Holographic operationalization | F = U[:,0] (§8); ρ = Pearson ≥ 0.70 (§8); baseline comparison requirement |
| C3 | Standing wave terminology confusion | "Standing wave" = LFP spatial eigenmode (§7); quasi-static justification (§6) |
| C4 | Gap junction parameter sensitivity | g_j in nS; K dimensionless; sweep 0.1–10 nS required (§5) |
| M1 | HH ODE stiffness | dt ≤ 0.025 ms; Crank-Nicolson + backward Euler (§12) |
| M2 | EEG bands without mechanism control | g_j = 0 uncoupled control required; spatial mode test (§9, §7) |
| M3 | Tissue homogeneity error | σ = 0.33 S/m; error bound 2–3×; stated as limitation (§6) |
| M4 | Synchrony vs. field coherence conflation | Kuramoto R defined (§9); causal direction: neural → field |
| M5 | Wrong observable | LFP not scalp EEG; absolute amplitude in V/m required (§1, §6) |
| m1 | HH initialization | Steady-state initialization at −65 mV; 100 ms settling (§12) |
| m2 | Voltage-gated g_j during spikes | Fixed g_j valid for g_j < 1 nS; document assumption (§5) |
| m3 | Ephaptic coupling scope | Quantify in Phase 1; defer unless > 1% of spike threshold |

---

_Conventions ledger created: 2026-03-16_
_Last updated: 2026-03-16 (Phase 0 — project initialization)_
_Next update: Phase 1 completion — add any new conventions discovered during HH implementation and LFP kernel validation_
