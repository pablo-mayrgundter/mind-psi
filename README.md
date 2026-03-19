# mind-psi

_Developing a physical [theory of thalamo-cortical function](https://sites.google.com/site/pablomayrgundter/mind) with [PBC's Get Physics Done AI assistant](https://github.com/psi-oss/get-physics-done)._

**Core question:** Can a Maxwell-equation treatment of the endogenous Electromagnetic (EM) field of a parametric thalamic syncytium (Hodgkin-Huxley (HH) cable neurons, Connexin 36 (Cx36) gap junction coupling, cortical column controllers) produce EM manifold dynamics that holographically encode 3D sensory inputs and reproduce biologically observed Local Field Potential (LFP) oscillatory signatures correlated with stimulus identity?

The hypothesis: the thalamus operates as a folding electromagnetic manifold. Cortical columns drive it as computational controllers. The collective LFP of N gap-junction-coupled thalamic reticular nucleus (TRN) cells produces spatial standing-wave modes (Steven Lehar's harmonic resonance) that encode a holographic model of the sensed world — recoverable with high (eg ≥70%) fidelity after moderate (eg 20%) cell loss.

## Status — 2026-03-17

**Phase 1 complete with 3 open items.**

| Phase | Status |
|---|---|
| **1 · Theory Framework + Single-Cell Validation** | ⚠️ Complete (1 FAIL, 3 PENDING — see below) |
| 2 · N=100 Syncytium + Gap Junction Sweep | 🔲 Not started |
| 3 · Stimulus-Specific Spatial Mode Encoding | 🔲 Not started |
| 4 · N=1,000 Scaling Test | 🔲 Not started |

### Phase 1 Go/No-Go

| Criterion | | Value |
|---|---|---|
| GO-01 dt convergence | ✅ PASS | 0.035 ms < 0.1 ms |
| GO-02 LFP spatial convergence | ✅ PASS | 1.86% < 5% at N_comp=10 |
| GO-03 V_LTS threshold | ✅ PASS | −66.25 mV ∈ [−70, −60] mV |
| GO-04 I_T half-activation | ✅ PASS | −56.98 mV ∈ [−59, −55] mV |
| GO-05 f_spindle | ✅ PASS | 10.00 Hz ∈ [7, 14] Hz |
| GO-06 intra-burst frequency | ❌ FAIL | Single AP (needs I_A current) |
| GO-07 spike timing | ✅ PASS | 0.040 ms < 0.1 ms |
| GO-08 NEURON waveform RMS | ⏳ PENDING | modeldb.yale.edu unavailable |
| GO-09 LFP sign | ✅ PASS | φ > 0 for positive I_src |
| GO-10 LFP accuracy | ✅ PASS | 0.00e+00 relative error |
| GO-11 Brian2CUDA spike count | ⏳ PENDING | Brian2CUDA not installed |
| GO-12 Brian2CUDA spike timing | ⏳ PENDING | Brian2CUDA not installed |
| GO-13 gap junction zero-current | ✅ PASS | 0.000e+00 A |

**GO-06** requires adding A-type K⁺ current (I_A, Huguenard & Prince 1992). The LTS mechanism and I_T/I_h dynamics are validated; multi-spike burst structure requires rapid K⁺ inactivation not present in standard HH.

---

## Research Design

### Three-Layer Architecture

```
Layer 3 (L3): Phase-space meta-optimization
     ↕ (future)
Layer 2 (L2): Cortical column controllers
     ↕  efferent modulation of g_j and membrane drive
Layer 1 (L1): Thalamic TRN syncytium — N HH cable cells, Cx36 gap junctions
               ↓
         LFP field φ(x,t) = (1/4πσ) Σⱼ Iₘ,ⱼ / |r − rⱼ|
               ↓
         SVD spatial modes → holographic encoding
```

### Key Locked Definitions

- **Observable**: LFP (local field potential), NOT scalp EEG. N~100–1000 thalamic cells cannot produce scalp-detectable amplitudes; target is extracellular field at 100–1000 µm.
- **EM approximation**: Quasi-static Poisson — ∇·(σ∇φ) = −∇·J_imp; σ = 0.33 S/m. Retardation ratio L/λ_EM = 2.98×10⁻⁷ at f=1 kHz, L=10 mm. Full Maxwell adds nothing.
- **Standing wave**: LFP spatial eigenmode — dominant left singular vector U[:,0] of snapshot matrix Φ. NOT an EM radiation mode. Spatial period set by network architecture.
- **Holographic encoding**: Φ = U·S·Vᵀ; hologram H = U[:,0:K]; fidelity ρ = Pearson(H, H_reconstructed) ≥ 0.70 after 20% random cell removal; compared to matched-N random code baseline.
- **Syncytium substrate**: TRN (thalamic reticular nucleus), Cx36 gap junctions, g_j = 0.1–2 nS. Relay cells have absent/weak Cx36 and cannot form a syncytium.
- **Empirical benchmark**: Contreras et al. (1997) J. Neurosci. 17:1179 — multisite thalamic in-vivo LFP during spindles; 7–14 Hz target band.

---

## Repository Layout

```
.
├── docs/
│   └── THEORY-FRAMEWORK.md        ← locked foundations (LFP, quasi-static, SVD encoding, TRN)
├── simulations/
│   ├── hh_cable_cell.py            ← Brian2 SpatialNeuron HH cable (I_Na/K/L/T/h/NaP, Q10→37°C)
│   ├── lfp_kernel.py               ← JAX float64 LFP Green's function kernel
│   ├── brian2cuda_stability_test.py
│   ├── lfp_kernel_env_check.py
│   ├── neuron_reference/
│   │   └── parameter_table.md     ← McCormick & Huguenard (1992) parameter table
│   ├── analysis/                  ← NPZ data archives (convergence, LTS, spindle, cross-val)
│   └── figures/                   ← validation figures (convergence, LTS threshold, spindle, cross-val)
├── analysis/                      ← LFP kernel + Brian2CUDA validation archives
├── figures/                       ← LFP kernel accuracy + gap junction stability figures
└── .gpd/                          ← research management (plans, summaries, conventions, state)
    ├── PROJECT.md
    ├── ROADMAP.md
    ├── REQUIREMENTS.md
    ├── CONVENTIONS.md
    ├── STATE.md
    └── phases/
        └── 01-theory-framework-and-single-cell-validation/
            ├── 01-RESEARCH.md
            ├── 01-EXPERIMENT-DESIGN.md
            ├── 01-01-SUMMARY.md   ← theory framework
            ├── 01-02-SUMMARY.md   ← HH cell validation
            └── 01-03-SUMMARY.md   ← LFP kernel + GPU test
```

---

## Documents

### Theory

| Document | Description |
|---|---|
| [docs/THEORY-FRAMEWORK.md](docs/THEORY-FRAMEWORK.md) | Locked foundations: LFP observable, quasi-static derivation, SVD holographic encoding operator, TRN decision, Phase 3 benchmark |
| [.gpd/CONVENTIONS.md](.gpd/CONVENTIONS.md) | All locked conventions: units, sign conventions, observable, encoding, simulation timestep |
| [.gpd/REQUIREMENTS.md](.gpd/REQUIREMENTS.md) | Full requirement set (DERV-01–04, SIMU-01–04, CALC-01–03, VALD-01–04) |

### Roadmap & Planning

| Document | Description |
|---|---|
| [.gpd/ROADMAP.md](.gpd/ROADMAP.md) | 4-phase roadmap with success criteria and contract coverage |
| [.gpd/STATE.md](.gpd/STATE.md) | Current research state, go/no-go table, open questions, decisions |
| [.gpd/PROJECT.md](.gpd/PROJECT.md) | Scope contract, anchor registry, in/out-of-scope boundaries |

### Phase 1 Artifacts

| Document | Description |
|---|---|
| [.gpd/phases/01-theory-framework-and-single-cell-validation/01-RESEARCH.md](.gpd/phases/01-theory-framework-and-single-cell-validation/01-RESEARCH.md) | Domain research: TRN substrate confirmation, Brian2CUDA SpatialNeuron support, Hales quasi-static, benchmark identification |
| [.gpd/phases/01-theory-framework-and-single-cell-validation/01-EXPERIMENT-DESIGN.md](.gpd/phases/01-theory-framework-and-single-cell-validation/01-EXPERIMENT-DESIGN.md) | 6 experiments, 13 hard-block + 3 advisory go/no-go criteria |
| [.gpd/phases/01-theory-framework-and-single-cell-validation/01-01-SUMMARY.md](.gpd/phases/01-theory-framework-and-single-cell-validation/01-01-SUMMARY.md) | Theory framework — all 4 acceptance tests PASS |
| [.gpd/phases/01-theory-framework-and-single-cell-validation/01-02-SUMMARY.md](.gpd/phases/01-theory-framework-and-single-cell-validation/01-02-SUMMARY.md) | HH cable cell — 7/8 criteria assessed PASS; GO-06 FAIL (I_A needed); GO-08 PENDING |
| [.gpd/phases/01-theory-framework-and-single-cell-validation/01-03-SUMMARY.md](.gpd/phases/01-theory-framework-and-single-cell-validation/01-03-SUMMARY.md) | LFP kernel — GO-09/10/13 PASS; GO-11/12 PENDING (Brian2CUDA) |

### Background Research

| Document | Description |
|---|---|
| [.gpd/research/PRIOR-WORK.md](.gpd/research/PRIOR-WORK.md) | Prior work: CEMI theory, thalamo-cortical oscillation models, LFP computation methods |
| [.gpd/research/METHODS.md](.gpd/research/METHODS.md) | Methods survey: Brian2, JAX, LFPy, gap junction implementations |
| [.gpd/research/COMPUTATIONAL.md](.gpd/research/COMPUTATIONAL.md) | Computational landscape: scaling, GPU requirements, memory estimates |
| [.gpd/research/PITFALLS.md](.gpd/research/PITFALLS.md) | Known pitfalls: EM amplitude scale gap, holographic operationalization, quasi-static confusion, g_j sensitivity |

---

## Simulation Code

### [`simulations/hh_cable_cell.py`](simulations/hh_cable_cell.py)
Brian2 SpatialNeuron implementation of the thalamic TRN relay cell.
- **Currents**: I_Na, I_K, I_L, I_T (low-threshold Ca²⁺), I_h (hyperpolarization-activated), I_NaP (persistent Na⁺)
- **Cable**: 500 µm, 10 µm diameter, 10 compartments (50 µm each)
- **Temperature**: Q10 corrections to 37°C (T_ref=24°C for all gates, McCormick & Huguenard 1992)
- **Validated**: LTS threshold −66.25 mV, f_spindle 10 Hz, I_T half-activation −56.98 mV
- **Known limitation**: Multi-spike burst (GO-06) requires I_A current not yet implemented

### [`simulations/lfp_kernel.py`](simulations/lfp_kernel.py)
JAX float64 LFP Green's function kernel.
- **Formula**: φ(r) = (1/4πσ) · Σⱼ Iₘ,ⱼ / |r − rⱼ|; σ = 0.33 S/m
- **Validated**: 0.00e+00 relative error vs. analytical at r = [100 µm, 1 mm, 10 mm]
- **Features**: Dense G tensor, singularity guard at r_min=1 µm, float64 enforced

---

## Conventions (locked)

```
V_m = V_intracellular − V_extracellular;  resting ≈ −65 mV
Current sign: outward-positive;  C_m dV/dt = −I_Na − I_K − I_L − I_T − I_h − I_NaP + I_ext
h-gate: fraction NOT inactivated;  h_Na_inf(−65 mV) = 0.596
Gap junction: I_gap,i = g_j·(V_i − V_j);  outward-positive from cell i
LFP units: V/m or µV (never mV);  SI throughout (m, A, S/m, V)
dt ≤ 0.025 ms;  initialize gates to steady-state at −65 mV;  100 ms settling before recording
```

Full convention lock: [.gpd/CONVENTIONS.md](.gpd/CONVENTIONS.md)

---

## Next Steps

1. **Resolve GO-06** — add I_A (A-type K⁺, Huguenard & Prince 1992) to enable multi-spike LTS burst at 100–400 Hz intra-burst frequency
2. **Retry NEURON cross-validation** — modeldb.yale.edu timed out; retry for GO-08
3. **Install Brian2CUDA** on GPU-equipped machine for GO-11/12
4. **Plan Phase 2** — N=100 TRN syncytium with gap junctions; g_j sweep 0.1–10 nS; LFP spatial mode extraction

---

## References

- Hales, C. (2014). Origins of the brain's endogenous electromagnetic field. *J. Integr. Neurosci.* 13(2):313–336. — EM field method anchor
- Lehar, S. (2003). Harmonic resonance theory. *Behav. Brain Sci.* 26(4):375–408. — Holographic encoding theory
- McCormick, D.A. & Huguenard, J.R. (1992). *J. Neurophysiol.* 68(4):1384–1400. — HH model parameters
- Huguenard, J.R. & McCormick, D.A. (1992). *J. Neurophysiol.* 68(4):1373–1383. — I_T, I_h kinetics
- Landisman, C.E. et al. (2002). *J. Neurosci.* 22(3):1002–1009. — TRN Cx36 gap junctions
- Contreras, D. et al. (1997). *J. Neurosci.* 17:1179–1196. — Phase 3 empirical LFP benchmark
- Steriade, M., McCormick, D.A. & Sejnowski, T.J. (1993). *Science* 262:679–685. — Spindle oscillation physiology
- Nunez, P.L. & Srinivasan, R. (2006). *Electric Fields of the Brain* (2nd ed.). OUP. — Quasi-static LFP theory
