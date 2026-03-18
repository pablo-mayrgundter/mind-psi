---
plan: 01-01
phase: 01-theory-framework-and-single-cell-validation
status: complete
---

# Summary: Plan 01-01 — Theory Framework

## Outcome

`docs/THEORY-FRAMEWORK.md` (421 lines, 7 sections) was created in a prior session and verified
complete in this session. The document locks all four theoretical foundations required before any
multi-cell simulation begins: LFP as the target observable (scalp EEG explicitly ruled out),
quasi-static Poisson as the EM method (numerically justified), TRN as the syncytium substrate
(Landisman 2002 Cx36 evidence), and holographic encoding as a concrete SVD reconstruction
operator with a quantified fidelity criterion.

## Key Results

- Quasi-static ratio: L/λ_EM = 2.98×10⁻⁷ at f=1 kHz, L=10 mm, ε_r=80 — confirmed << 1 (retardation correction ~10⁻¹³ relative; Poisson equation is exact at neural scales)
- TRN substrate: locked (Landisman 2002 J. Neurosci. 22(3):1002–1009; Cx36 gap junctions; g_j = 0.1–2 nS; coupling coefficient K = 0.01–0.1; relay cells explicitly rejected)
- Holographic encoding: F(x,t) = φ(x_obs,t); Φ = U·S·V^T ∈ ℝ^(N_obs×T); H = U[:,0]; ρ = |U[:,0]^T U_M[:,0]| ≥ 0.70 criterion after 20% cell removal; 50-replicate Monte Carlo random-code baseline required
- Phase 3 benchmark: Contreras et al. (1997) J. Neurosci. 17(3):1179–1196; Figure 4 PSD; 7–14 Hz spindle band target; in-vivo thalamic LFP; barbiturate-anesthetized cat

## Acceptance Tests

- test-quasi-static-ratio: PASS — L/λ_EM = 2.98×10⁻⁷ (in required range [1e-8, 1e-6]); numerical derivation in Section 2.2; Hales (2014) Eq. 6 confirmed quasi-static Poisson
- test-encoding-operator: PASS — all 6 required elements present: F(x,t) def (§4.1), Φ matrix (§4.2), SVD decomposition Φ=USV^T (§4.3), H=U[:,0:K] (§4.4), fidelity ρ≥0.70 (§4.6), 50-replicate random-code baseline (§4.7)
- test-trn-decision: PASS — Landisman et al. (2002) J. Neurosci. 22(3):1002–1009 cited; g_j=0.1–2 nS; relay cells explicitly rejected for absent/weak Cx36 (§3.2–3.3)
- test-benchmark-lock: PASS — Contreras et al. (1997) J. Neurosci. 17(3):1179–1196 cited; Figure 4 identified; 7–14 Hz spindle band target stated (§1.3, §6.3)

## Deliverables

- docs/THEORY-FRAMEWORK.md: 421 lines, 7 sections (committed in chore: commit pre-rate-limit work from Phase 1 execution, hash c32fb64)

## Requirements Met

- DERV-01: PASS — quasi-static Maxwell derivation (§2) and standing wave definition locked as LFP spatial eigenmode U[:,0] (§5.3); no EM radiation interpretation possible
- DERV-02: PASS — holographic encoding operator fully specified (§4): Φ=U·S·V^T, H=U[:,0:K], ρ=cosine_sim≥0.70, random-code baseline procedure; implementation deferred to Phase 2
- DERV-04: PARTIAL — substrate decision locked (TRN, Cx36, g_j=0.1–2 nS); formal controller coupling law deferred to Phase 2

## Forbidden Proxy Status

- fp-quasi-static-unconfirmed: RESOLVED — Hales (2014) Eq. 6 confirmed quasi-static Poisson; no retarded potential; no deviation required
- fp-standing-wave-ambiguous: RESOLVED — "standing wave" exclusively defined as U[:,0] dominant SVD spatial mode of LFP field (§5.3); enforced in CONVENTIONS.md Phase 0
- fp-encoding-undefined: RESOLVED — concrete reconstruction operator Φ_M→U_M[:,0] and fidelity ρ≥0.70 criterion in §4; fully computable formula, not verbal description

## Reference Anchor Actions

- Ref-Hales (Hales 2014 J. Integr. Neurosci. 13(2):313–361): read, confirmed quasi-static, cited — COMPLETE
- Ref-Lehar (Lehar 2003 Behav. Brain Sci. 26(4):375–408): operationalized via SVD mapping, cited — COMPLETE
- Ref-EEG-empirical (Contreras et al. 1997 J. Neurosci. 17(3):1179–1196): cited, Figure 4 identified, 7–14 Hz target stated — COMPLETE
- Ref-Landisman (Landisman et al. 2002 J. Neurosci. 22(3):1002–1009): cited, g_j range noted, TRN substrate locked — COMPLETE

## Blockers/Open Questions

None. All four foundations are locked. Plans 01-02 (Brian2 TRN cable implementation) and 01-03
(JAX LFP Green's function kernel) can proceed without theoretical ambiguity. The Lehar (2003)
SVD operationalization caveat is documented in §5.2 — if quantitative spatial mode predictions
from Lehar conflict with SVD results in Phase 2, the conflict must be logged before Phase 3.
