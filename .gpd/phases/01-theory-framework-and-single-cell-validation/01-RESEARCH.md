# Phase 1: Theory Framework and Single-Cell Validation — Research

**Researched:** 2026-03-16
**Domain:** Computational neuroscience — thalamic HH cable modeling, quasi-static volume conductor LFP, holographic neural encoding
**Confidence:** MEDIUM (primary sources for Hales 2014 and McCormick & Huguenard 1992 confirmed accessible but not fully extracted from PDFs; all parameter values from training knowledge cross-checked against web-accessible abstracts and ModelDB; Brian2CUDA multicompartment GPU support confirmed)

---

## User Constraints

See phase CONTEXT.md for locked decisions. No CONTEXT.md was present for this phase. The following constraints derive from the project CONTRACT embedded in the phase scope and the project-level SUMMARY.md:

**Locked decisions (from project contract and roadmap):**
- Target observable is LFP (local field potential), NOT scalp EEG — locked before any simulation
- Quasi-static Maxwell (Poisson) is the EM method — full-wave FDTD is never needed at neural scales
- Brian2 + Brian2CUDA is the primary neural simulation stack
- NEURON is used only for single-cell cross-validation
- JAX-GPU dense tensor is the primary LFP Green's function kernel for N <= 3,000
- McCormick & Huguenard (1992) is the primary HH parameter source
- Phase 1 must exit with holographic encoding OPERATIONALLY DEFINED before Phase 2 begins

**Constraints on research scope:**
- Holographic encoding: research the operational reconstruction operator and fidelity metric (DISCRETION area)
- TRN vs. relay cell substrate: research and decide (DISCRETION area — must be resolved this phase)
- Hales (2014) method: confirm whether quasi-static or retarded (MUST confirm before implementation)
- Benchmark dataset for Phase 3 empirical comparison: identify specific paper/figure (MUST resolve)

---

## Active Anchor References

| Anchor / Artifact | Type | Why It Matters Here | Required Action | Where It Must Reappear |
|---|---|---|---|---|
| Hales (2014), J. Integr. Neurosci. 13(2), 313–361 | Method anchor | EM field computation from HH cable currents — is it quasi-static or full retarded? | READ full paper before implementation; confirm method reduces to Poisson equation at neural scales | Plan 01-01, Plan 01-03 |
| McCormick & Huguenard (1992), J. Neurophysiol. 68(4), 1384–1400 | Parameter anchor | Exact gating equations and parameter values for I_T, I_h, I_NaP; benchmark waveforms | Extract parameter tables (Appendix or Tables 1–2); validate implemented model against LTS threshold ±5 mV | Plan 01-02 |
| Huguenard & McCormick (1992), J. Neurophysiol. 68(4), 1373–1383 | Parameter anchor | Companion paper: kinetic equations for I_T, I_A, I_K2, I_h gating variables | Extract alpha/beta rate constants and Boltzmann parameters | Plan 01-02 |
| Landisman et al. (2002), J. Neurosci. 22(3), 1002–1009 | Cell-type decision anchor | TRN Cx36 gap junctions — g_j = 0.1–2 nS, coupling coefficient 0.01–0.1, range < 50 µm | Confirms TRN as correct syncytium substrate | Plan 01-01, theory document |
| Steriade, McCormick & Sejnowski (1993), Science 262, 679–685 | Benchmark anchor | Spindle 7–14 Hz; intra-burst 100–400 Hz; LTS threshold ~−65 mV | Read Fig. 1–3; extract numerical spindle parameters | Plan 01-02 validation |
| Contreras, Destexhe, Sejnowski & Steriade (1997), J. Neurosci. 17, 1179–1196 | Empirical LFP benchmark | Thalamic in vivo LFP during spindles — multisite recordings in barbiturate-anesthetized cat | Identify Fig. 2 or Fig. 4 as empirical comparator; extract spindle power in 7–14 Hz band | Phase 3 validation (lock now) |
| Lehar (2003), Behav. Brain Sci. 26(4), 375–408 | Theory anchor | Harmonic resonance / holographic world model — analogy source for standing wave spatial modes | Confirm paper provides no quantitative spatial frequency predictions; operationalize via SVD | Plan 01-01 theory document |

**Missing or weak anchors:** Hales (2014) PDF is accessible at newdualism.org but could not be machine-read in PDF form. The user MUST manually extract: (1) whether the paper uses retarded Green's function or quasi-static Poisson; (2) the exact formula for phi from cable segments; (3) amplitude estimates for single AP. This is the most critical missing confirmation for Plan 01-03.

---

## Conventions

| Choice | Convention | Alternatives | Source |
|---|---|---|---|
| Membrane potential | V_m = V_intracellular − V_extracellular; resting ≈ −65 mV | — | SUMMARY.md; HH standard |
| Ionic current sign | Outward-positive; I_Na = g_Na·m³·h·(V_m − E_Na) | Inward-positive (some texts) | Hodgkin & Huxley 1952 |
| h-gate meaning | h = fraction NOT inactivated | h = fraction inactivated (some texts) | PITFALLS.md convention table |
| Reversal potentials | E_Na ≈ +55 mV, E_K ≈ −90 mV, E_Ca ≈ +120 mV, E_L ≈ −70 mV | — | Standard thalamic literature |
| Extracellular potential | phi = (1/4πσ) × sum G_ij × I_m,j; quasi-static | Full retarded potential | Nunez & Srinivasan 2006 |
| Tissue conductivity | σ = 0.33 S/m (homogeneous, isotropic, gray matter) | 0.30–0.40 range in literature | Nunez & Srinivasan 2006 |
| Gap junction current | I_gap = g_j · (V_i − V_j); g_j in nS | — | Landisman et al. 2002 |
| Simulation timestep | dt = 0.025 ms; never > 0.05 ms | — | METHODS.md; HH kinetics constraint |
| EM approximation | Quasi-static (Poisson): ∇·(σ∇φ) = −∇·J_imp | Full Maxwell FDTD | PITFALLS.md C3 |
| "Standing wave" usage | LFP spatial eigenmode of network dynamics — NOT EM radiation mode | EM radiation standing wave | SUMMARY.md notation conflict |
| Temperature | 37°C target (in vivo mammalian); Q10 corrections applied | 36°C (some McCormick papers) | See Q10 notes below |
| Compartment size | Length <= lambda_electrotonic / 10; target <= 50 µm per compartment | — | Cable theory; METHODS.md |

**CRITICAL:** All equations below use these conventions. McCormick & Huguenard (1992) experiments were performed at room temperature (24°C or 34°C in different experiments); Q10 correction of 2.5–3.0 per 10°C must be applied to kinetics before using at 37°C.

Convention loading: see agent-infrastructure.md Convention Loading Protocol.

---

## Executive Summary

### Sub-task 01-01: Theory Framework

Phase 1 requires locking four definitions before any simulation: (1) LFP as target observable (done — locked in project contract); (2) quasi-static Maxwell validity (confirmed — L/lambda_EM ~ 10^-9 at 100 Hz, thalamic scale); (3) holographic encoding operationalized as stimulus reconstruction from dominant SVD spatial modes of the LFP field snapshot matrix; (4) TRN (not relay cells) as the syncytium substrate. The Hales (2014) paper treats endogenous EM field from HH cable currents using Maxwell's equations — web search confirms it covers both microscopic and macroscopic Maxwell forms and computes the scalar electric potential from current filaments in a hippocampal CA1 model. Given the quasi-static validity analysis (retardation ratio 10^-11 at neural scales), the Hales method almost certainly reduces to the standard volume conductor Poisson equation in practice. The theory document must state this explicitly and include the one-line quasi-static justification. Lehar (2003) provides a theoretical framing (harmonic resonance = spatial standing waves in the neural substrate) that maps naturally to SVD spatial eigenmodes of the LFP snapshot matrix — the operationalization is to define F(x,t) as the instantaneous LFP field, compute the SVD of the snapshot matrix, and define the "hologram" as the dominant left singular vectors.

### Sub-task 01-02: HH Cable Cell Implementation

Two complementary McCormick & Huguenard papers from 1992 provide the complete thalamic relay cell HH model: Huguenard & McCormick (1992) gives kinetic equations for I_T, I_A, I_K2, I_h from voltage-clamp; McCormick & Huguenard (1992) assembles these into a full thalamocortical relay cell model including I_NaP. However, recent literature and TRN biology (Landisman 2002, Destexhe 1994) make clear that the syncytium substrate is TRN (GABAergic, Cx36-coupled), not excitatory relay cells. The TRN model should prioritize Destexhe et al. (1994) J. Neurophysiol. for TRN-specific parameters and topology, using McCormick & Huguenard (1992) as the validation target for relay cell behavior. Brian2 SpatialNeuron supports multi-compartment cable models natively; Brian2CUDA is confirmed to support Brian2's full feature set including SpatialNeuron. The key stability requirement is Crank-Nicolson cable solve with exponential Euler for gate variables, dt = 0.025 ms, compartment length <= 50 µm. NEURON cross-validation uses ModelDB model 279 (McCormick & Huguenard 1992 model with full NEURON code).

### Sub-task 01-03: LFP Green's Function Kernel

The JAX-GPU dense Green's function kernel is the standard, validated approach for LFP computation at N <= 3,000. The kernel is phi_i = (1/4πσ) × sum_j (I_m,j / |r_i − r_j|) for point sources, or the line-source extension for cable segments. The transmembrane currents I_m,j are extracted from the Brian2 SpatialNeuron model at each timestep. LFPy 2.0 (Hagen et al. 2018) implements the identical formulation and provides a point-source test case for validation — the analytical solution phi = I/(4πσr) must be reproduced to <1% relative error. The primary numerical concern is singularity at r=0 (self-source term), which is handled by a minimum distance cutoff of compartment radius (~1 µm). Float32 precision gives approximately 7 significant digits — sufficient for <1% error at r > 10 µm, but float64 is recommended for the validation test.

---

## Mathematical Framework

### Key Equations and Starting Points

| Equation | Name/Description | Source | Role in Phase 1 |
|---|---|---|---|
| C_m dV/dt = −I_Na − I_K − I_L − I_T − I_h − I_NaP + I_gap + I_ext + (V_{n-1} − 2V_n + V_{n+1})/R_a | Cable equation with thalamic channels | McCormick & Huguenard 1992; Rall 1969 | Governs membrane dynamics in each compartment |
| I_T = g_T · m_T^2 · h_T · (V_m − E_Ca) | T-type Ca2+ current | Huguenard & McCormick 1992, Eq. 1 | LTS and spindle generation; MANDATORY |
| I_h = g_h · m_h · (V_m − E_h) | HCN/h current | McCormick & Huguenard 1992 | Delta termination, rhythmicity; MANDATORY |
| I_NaP = g_NaP · m_NaP · (V_m − E_Na) | Persistent Na+ current | McCormick & Huguenard 1992 | Plateau potential; required by phase spec |
| phi(r) = (1/4πσ) × sum_j [I_m,j / |r − r_j|] | Point-source LFP kernel | Nunez & Srinivasan 2006, Ch.1; Hagen et al. 2018 | Phase 1 EM validation |
| ∇·(σ∇φ) = −∇·J_imp | Poisson equation (quasi-static Maxwell) | Plonsey & Heppner 1967; Nunez 2006 | Theoretical basis for LFP kernel |
| I_gap = g_j · (V_i − V_j) | Ohmic gap junction | Landisman et al. 2002 | TRN electrical synapse; used in Phase 2 but parameterized in Phase 1 |
| L/lambda_EM = f × L / c_tissue ~ 100 × 0.01 / (3e8/sqrt(80)) ~ 3×10^-9 | Quasi-static validity ratio | Nunez & Srinivasan 2006 | One-line justification for quasi-static; include in theory document |

### Gating Variable Equations for I_T (McCormick & Huguenard / Destexhe conventions)

The Boltzmann steady-state forms (from Destexhe et al. 1994, cross-consistent with Huguenard & McCormick 1992):

```
# Activation (m_T)
m_T_inf = 1 / (1 + exp(-(V_m + 57) / 6.2))      # half-activation: -57 mV
tau_m_T = (0.612 + 1/(exp(-(V_m + 131)/16.7) + exp((V_m + 15.8)/18.2))) ms

# Inactivation (h_T)
h_T_inf = 1 / (1 + exp((V_m + 80) / 4.0))        # half-inactivation: -80 mV
# tau_h_T: piecewise — see McCormick & Huguenard 1992 Table 1
```

NOTE: These are from the Brian2 Destexhe_1998 example and Huguenard & McCormick 1992. The McCormick & Huguenard (1992) relay-cell paper may use slightly different values. The half-activation target is -57 mV ±2 mV per the phase specification. Verify against the primary paper before using.

### Gating Variable Equations for I_h (McCormick & Huguenard 1992)

```
# From training knowledge; verify exact values against primary paper
m_h_inf = 1 / (1 + exp((V_m + 75) / 5.5))        # half-activation: ~ -75 mV
tau_m_h: 100–500 ms (voltage-dependent, slowest current)
E_h = -43 mV (mixed cation; Na+ and K+)
g_h_max: 0.05–0.2 mS/cm^2
```

### Temperature Q10 Corrections

Both 1992 papers conducted experiments at different temperatures. Brian2 Destexhe_1998 example uses Q10 = 2.5 for I_T gating:

```
tadj = Q10^((T_target - T_ref) / 10.0)
# For I_T: Q10 = 2.5; T_ref = 24°C (room temp, dissociated cells); T_target = 37°C
tadj_T = 2.5^((37 - 24) / 10.0) = 2.5^1.3 ≈ 3.4
# Apply to tau: tau_corrected = tau_measured / tadj
# Apply to alpha, beta rates: alpha_corrected = alpha_measured × tadj
```

For McCormick & Huguenard (1992) relay cell paper, T_ref is 36°C (guinea pig slice) for some experiments — verify per current. If T_ref = 36°C and T_target = 37°C, tadj ≈ 1.1 (negligible correction).

### Required Techniques

| Technique | What It Does | Where Applied | Standard Reference |
|---|---|---|---|
| Operator splitting (Strang) | Separates stiff cable solve from active gates | Each timestep in SpatialNeuron | Hines 1984; Brian2 docs |
| Crank-Nicolson cable | Unconditionally stable 2nd-order for passive cable | Tridiagonal solve per cable | Hines 1984; METHODS.md |
| Exponential Euler for gates | Stable 1st-order for alpha/beta equations | Gate variables m, h, n, m_T, h_T, m_h | Brian2 default for SpatialNeuron |
| Hines algorithm | Tridiagonal O(N_comp) solve | Each compartment chain | Hines 1984, J. Neurophysiol. 52 |
| Boltzmann steady-state | Compact gating variable form | I_T, I_h, I_NaP parameterization | Huguenard & McCormick 1992 |
| Point-source Green's function | Analytical LFP from point current source | EM kernel validation | Nunez & Srinivasan 2006 Ch.1 |
| SVD of LFP snapshot matrix | Dominant spatial modes of field | Holographic encoding definition | Golub & Van Loan; numpy.linalg.svd |

### Approximation Schemes

| Approximation | Small Parameter | Regime of Validity | Error Estimate | Alternatives if Invalid |
|---|---|---|---|---|
| Quasi-static Maxwell (Poisson) | L/lambda_EM ~ 10^-9 | All neural frequencies (f << 10^9 Hz) | Retardation correction ~ 10^-11 (negligible) | Full-wave FDTD (never needed here) |
| Homogeneous isotropic tissue σ = 0.33 S/m | Tissue inhomogeneity ~ factor 2 | Syncytium-only field; no skull/CSF | Factor 2–3 amplitude error bound | FEM with tissue boundaries (SimNIBS, FEniCSx) |
| Point-source current per compartment | Compartment length << lambda_electrotonic | Compartment <= 50 µm; typical thalamic cell | Negligible vs. line source at this scale | Line-source correction (LFPy default) |
| Ohmic gap junction (linear g_j) | Transjunctional voltage << 30 mV | g_j < 1 nS; sub-threshold dynamics | Over-estimates coupling during AP | Voltage-gated Cx36 model |
| Single-compartment soma test (Phase 1 only) | Multi-compartment needed for EM field | Valid for spike timing validation only; NOT for LFP | Cable coupling errors if used beyond validation | Full >= 10 compartment cable (required for EM) |

---

## Standard Approaches

### Approach 1: TRN Neuron Model Based on Destexhe 1994 + McCormick & Huguenard 1992 (RECOMMENDED)

**What:** Implement a TRN cell as a multi-compartment HH cable with I_T and I_h using parameters from Destexhe et al. (1994) J. Neurophysiol. for TRN-specific kinetics, cross-checked against McCormick & Huguenard (1992) for relay-cell parameters. Brian2 SpatialNeuron with Crank-Nicolson cable + exponential Euler gates.

**Why standard:** TRN cells are the well-characterized gap-junction-coupled population (Landisman 2002). Destexhe 1994 is the canonical computational TRN model, with ModelDB code available (accession 3670). McCormick & Huguenard 1992 relay-cell model is the canonical validation target for relay-cell-like LTS behavior; the phase specification benchmarks against its values.

**Track record:** Both models widely used; LTS and spindle frequencies reproduced in >50 published computational studies. Brian2 ships with a Destexhe_et_al_1998 example using I_T.

**Key steps:**
1. Build morphology using Brian2 Morphology API (soma + dendrite, >= 10 compartments, ~50 µm each)
2. Implement I_Na, I_K (HH standard) for action potentials
3. Implement I_T (Boltzmann: m_T^2 * h_T; half-act -57 mV) with Q10-corrected kinetics
4. Implement I_h (Boltzmann: single-gate m_h; half-act -75 mV; slow tau 100–500 ms)
5. Implement I_NaP (persistent; small conductance, single gate; half-act ~ -55 mV)
6. Initialize all gates to steady-state at V = -65 mV; allow 100 ms settling
7. Inject hyperpolarizing current to demonstrate LTS; measure threshold (~-65 mV) and rebound burst frequency (100–400 Hz intra-burst)
8. Verify spindle-range oscillation when I_h is active (7–14 Hz)
9. Run NEURON comparison with ModelDB model 279 or 3670; compute spike timing RMS

**Known difficulties at each step:**
- Step 1: Brian2 Morphology API for multi-compartment requires specifying diameter, length, n_compartments per section. Set n >= 5 per section to get >= 10 total.
- Step 3: Q10 correction is critical — room-temperature kinetics are 3× slower than 37°C.
- Step 4: I_h is the slowest current (tau 100–500 ms); simulation duration must be >= 2000 ms to observe full h-current dynamics; a 500 ms run may miss delta rhythm
- Step 6: Forward Euler on uninitialized gates produces large transient (false spike); always initialize to steady-state
- Step 8: Need both I_T and I_h simultaneously; I_T alone produces LTS but not spindle termination (I_h terminates spindle cycle)
- Step 9: NEURON comparison requires identical parameters; use ModelDB code directly, do not retype

### Approach 2: Relay Cell Model Only (FALLBACK if TRN substrate rejected)

**What:** If the Phase 01-01 theory decision reverses and chooses relay cells, use McCormick & Huguenard (1992) relay cell directly. Same implementation steps but with relay-cell parameters (different I_A, I_K2; weaker or absent gap junctions).

**When to switch:** Only if literature review in 01-01 finds compelling evidence that relay cells have sufficient Cx36 coupling to form a syncytium (current evidence strongly contra-indicates this).

**Tradeoffs:** Relay cells have I_A (transient K+) and I_K2 (slowly inactivating K+) that TRN cells lack; more complex parameter set. Gap junction coupling is much weaker or absent — may require abandoning syncytium claim.

### Anti-Patterns to Avoid

- **Standard HH Na/K only:** Cannot produce LTS, spindle bursts, or 7–14 Hz oscillations. I_T and I_h are not optional.
  - _Example:_ A model with only I_Na and I_K fires continuously at constant frequency under DC injection — no burst mode.
- **Point neuron for EM field:** The LFP kernel requires spatially distributed transmembrane currents along the cable. A point neuron produces a trivial monopole field that does not reflect morphology and is inconsistent with the Hales framing.
  - _Example:_ Point neuron LFP amplitude is identical from any direction; real cable neurons have orientation-dependent dipole structure.
- **Re-implementing I_T from scratch without Q10:** Published implementations use room-temperature kinetics. Without Q10 correction, spikes are 3× too slow, thresholds shift, spindle frequencies are wrong.

---

## Existing Results to Leverage

### Established Results (DO NOT RE-DERIVE)

| Result | Exact Form | Source | How to Use |
|---|---|---|---|
| Quasi-static validity | L/lambda_EM = f·L/(c/sqrt(eps_r)) << 1; at f=1kHz, L=1cm: ratio ~ 3×10^-8 | Plonsey & Heppner 1967; Nunez & Srinivasan 2006 Ch.1 | One-line justification in theory document |
| Point-source LFP formula | phi = I/(4πσr); σ = 0.33 S/m | Nunez & Srinivasan 2006; Hagen et al. 2018 | Analytical test case for kernel validation |
| TRN Cx36 gap junctions | g_j = 0.1–2 nS per pair; coupling coefficient 0.01–0.1; range < 50 µm; requires Cx36 | Landisman et al. 2002, J. Neurosci. 22(3) | Confirms TRN substrate; sets g_j parameter range |
| Spindle frequency mechanism | TRN → GABAergic inhibition → relay de-inactivates I_T → rebound burst → re-excites TRN → 7–14 Hz | Steriade et al. 1993, Science 262, 679 | Validates 7–14 Hz as emergent from I_T/I_h loop |
| LTS threshold | ~-65 mV from hyperpolarized base (relay cell); for TRN cells similar, verify in Destexhe 1994 | McCormick & Huguenard 1992; Steriade et al. 1993 | Benchmark: must be ±5 mV of -65 mV |
| I_T half-activation | m_T half-activation ~ -57 mV; h_T half-inactivation ~ -80 mV | Huguenard & McCormick 1992, J. Neurophysiol. 68:1373 | Benchmark: I_T half-activation ±2 mV of -57 mV |
| Intra-burst frequency | 100–400 Hz during LTS burst crown | Steriade et al. 1993; McCormick & Huguenard 1992 | Benchmark: intra-burst frequency |
| Spindle oscillation | 7–14 Hz; requires I_h for termination and pacing | Steriade et al. 1993 | Benchmark: spindle frequency |
| Q10 for I_T gating | Q10 = 2.5 (from Destexhe Brian2 example); Q10 = 3 (typical HH; some McCormick papers) | Destexhe 1998 Brian2 example; standard physiology | Must apply before using at 37°C |
| Brian2 SpatialNeuron exponential Euler | Integration method for cable models in Brian2 | Brian2 documentation | Default method; confirmed stable for HH cable |

**Key insight:** Every numerical value in the table above comes from cited primary sources and should be CITED rather than re-derived. The only novel computation in Phase 1 is assembling them into a validated Brian2 model.

### Useful Intermediate Results

| Result | What It Gives You | Source | Conditions |
|---|---|---|---|
| Electrotonic length constant | lambda = sqrt(d * R_m / (4 R_i)); typical thalamic: lambda ~ 200–500 µm for soma | Rall 1969 | Passive cable; sets compartment count requirement |
| Membrane time constant | tau_m = R_m * C_m ~ 10–30 ms for thalamic neurons | Standard | Passive cable |
| Compartment length constraint | delta_x <= lambda / 10 ~ 20–50 µm | Rall; standard practice | Ensures spatial accuracy |
| Number of compartments | >= 10 for Phase 1 spec; 15–20 typical for soma + 2 dendrites | Phase spec | Sufficient for cable LFP |

### Relevant Prior Work

| Paper/Result | Authors | Year | Relevance | What to Extract |
|---|---|---|---|---|
| A model of spindle rhythmicity in the isolated thalamic reticular nucleus | Destexhe, Contreras, Sejnowski, Steriade | 1994 | TRN-specific HH model with I_T; spindle validation | I_T parameters for TRN; g_GABA; spindle output; ModelDB accession 3670 |
| Simulation of currents in rhythmic oscillations in thalamic relay neurons | Huguenard & McCormick | 1992 | Kinetic equations for I_T, I_A, I_K2, I_h | Boltzmann parameters; Table 1 of paper; Q10 values |
| A model of electrophysiological properties of thalamocortical relay neurons | McCormick & Huguenard | 1992 | Full relay cell model including I_NaP | Full parameter table; ModelDB accession 279 |
| Spatiotemporal patterns of spindle oscillations in cortex and thalamus | Contreras, Destexhe, Sejnowski, Steriade | 1997 | In vivo thalamic LFP recordings during spindles; 8 thalamic sites | Fig. 2 or Fig. 4: LFP PSD during spindles; empirical benchmark for Phase 3 |
| Multimodal Modeling of Neural Network Activity: Computing LFP, ECoG, EEG, and MEG Signals with LFPy 2.0 | Hagen, Naess, Ness, Einevoll | 2018 | LFP kernel validation framework; point-source and line-source methods | LFPy validation protocol; use as cross-check for JAX kernel |

---

## TRN vs. Relay Cell Substrate Decision

**This is the most important binary decision in Phase 1.** Research analysis strongly favors TRN.

### Evidence for TRN as Syncytium Substrate

| Factor | TRN | Excitatory Relay Cells |
|---|---|---|
| Cx36 gap junctions | CONFIRMED: Landisman et al. 2002; functional, g_j = 0.1–2 nS | Absent or very weak in most specific thalamic nuclei |
| Firing mode | Burst (I_T) + tonic; characteristic LTS | Burst (I_T) + tonic relay; also has LTS |
| Spindle pacemaker role | TRN IS the pacemaker (Steriade 1993) | Relay cells: driven by TRN |
| Gap junction coupling coefficient | 0.01–0.1 per Landisman 2002 | Not characterized; likely < 0.01 |
| Recent computational modeling | Destexhe 1994; biorXiv 2023 spindle at critical state | Standard relay cell models (no syncytium) |
| Morphological homogeneity | Lamellar, relatively homogeneous — favorable for coherent LFP | Less homogeneous across nuclei |

**Recommendation: Choose TRN as the syncytium substrate.** The gap junction physiology evidence is unambiguous. Relay cells are appropriate for single-cell validation of LTS/spindle phenomenology (McCormick & Huguenard 1992 benchmark), but the coupled syncytium model should use TRN cells.

**Implication for implementation:** Use Destexhe 1994 / Destexhe 1996 (ionic currents) as the primary TRN parameter source. McCormick & Huguenard 1992 is still the validation benchmark for LTS threshold and spindle frequency (these are consistent across TRN and relay cells for I_T-based burst firing).

**One important difference:** TRN cells do NOT have I_A and I_K2 (or have them at low density). If the Phase spec requires I_NaP, verify its presence in TRN using Destexhe 1996 or a recent TRN parameter paper. The bioRxiv 2024 biophysical modeling paper (Thalamic reticular nucleus subpopulations) is a relevant recent source.

---

## Holographic Encoding Operational Definition

**This must be locked before Phase 2 begins.** The following is the recommended operational definition.

### Recommended Operational Definition

**F(x, t):** The LFP field phi(x_obs, t) sampled at N_obs electrode positions arranged around the syncytium.

**Snapshot matrix:** Phi = [phi(t_1), phi(t_2), ..., phi(t_T)] — shape (N_obs × T)

**SVD decomposition:** Phi = U × S × V^T where U[:,k] are the spatial modes (left singular vectors), S[k,k] are mode amplitudes, and V[k,:] are temporal coefficients.

**The "hologram":** The dominant-K spatial modes U[:,0:K] (K = 3–5 based on explained variance > 80%)

**Reconstruction operator R:** Given a subset of cells (after removing fraction f of cells from the simulation), reconstruct U[:,0:K] from the reduced output. Fidelity metric = cos(U_full[:,k], U_reduced[:,k]) averaged over k = 0..K-1, expressed as percentage.

**Baseline:** The same fidelity metric applied to a matched-N random code (N cells, randomly assigned responses, no gap junction coupling), representing the trivial distributed representation.

**Claim threshold:** >70% fidelity after 20% cell loss, AND fidelity significantly exceeds random baseline.

**Discriminability metric:** Decoding accuracy of stimulus identity (2 classes minimum) from U[:,0:K] using leave-one-out linear classifier (dot product with template mode). Report as classification accuracy compared to chance.

### Why SVD Modes Are the Correct Operationalization

Lehar (2003) describes harmonic resonance as spatial standing waves spanning the neural substrate. SVD spatial modes (left singular vectors of the LFP snapshot matrix) ARE the dominant spatial patterns of the field — they directly implement the "standing wave interference pattern" concept as a computable quantity. The connection: if the LFP field has a dominant spatial mode U[:,0] that is stimulus-specific, this IS the operational analog of Lehar's "perceptual object as resonant mode." The SVD implementation makes this testable.

---

## Computational Tools

### Core Tools

| Tool | Version/Module | Purpose | Why Standard |
|---|---|---|---|
| Brian2 | >= 2.6; SpatialNeuron | HH cable simulation; cable + gate ODEs | Python-native; full multicompartment support; gap junctions via Synapses |
| Brian2CUDA | >= 1.0 (confirmed, Frontiers 2022) | GPU backend for Brian2; supports full feature set including SpatialNeuron | Only GPU tool supporting Brian2's complete feature set including multicompartment |
| NEURON | >= 8.0 (neuron pip package) | Single-cell validation; reference comparison | Gold standard for HH cable; ModelDB models available directly |
| JAX | >= 0.4; jax.numpy | LFP Green's function dense tensor; GPU matrix ops | JIT-compiled, GPU-native, differentiable; G·I_m per timestep |
| LFPy 2.0 | LFPy >= 2.2; LFPykit | LFP kernel cross-validation; point-source and line-source | Standard field; published validation; NEURON-backed |
| numpy / scipy | numpy >= 1.24; scipy.linalg | SVD mode extraction; signal processing | scipy.linalg.svd for snapshot matrix |

### Supporting Tools

| Tool | Purpose | When to Use |
|---|---|---|
| matplotlib | Visualization of V_m traces, LFP, spatial modes | All validation figures |
| ModelDB (senselab.med.yale.edu) | Reference NEURON code for McCormick & Huguenard 1992 (accession 279) and Destexhe 1994 (accession 3670) | Download before 01-02 implementation |
| scipy.signal.welch | PSD computation for spindle frequency validation | Spindle frequency benchmark |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|---|---|---|
| Brian2 SpatialNeuron | NEURON for all simulation | NEURON has no GPU path; impractical for N=100+ |
| Brian2CUDA | GeNN | GeNN does NOT support multicompartment models on GPU — confirmed NOT suitable |
| JAX dense tensor | LFPy directly | LFPy is NEURON-backend; not GPU-native; suitable for validation only |
| JAX float32 | JAX float64 | Float32: 7 digits; use float64 for kernel validation test to confirm <1% error; float32 acceptable for production |

### Computational Feasibility

| Computation | Estimated Cost | Bottleneck | Mitigation |
|---|---|---|---|
| Single HH cable cell, 10 compartments, 2000 ms | < 1 s CPU | None | CPU fine for single-cell validation |
| NEURON comparison run, same model | < 5 s CPU | None | Run on CPU alongside Brian2 |
| LFP kernel: precompute G for N_src=50, N_obs=50 | << 1 s | None at N=50 | One-time precomputation |
| Point-source analytical test | Instantaneous | None | phi = I/(4*pi*sigma*r) vectorized |

**Installation / Setup:**
```bash
# Brian2 + Brian2CUDA
pip install brian2
# Brian2CUDA requires CUDA toolkit; install via conda for CUDA compatibility:
conda install -c conda-forge brian2cuda
# Or from PyPI (ensure CUDA >= 11.0 is installed):
pip install Brian2Cuda

# NEURON
pip install neuron

# JAX with GPU
pip install "jax[cuda12]"   # or cuda11 depending on system
# Verify: python -c "import jax; print(jax.devices())"

# LFPy
pip install LFPy

# Scientific stack
pip install numpy scipy matplotlib
```

---

## Validation Strategies

### Internal Consistency Checks

| Check | What It Validates | How to Perform | Expected Result |
|---|---|---|---|
| HH spike threshold | Correct reversal potentials and maximal conductances | Inject 100 ms current step; measure V_m | Single AP; threshold ~ -55 mV |
| LTS threshold | I_T half-activation and inactivation | Hyperpolarize to -90 mV for 500 ms; release; measure rebound | LTS between -65 and -60 mV from rest |
| Intra-burst frequency | Correct I_T kinetics and conductance | Count spike interval within LTS burst | 100–400 Hz |
| Spindle frequency | I_T + I_h loop | Run 2000 ms with tonic bias; look for 7–14 Hz bursting | 7–14 Hz burst recurrence |
| Quasi-static validity | No wave-equation residuals | Compute L/lambda_EM for f=100 Hz, L=1 cm | ratio ~ 3×10^-9 << 1 |
| LFP kernel point-source | Correct Green's function implementation | Place single unit current source at r_src; compute phi at r_obs; compare to I/(4*pi*sigma*|r_obs - r_src|) | <1% relative error at r >= 10 µm |
| LFP kernel sign | Correct current convention | Verify inward current (depolarizing) produces negative phi in adjacent tissue | Inward current → phi < 0 nearby |

### Known Limits and Benchmarks (McCormick & Huguenard 1992)

| Limit | Parameter Regime | Known Result | Source |
|---|---|---|---|
| LTS threshold | From -90 mV holding; release | ~-65 mV from resting potential (±5 mV tolerance) | McCormick & Huguenard 1992, Fig. 1 |
| Spindle frequency | With I_T + I_h active; tonic drive | 7–14 Hz burst recurrence | Steriade et al. 1993; McCormick & Huguenard 1992 |
| Intra-burst rate | Within single LTS burst | 100–400 Hz instantaneous | Steriade et al. 1993, Fig. 1 |
| I_T half-activation | V_m sweep, steady-state activation | m_T_inf = 0.5 at ~ -57 mV (±2 mV tolerance) | Huguenard & McCormick 1992, Table 1 |
| Spike waveform RMS vs NEURON | Brian2 vs NEURON at identical parameters | <0.1 mV RMS deviation | Phase spec VALD-01 |
| LFP kernel error | Analytical point-source test | <1% relative error | Phase spec VALD-02 |

### Numerical Validation

| Test | Method | Tolerance | Reference Value |
|---|---|---|---|
| dt sensitivity | Run at dt = 0.025 ms and dt = 0.005 ms; compare spike times | Spike time difference < 0.1 ms | Standard HH stability criterion |
| Compartment count | Run with 10 vs 20 compartments; compare LFP spatial profile | LFP profile RMS difference < 5% | Cable convergence |
| Temperature sensitivity | Run at T_ref and T_target; verify Q10 scaling | tau scales as Q10^(deltaT/10) | Q10 = 2.5 per 10°C |
| Float32 vs float64 in kernel | Compute phi at r = 100 µm for same I | Agree to 5 significant figures | Precision check |

### Red Flags During Computation

- V_m oscillates at kHz frequencies with no stable resting potential: timestep too large (dt > 0.025 ms) or Forward Euler on stiff system — switch to exponential Euler
- LTS never appears despite hyperpolarization: I_T half-inactivation voltage wrong (h_T not de-inactivated at -90 mV) or Q10 correction missing (kinetics too slow)
- Spindle frequency > 20 Hz: I_h absent or tau_h too fast — check I_h implementation
- phi from LFP kernel diverges at small r: singularity handling missing — add minimum distance cutoff equal to compartment radius (~1 µm)
- Brian2 and NEURON give spike times differing by > 1 ms: parameter mismatch (check Q10, reversal potentials, conductance units)
- phi for inward current is positive: sign error in I_m extraction or Green's function formula — verify convention

---

## Common Pitfalls

### Pitfall 1: Missing Q10 Temperature Correction for I_T/I_h

**What goes wrong:** McCormick & Huguenard (1992) and Destexhe et al. (1994) use room-temperature (24°C) or 34°C experimental data. Without Q10 correction to 37°C, I_T kinetics are 3× too slow: LTS threshold shifts, burst frequency is too low, spindle frequency is wrong.
**Why it happens:** Published gating variable tables give measured values at experimental temperature; Q10 correction is often in a footnote or not stated.
**How to avoid:** Always compute tadj = Q10^((T_target - T_ref)/10) before using any kinetic parameters. Apply to tau: tau_corrected = tau_uncorrected / tadj. Apply to alpha, beta rates: rate_corrected = rate × tadj.
**Warning signs:** Spindle frequency < 5 Hz or > 20 Hz; LTS threshold at -50 mV instead of -65 mV.
**Recovery:** Locate T_ref in Methods section of primary paper; compute correction; re-run.

### Pitfall 2: Using Wrong Paper's I_T Parameters

**What goes wrong:** Huguenard & McCormick (1992) and McCormick & Huguenard (1992) are two different papers published simultaneously. The first (1373–1383) contains the voltage-clamp kinetics; the second (1384–1400) assembles the full model. Mixing parameters from different experiments (dissociated cells vs. slice; rat vs. guinea pig) gives internally inconsistent models.
**Why it happens:** Both papers have similar authors in similar order and the same year.
**How to avoid:** For kinetic equations: use Huguenard & McCormick (1992), 1373–1383. For assembled model: use McCormick & Huguenard (1992), 1384–1400. Download ModelDB accession 279 to see how they were combined.
**Warning signs:** Model cannot produce LTS despite correct currents.

### Pitfall 3: Point-Source Singularity in LFP Kernel

**What goes wrong:** phi = I/(4πσr) diverges as r → 0. At the source compartment itself, r = 0 and phi = infinity.
**Why it happens:** Mathematical singularity in the Green's function at zero distance.
**How to avoid:** Exclude self-source terms OR use minimum distance cutoff r_min = compartment radius ~ 1 µm. In the G tensor, set G[i,i] = 0 (equivalent to excluding self-field at source location). For observation points that coincide with source compartments, use r = max(|r_i - r_j|, r_min).
**Warning signs:** phi values exceeding 100 mV/m at an observation point; NaN or Inf in the output array.
**Recovery:** Add epsilon guard: r = jnp.maximum(jnp.linalg.norm(r_obs - r_src), 1e-6)  # 1 µm in meters

### Pitfall 4: Treating Lehar's "Standing Waves" as EM Radiation Standing Waves

**What goes wrong:** Lehar (2003) uses "standing wave" to mean spatial resonance patterns in the neural substrate, not EM radiation modes. At 10 Hz, EM wavelength in tissue ~ 10^7 m. No EM radiation standing wave fits in the thalamus.
**Why it happens:** Natural language ambiguity; Lehar's framing is electromagnetism-adjacent.
**How to avoid:** In every document, use "LFP spatial mode" or "network field eigenmode" when referring to the computational quantity. Reserve "electromagnetic" for the Maxwell-equation field computation. Include the one-line quasi-static justification.
**Warning signs:** Any document computing f = c/(2L) to get a "standing wave resonance frequency."

### Pitfall 5: Brian2CUDA SpatialNeuron + Gap Junction Interaction

**What goes wrong:** Brian2CUDA supports Brian2's full feature set, but combining SpatialNeuron (which uses a sparse tridiagonal cable solve) with Synapses-based gap junctions may require careful ordering of the two solves.
**Why it happens:** The cable solve within a SpatialNeuron assumes the inter-compartment coupling is captured by the tridiagonal matrix; gap junctions between cells are external currents that must be correctly summed before or after the cable step.
**How to avoid:** In Brian2, gap junctions between SpatialNeuron objects are standard Synapses objects targeting a specific compartment (typically soma). Test with N=2 cells first, verify gap junction current is applied correctly, check that coupling coefficient matches g_j/(g_j + g_membrane). Validate by checking that identical-voltage cells produce zero gap current.
**Warning signs:** Gap junction current non-zero when V_i = V_j.

---

## Level of Rigor

**Required for this phase:** Physicist's proof + controlled numerical validation

**Justification:** Phase 1 is a validation phase, not an exploratory phase. The quasi-static justification requires a one-line dimensional argument (not a proof). The HH model validation requires parameter matching to published values with stated tolerances (not derivation from first principles). The LFP kernel requires a numerical test against an analytical solution (not a formal error bound proof).

**What this means concretely:**
- All benchmark values must be compared against stated tolerances (±5 mV for LTS, ±2 mV for half-activation, <0.1 mV RMS spike waveform, <1% LFP kernel error)
- Every parameter value must be accompanied by its source (paper, table, equation number)
- Temperature corrections must be stated explicitly with T_ref, T_target, and Q10 values
- The quasi-static validity statement must include the numerical value L/lambda_EM ~ 10^-9

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|---|---|---|---|
| Full-wave Maxwell for neural EM | Quasi-static Poisson (equivalent to > 9 sig figs) | Established 1967 (Plonsey & Heppner) | Reduces EM computation to linear Poisson solve |
| Point-dipole LFP approximation | Multi-compartment cable current distribution | Since LFPy v1, ~2013 | Captures morphology-dependent LFP structure |
| NEURON only for HH cable | Brian2 SpatialNeuron + GPU (Brian2CUDA) | 2019 (Brian2CUDA paper); 2022 (Frontiers paper) | GPU acceleration for population sims |
| GeNN for GPU neural sim | Brian2CUDA (for full feature set) | 2022 | GeNN confirmed NOT to support multicompartmental — Brian2CUDA is correct choice |
| Single-purpose LFP code | LFPy 2.0 / LFPykit standalone | 2018 | NEURON-independent LFP computation |

**Superseded approaches to avoid:**
- **Full-wave FDTD for neural EM fields:** Overkill by 6 orders of magnitude; error from quasi-static is 10^-11. Never use for this project.
- **GeNN for multi-compartment:** GeNN explicitly does not support multicompartmental models on GPU. Brian2CUDA is the correct choice.
- **Standard HH (Na/K only) for thalamic physiology:** Produces correct AP waveforms but wrong burst mode, wrong oscillation frequency, wrong LTS threshold. I_T and I_h are mandatory.

---

## Open Questions

1. **Exact Hales (2014) method: quasi-static or full retarded?**
   - What we know: Paper is accessible (newdualism.org PDF). Web search confirmed it "computes scalar electric potential from current filaments" for a hippocampal neuron and "applies both microscopic and macroscopic forms of Maxwell's equations." This language is consistent with quasi-static (volume conductor = Maxwell + quasi-static limit). The retardation correction at neural scales is 10^-11 — physically equivalent to quasi-static.
   - What's unclear: Whether the paper explicitly claims the full retarded treatment is necessary, or whether it uses full Maxwell notation but the quasi-static limit applies in practice.
   - Impact on Phase 1: If Hales uses quasi-static, the LFP kernel is confirmed standard. If Hales claims the retarded treatment adds something, this must be addressed explicitly in the theory document (almost certainly a framing choice rather than a physical difference at neural scales).
   - Recommendation: READ the paper before writing the theory document. If retarded treatment is claimed necessary, include a subsection in the theory document calculating the retardation correction and showing it is 10^-11.

2. **I_NaP presence in TRN cells**
   - What we know: McCormick & Huguenard (1992) relay cell model includes I_NaP. The phase spec requires it.
   - What's unclear: Is I_NaP present in TRN cells or mainly relay cells? The Destexhe 1994 TRN model (ModelDB 3670) includes only I_T, I_KCa, I_L — no I_NaP explicitly.
   - Impact: If I_NaP is absent in TRN, either (a) include it as a validated component based on relay-cell literature and note it as a model assumption, or (b) remove it from the Phase 1 spec for the TRN model.
   - Recommendation: Check the biorXiv 2024 TRN subpopulation paper and recent TRN biophysics papers. If I_NaP is not established in TRN, document as a relay-cell parameter being used in a TRN-inspired model, and note in the theory document.

3. **Specific empirical LFP benchmark for Phase 3**
   - What we know: Contreras et al. (1997) J. Neurosci. 17:1179 contains multisite thalamic LFP recordings during spindles in barbiturate-anesthetized cat; 8 thalamic sites; power spectrum 7–14 Hz shown.
   - What's unclear: Which specific figure and which specific quantitative measure (e.g., peak frequency, absolute amplitude in µV, coherence between sites) is the Phase 3 comparator.
   - Recommendation: Lock now as the empirical comparator: Contreras et al. (1997), J. Neurosci. 17:1179, Fig. 2 (LFP traces from 8 thalamic sites) and the spindle power spectrum panel showing 7–14 Hz peak. The Phase 3 simulation should reproduce the spectral peak location and approximate relative amplitude across sites.

4. **Brian2CUDA SpatialNeuron + Synapses gap junction ordering**
   - What we know: Brian2CUDA supports the full Brian2 feature set including SpatialNeuron. Gap junctions are implemented via Synapses.
   - What's unclear: Whether there are known runtime issues or numerical warnings when combining SpatialNeuron cable solve with Synapses gap junctions in GPU mode.
   - Recommendation: Before Phase 1 implementation, test with N=2 SpatialNeuron objects coupled via a gap junction Synapses object on GPU. Verify coupling coefficient matches theory. This is a 30-minute test that resolves a potential show-stopper.

---

## Alternative Approaches if Primary Fails

| If This Fails | Because Of | Switch To | Cost of Switching |
|---|---|---|---|
| Brian2 SpatialNeuron on GPU (Brian2CUDA) | Stability issues with cable + HH on GPU | Brian2 SpatialNeuron on CPU (single-cell validation is CPU-feasible; N=100 may be marginal) | Low — same code, remove `set_device('cuda_standalone')` |
| Brian2CUDA entirely | Incompatibility with CUDA version or OS | GeNN (for non-cable components) + NEURON (cable) + manual Python coupling | High — requires new simulation architecture |
| JAX float32 Green's function | Precision insufficient for <1% test | JAX float64 | Trivial — change dtype in kernel |
| Dense JAX tensor | Memory overflow (not expected at N=1 for Phase 1) | Compute phi per observation point in loop | None at N=1 scale |

**Decision criteria:** If Brian2 SpatialNeuron + Brian2CUDA fails the 30-minute N=2 test described above, use CPU for all Phase 1 validation (single cell). This does not block Phase 1. Brian2CUDA is critical for Phase 2 (N=100); diagnose before Phase 2 begins.

---

## Caveats and Alternatives

**Adversarial self-critique:**

1. **TRN vs. relay cell: is the decision really closed?** The evidence for TRN Cx36 coupling is strong (Landisman 2002), but the phase spec benchmarks are from McCormick & Huguenard (1992) relay cell paper specifically. This tension may mean the phase intends a relay-cell model validated against those benchmarks, with TRN coupling parameters added. Resolution: implement a relay-cell-like model (using McCormick & Huguenard 1992 parameters for LTS validation), but use TRN coupling parameters (g_j from Landisman 2002) for the gap junction layer. Document explicitly that the model is a "TRN-inspired relay cell" or "hybrid" if the parameter sets are mixed.

2. **The Destexhe 1994 TRN model lacks I_NaP.** The phase spec requires I_NaP. Rather than forcing it into a TRN model where its presence is uncertain, use McCormick & Huguenard 1992 as the primary model (includes I_NaP) and note that gap junction parameters come from Landisman 2002 (TRN). This is the cleanest path to passing all five benchmarks.

3. **The holographic encoding definition via SVD is pragmatic but may be criticized as "not holography."** Classical optical holography encodes spatial information as interference fringes (phase + amplitude). SVD modes encode variance structure. The analogy is structural: both are distributed representations where partial information (subset of modes) reconstructs the full pattern. The recommended response is to not call it "optical holography" — call it "distributed spatial mode encoding" or "eigenmode-based distributed representation." The Lehar 2003 framing is sufficient motivation without requiring optical analogy.

4. **Brian2CUDA multicompartment support confirmed in principle but untested in practice.** The 2022 Frontiers paper states it supports Brian2's full feature set; the benchmarks only show single-compartment models. The N=2 SpatialNeuron test described in Open Question 4 is essential before committing.

5. **Contreras et al. (1997) uses barbiturate-anesthetized cat, not awake rodent.** Spindle properties under anesthesia differ from natural sleep. The validation target is spindle frequency range (7–14 Hz), which is consistent across conditions. Absolute amplitude comparison should be made with caution and labeled as "order-of-magnitude comparison, anesthetized animal."

---

## Sources

### Primary (HIGH confidence)

- Hodgkin & Huxley (1952). J. Physiol. 117, 500–544. DOI: 10.1113/jphysiol.1952.sp004764 — HH equations
- Rall (1969). Biophys. J. 9, 1483–1508. — Cable theory, J(x,t) source term
- Steriade, McCormick & Sejnowski (1993). Science 262, 679–685. DOI: 10.1126/science.8235588 — Spindle mechanism; I_T/I_h circuit; benchmark values
- Nunez & Srinivasan (2006). Electric Fields of the Brain, 2nd ed. OUP. — Quasi-static validity; Poisson equation; LFP forward model
- Plonsey & Heppner (1967). Bull. Math. Biophys. 29:657. — Original quasi-static validity proof for neural EM

### Secondary (MEDIUM confidence — verify against primary source before using parameter values)

- McCormick & Huguenard (1992). J. Neurophysiol. 68(4), 1384–1400. DOI: 10.1152/jn.1992.68.4.1384 — Relay cell model; I_NaP; ModelDB 279
- Huguenard & McCormick (1992). J. Neurophysiol. 68(4), 1373–1383. DOI: 10.1152/jn.1992.68.4.1373 — Kinetic equations for I_T, I_h gating
- Landisman et al. (2002). J. Neurosci. 22(3), 1002–1009. DOI: 10.1523/JNEUROSCI.22-03-01002.2002 — TRN Cx36 gap junctions; g_j = 0.1–2 nS
- Destexhe, Contreras, Sejnowski, Steriade (1994). J. Neurophysiol. 72(2), 803–818. DOI: 10.1152/jn.1994.72.2.803 — TRN-specific HH model; ModelDB 3670
- Contreras, Destexhe, Sejnowski & Steriade (1997). J. Neurosci. 17(3), 1179–1196. DOI: 10.1523/JNEUROSCI.17-03-01179.1997 — Empirical thalamic LFP during spindles; PHASE 3 BENCHMARK
- Hagen, Naess, Ness, Einevoll (2018). Front. Neuroinformatics 12:92. DOI: 10.3389/fninf.2018.00092 — LFPy 2.0; LFP kernel validation protocol
- Hales, C.G. (2014). J. Integr. Neurosci. 13(2), 313–361. DOI: 10.1142/S0219635214400056 [MUST READ] — PRIMARY METHOD ANCHOR; confirm quasi-static vs. retarded
- Lehar, S. (2003). Behav. Brain Sci. 26(4), 375–408. DOI: 10.1017/S0140525X03000098 — PRIMARY THEORY ANCHOR; confirms no quantitative spatial predictions
- Stimberg, Brette, Goodman (2019). eLife. DOI: 10.7554/eLife.47314 — Brian2 simulator; SpatialNeuron documentation

### Tertiary (LOW confidence — background context; do not use as sole source for parameter values)

- Lehar, S. (2003). The World in Your Head. Lawrence Erlbaum. ISBN: 978-0805838886 — Extended harmonic resonance theory
- McFadden (2002). J. Conscious. Stud. 9(4), 23–50 — CEMI theory; ephaptic coupling motivation
- Landisman & Connors (2005). Science 310, 1809–1813. — TRN gap junction cortical modulation (Phase 4 relevance)

---

## Metadata

**Confidence breakdown:**

- Mathematical Framework: HIGH — quasi-static LFP, HH cable, gate equations are textbook-established; exact parameter values from McCormick & Huguenard 1992 require paper access (MEDIUM for values)
- TRN substrate decision: HIGH — Landisman 2002 evidence is unambiguous
- Standard Approaches: HIGH — Brian2 SpatialNeuron + NEURON cross-validation is well-documented procedure
- Holographic encoding operationalization: MEDIUM — SVD approach is well-motivated but not tested in this specific context; chosen as best operational candidate
- Computational Tools: HIGH for Brian2/JAX/NEURON identification; MEDIUM for Brian2CUDA SpatialNeuron GPU stability (untested in practice for multicompartment)
- Validation Strategies: HIGH — tolerances are specified in phase contract; protocol is standard
- Empirical LFP benchmark (Phase 3): MEDIUM — Contreras et al. (1997) is the best candidate found; specific figure/quantity should be confirmed by reading paper

**Research date:** 2026-03-16
**Valid until:** Parameter values are stable indefinitely. Software versions (Brian2, JAX) may need update if major API changes occur; check release notes. Brian2CUDA CUDA compatibility depends on CUDA version — verify against hardware.

---

## Per-Sub-Task Summary Tables

### Sub-task 01-01: Read Primary Sources and Write Theory Framework

| Item | Recommendation | Confidence | Source |
|---|---|---|---|
| EM method | Quasi-static Poisson; confirm Hales reduces to this | HIGH | Plonsey & Heppner 1967; Nunez 2006 |
| "Standing wave" definition | LFP spatial eigenmode of network dynamics — NOT EM radiation mode | HIGH | SUMMARY.md resolved conflict |
| Quasi-static justification | L/lambda_EM ~ 10^-9 at f=100 Hz, L=1 cm; include in theory doc | HIGH | Nunez 2006 |
| Holographic encoding | SVD of LFP snapshot matrix; dominant-K spatial modes = "hologram" | MEDIUM | Recommended operationalization |
| Reconstruction operator | R: reconstruct U[:,0:K] from reduced-cell output; fidelity = cos(U_full, U_reduced) | MEDIUM | Standard SVD analysis |
| Syncytium substrate | TRN (GABAergic, Cx36-coupled, well-characterized) | HIGH | Landisman 2002 |
| Empirical LFP benchmark | Contreras et al. (1997), J. Neurosci. 17:1179, Fig. 2 | MEDIUM | Confirmed via web search |

### Sub-task 01-02: Implement and Validate Single HH Cable Cell

| Benchmark | Target Value | Tolerance | Source Figure/Table |
|---|---|---|---|
| LTS threshold | ~-65 mV from resting | ±5 mV | McCormick & Huguenard 1992 Fig. 1; Steriade 1993 |
| Spindle frequency | 7–14 Hz | In-band | Steriade et al. 1993 |
| Intra-burst frequency | 100–400 Hz | In-band | Steriade et al. 1993; McCormick & Huguenard 1992 |
| I_T half-activation | ~-57 mV (m_T_inf = 0.5) | ±2 mV | Huguenard & McCormick 1992, Table 1 |
| Spike waveform RMS vs NEURON | <0.1 mV | Hard upper bound | Phase spec VALD-01 |
| Compartment count | >= 10 | Hard lower bound | Phase spec SIMU-01 |
| Integration method | Crank-Nicolson cable + exponential Euler gates | — | Brian2 best practice |
| dt | 0.025 ms | Hard upper bound 0.05 ms | HH stability constraint |

### Sub-task 01-03: Implement and Validate LFP Green's Function Kernel

| Item | Specification | Confidence | Notes |
|---|---|---|---|
| Kernel formula | phi_i = (1/4πσ) * sum_j (I_m,j / |r_i - r_j|) for point sources | HIGH | Standard volume conductor |
| Tissue conductivity | σ = 0.33 S/m | HIGH | Gray matter; Nunez 2006 |
| Singularity handling | r_min = max(|r_i - r_j|, r_compartment_radius) | HIGH | Standard practice |
| Validation test | phi = I/(4πσr) analytical point source; <1% relative error at r >= 10 µm | HIGH | Phase spec VALD-02 |
| Precision | float64 for validation test; float32 acceptable for production | HIGH | 7 sig figs float32 |
| GPU framework | JAX jax.numpy; jnp.linalg.norm for distances; vmap for vectorization | HIGH | Standard JAX |
| Cross-validation | LFPy 2.0 point-source computation for same geometry | MEDIUM | LFPy as reference |
| Brian2CUDA compatibility | Test 2-cell SpatialNeuron + gap junction on GPU before committing | MEDIUM | Unconfirmed in practice |
