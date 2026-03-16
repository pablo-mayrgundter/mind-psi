---
generated: 2026-03-16
domain: Thalamo-cortical EM manifold modeling
dimension: Known Results / Prior Work
confidence: MEDIUM (web search unavailable — DOIs marked for verification)
---

# Prior Work: Thalamo-Cortical EM Manifold Modeling

**Confidence:** MEDIUM — all results from training data (cutoff Aug 2025). Textbook physics results (HH, Maxwell, cable theory, EEG band definitions) are HIGH confidence. Paper-specific results (Hales 2014 exact amplitudes, Lehar 2003 quantitative predictions) are MEDIUM and must be verified against primary sources before use in derivations.

---

## Key Results Summary

| Result | Expression / Value | Source | Confidence |
|--------|-------------------|--------|------------|
| HH membrane current | I_Na=g_Na·m³h(V-E_Na), I_K=g_K·n⁴(V-E_K) | Hodgkin & Huxley 1952 | HIGH |
| HH resting params (vertebrate) | E_Na≈+55mV, E_K≈-90mV, E_L≈-70mV, C_m≈1μF/cm² | Standard textbook | HIGH |
| Passive cable equation | ∂V/∂t = λ²∂²V/∂x² - V/τ_m | Rall 1969 | HIGH |
| Quasi-static EM validity | Valid for f << σ/(2πε) ≈ 10⁹ Hz in tissue | Nunez & Srinivasan 2006 | HIGH |
| TRN gap junction conductance (Cx36) | g_j ≈ 0.1–2 nS per pair | Landisman et al. 2002 | MEDIUM |
| TRN coupling coefficient | V_post/V_pre ≈ 0.01–0.1 | Landisman et al. 2002 | MEDIUM |
| Sleep spindle frequency | 7–14 Hz; TRN–relay loop | Steriade et al. 1993 | HIGH |
| Alpha band | 8–12 Hz; thalamo-cortical | Berger 1929 / Nunez 2006 | HIGH |
| Gamma band | 30–80 Hz; stimulus-correlated | Gray & Singer 1989 | HIGH |
| I_T half-activation | V_half_act ≈ -57 mV, V_half_inact ≈ -81 mV | McCormick & Huguenard 1992 | MEDIUM |
| I_h half-activation | V_half ≈ -75 mV; τ ≈ 100–500 ms | McCormick & Huguenard 1992 | MEDIUM |
| FlyWire scale | ~140,000 neurons, ~50M synapses | Shiu et al., Nature 2024 | MEDIUM |

---

## Foundational Work

### Hodgkin & Huxley (1952) — Quantitative Membrane Current Model

Complete kinetic model of ionic conductances. Four coupled ODEs:
```
C_m dV/dt = -I_Na(m,h) - I_K(n) - I_L + I_ext
dm/dt = α_m(V)(1-m) - β_m(V)·m   [similarly for h, n]
```
Predicts propagating action potential. Squid axon; vertebrate thalamic neurons require re-parameterization.

**Missing for thalamic modeling:** I_NaP (persistent Na⁺), I_h (HCN), I_T (T-type Ca²⁺) — all critical for thalamic burst firing and oscillation.

**Reference:** Hodgkin & Huxley (1952). J. Physiol. 117, 500–544. DOI: 10.1113/jphysiol.1952.sp004764

---

### Rall (1967, 1969) — Cable Theory for Dendritic Trees

Passive cable equation; space constant λ = √(d·R_m/(4R_i)); branching condition d₁^(3/2) = d₂^(3/2) + d₃^(3/2). DC solution: V(x) = V₀·exp(-x/λ).

Determines J(x,t) spatial distribution — the Maxwell source term. Compartmental models are the standard extension to active dendrites.

**Reference:** Rall (1969). Biophys. J. 9, 1483–1508. DOI: 10.1016/S0006-3495(69)86435-9

---

### Steriade, McCormick & Sejnowski (1993) — Thalamocortical Oscillations

Mechanistic account of spindle generation: TRN GABAergic cells inhibit relay cells → de-inactivate I_T → relay cell rebound burst → re-excite TRN → 7–14 Hz cycle. I_h mediates rhythm termination and delta transition.

**Key parameters:**
- Spindle: 7–14 Hz; 3–8 spikes per LTS at 100–400 Hz intra-burst
- I_T LTS threshold: ~-65 mV from hyperpolarized base
- I_h sag time constant: ~100–500 ms

**Critical implication:** Standard HH (Na/K only) cannot reproduce thalamic EEG bands. I_T and I_h are mandatory additions.

**Reference:** Steriade et al. (1993). Science 262, 679–685. DOI: 10.1126/science.8235588

---

### McCormick & Huguenard (1992) — Thalamic Relay Cell Biophysics

Comprehensive HH-style model with I_T, I_h, I_NaP. Primary parameter source for thalamic relay cell modeling.

**Key parameters (guinea pig, room temperature):**
- I_T: g_T ≈ 0.5–2 mS/cm²; V_half_act ≈ -57 mV; V_half_inact ≈ -81 mV
- I_h: g_h ≈ 0.05–0.2 mS/cm²; V_half ≈ -75 mV; τ ≈ 100–500 ms
- Temperature correction: Q₁₀ ≈ 3 for gating kinetics

**Reference:** McCormick & Huguenard (1992). J. Neurophysiol. 68(4), 1384–1400. DOI: 10.1152/jn.1992.68.4.1384

---

### Landisman et al. (2002) — Electrical Synapses in the Thalamic Reticular Nucleus

First demonstration of functional Cx36 gap junctions in TRN. Paired patch-clamp in mouse slice.

**Key results:**
- Coupling coefficient: 0.01–0.1
- Junctional conductance: g_j = 0.1–2 nS per pair
- Coupling range: strongest within ~50 μm
- Cx36 knockout reduces TRN synchrony

**CRITICAL:** Results are for TRN (GABAergic), NOT excitatory relay cells. Gap junction coupling of relay cells is much weaker or absent in most specific thalamic nuclei. **The syncytium model may need to be TRN-based.**

**Reference:** Landisman et al. (2002). J. Neurosci. 22(3), 1002–1009. DOI: 10.1523/JNEUROSCI.22-03-01002.2002

---

### Landisman & Connors (2005) — Dynamic Modulation of TRN Gap Junctions

Gap junction coupling in TRN is dynamically modulated: mGluR1 → increased coupling; dopamine → decreased coupling. Timescale: minutes.

**Relevance:** Validates L2 conceptual framework. Cortical efferents (which release glutamate) can modulate mGluR1 → switch gap junction conductance → restructure syncytium topology. This is the biophysical mechanism for L2 "topology switching."

**Reference:** Landisman & Connors (2005). Science 310, 1809–1813. DOI: 10.1126/science.1114655

---

### Hales (2014) — Origins of the Brain's Endogenous Electromagnetic Field

**Primary method anchor for L1.** Maxwell-equation treatment of the endogenous EM field generated by neural cable currents. Multipole expansion from full distributed current distributions (axonal + dendritic).

**Key results (MEDIUM confidence — verify against paper):**
- Single action potential generates EM pulse: E ~ 0.1–1 V/m at ~100 μm; B ~ 0.1–1 pT
- Dominant contribution: transmembrane current distribution (radial), not axial current
- Temporal structure mirrors AP waveform (~1 ms)
- Full retarded potential treatment applied (vs. quasi-static)

**Limitations (critical):**
- Single neuron / very small population only — no syncytium treatment
- Does not address gap-junction-coupled populations
- No standing wave or holographic encoding analysis
- Physical necessity of full retarded treatment over quasi-static at neural scales (f < 1 kHz, r < 10 mm) requires scrutiny: retardation correction is ~10⁻¹¹ — quasi-static is physically equivalent unless Hales gives specific arguments otherwise

**MUST be read in full before Phase 1 planning.** The extension to N coupled cells is the core novel derivation of this project.

**Reference:** Hales, C.G. (2014). J. Integr. Neurosci. 13(2), 313–361. DOI: 10.1142/S021963521440013X [VERIFY]

---

### McFadden (2002) — CEMI Theory (Conscious Electromagnetic Information)

**Johnjoe McFadden's Conscious Electromagnetic Information** theory. Argues the brain's endogenous EM field is causally active in neural computation — not merely an epiphenomenal byproduct of neural activity.

**Core claims:**
- Coherent EM field dynamics integrate information across neural populations that are not directly synaptically connected
- The EM field can influence neuronal firing (via interaction with voltage-gated channels)
- EM field coherence = information integration = computational relevance
- Voluntary action involves the EM field "steering" neural firing across populations

**Key references:**
- McFadden, J. (2002a). Synchronous firing and its influence on the brain's electromagnetic field. J. Conscious. Stud. 9(4), 23–50.
- McFadden, J. (2002b). The conscious electromagnetic information (CEMI) field theory. J. Conscious. Stud. 9(8), 45–60.

**Theoretical position in this project:**
- CEMI supports L1 as a *causal* layer, not just a measurement: the EM field both encodes (Lehar/Pribram) AND acts back on the neural dynamics
- McFadden + Hales together provide the full L1 theoretical basis: Hales computes the field; McFadden argues it matters computationally
- Key for this project: if the EM field is causally active (CEMI), then the thalamic syncytium EM manifold is a two-way system — cortical columns shape the field, AND the field shapes thalamic firing (feedback loop)
- This bidirectional coupling is the physical basis for "folding the manifold" as an active computation

**Limitations:**
- No quantitative biophysical model connecting EM field amplitude to neural firing modulation
- Contested: EM field amplitudes in tissue (mV/m range) may be too small to significantly modulate neural firing above thermal noise
- Not yet incorporated into standard computational neuroscience

**Reference:** See above; Wikipedia overview: https://en.wikipedia.org/wiki/Electromagnetic_theories_of_consciousness

---

### Pribram (1971, 1991) — Holonomic Brain Theory

Proposes brain encodes memory/perception as distributed interference patterns (hologram analogy). Partial damage degrades but does not abolish memory. Fourier-domain encoding. Phase in oscillations encodes relational structure.

No precise quantitative predictions. Theoretical ancestor of Lehar's harmonic resonance theory.

**References:** Pribram (1971). Languages of the Brain. Prentice-Hall. | Pribram (1991). Brain and Perception. Lawrence Erlbaum. ISBN: 978-0898599954.

---

### Lehar (2003) — Harmonic Resonance Theory / Gestalt Isomorphism

**Primary theory anchor.** Proposes the brain creates a 3D volumetric world model via standing wave resonance. Perceptual objects = resonant modes. Gestalt grouping = constructive interference. Holographic redundancy → error correction.

**Key predictions:**
- Spatial periodicity of perceptual grouping (illusory contours) corresponds to standing wave spatial frequency
- Perceptual "folding" = topological deformation of the standing wave manifold
- 3D depth perception from volumetric standing waves

**Limitations (critical):**
- No quantitative predictions: no predicted spatial frequencies, wavelengths, or amplitudes in brain tissue
- Physical substrate unspecified — EM fields suggested but not derived
- No connection to measurable EEG signatures
- No derivation from Maxwell's equations or biophysical models
- Primarily phenomenological/psychophysical

**This project makes Lehar's theory falsifiable** by providing a biophysical implementation with testable EEG predictions.

**References:**
- Lehar, S. (2003). The World in Your Head. Lawrence Erlbaum. ISBN: 978-0805838886.
- Lehar, S. (2003). Gestalt isomorphism and the primacy of subjective conscious experience. Behav. Brain Sci. 26(4), 375–408. DOI: 10.1017/S0140525X03000098 [VERIFY]

---

### Gray & Singer (1989) — Stimulus-Specific Gamma Oscillations

Neurons responding to the same visual contour synchronize at 40–80 Hz; neurons responding to different contours do not. Stimulus-specific oscillatory synchrony as binding mechanism.

**Key results:**
- Gamma: 40–60 Hz in cat area 17; stimulus-specific cross-correlations
- Cortical in origin; thalamic contribution is relay timing, not autonomous gamma

**Relevance:** Establishes stimulus-specificity of oscillatory signatures as the key observable — directly relevant to the smoking-gun test.

**Reference:** Gray & Singer (1989). PNAS 86, 1698–1702. DOI: 10.1073/pnas.86.5.1698

---

### Linden et al. (2010) — Population LFP from Cable-Model Neurons

LFP from a population of multi-compartment cable neurons is strongly shaped by: (a) dendritic filtering and morphological orientation, (b) spatial arrangement, (c) population synchrony.

**Key result:** LFP amplitude scales with population synchrony, not simply N. Random dendritic orientations cause near-complete LFP cancellation.

**Relevance:** Critical for interpreting EEG-like signatures from the simulated thalamic syncytium. LFP depends critically on morphological orientation and synchrony — morphological parameters must be realistic.

**Reference:** Linden et al. (2010). PLoS Comput. Biol. 6(11): e1000972. DOI: 10.1371/journal.pcbi.1000972 [VERIFY]

---

### Nunez & Srinivasan (2006) — Electric Fields of the Brain

Quasi-static approximation valid for all neural EM < 10 kHz. Forward problem: ∇·(σ∇Φ) = -∇·J_imp. EEG requires ~10⁴–10⁵ synchronously firing neurons over cm² patches for scalp detection.

**Reference:** Nunez & Srinivasan (2006). Electric Fields of the Brain (2nd ed.). OUP. ISBN: 978-0195050387.

---

### Shiu et al. / FlyWire Consortium (2024) — Full Adult Fly Brain Simulation

~140,000 neurons, ~50M synapses at ms resolution. Simplified point-neuron (LIF) models; no HH cable. GPU cluster.

**CRITICAL CAVEAT:** FlyWire used point neurons. N~140,000 LIF ≠ N~10,000 HH cable — HH is ~4,000× more expensive per simulated second. FlyWire validates pipeline architecture, not HH cable scalability.

**Reference:** Shiu et al. (2024). Nature. DOI TBD [VERIFY].

---

## Critical Open Questions

1. **Relay cell vs TRN syncytium** — Gap junctions well-established in TRN, weak/absent in most excitatory relay cells. Which cell type forms the syncytium? This is a major unresolved design decision.

2. **"Standing waves" in thalamus** — At 10–80 Hz, free-space EM wavelength is 3,750–37,500 km — no EM radiation standing waves in a cm-scale thalamus. Lehar's "standing waves" must be LFP spatial modes (slow electromechanical waves in the tissue lattice). This distinction must be stated precisely.

3. **Quasi-static vs Hales retarded treatment** — At f < 1 kHz, r < 10 mm: retardation correction ~10⁻¹¹. Physical justification for full Maxwell over quasi-static must be established by reading Hales carefully before adoption.

4. **CEMI bidirectional coupling** — If the EM field modulates firing (CEMI), the system has feedback L1→neural→L1. Is the EM field amplitude in the syncytium large enough to significantly modulate Cx36 gating or voltage-gated channels? This is the quantitative test of CEMI applicability.

5. **LFP vs scalp EEG** — Local thalamic EM field is LFP-like, not scalp EEG. The smoking-gun comparison must use LFP power spectra (not scalp EEG bands) as the empirical comparator, or add a forward model to project to scalp.

6. **Lehar quantitative predictions** — No published spatial wavelength, oscillation frequency, or amplitude prediction from Lehar's theory in brain tissue. Must be derived from first principles.

---

## Notation Conventions

| Quantity | Symbol | Source |
|----------|--------|--------|
| Membrane potential | V_m | Standard |
| Sodium reversal potential | E_Na | Nernst/Hodgkin convention |
| Space constant | λ | Rall |
| Membrane time constant | τ_m | Rall |
| T-type Ca²⁺ current | I_T | Thalamic literature |
| HCN/h current | I_h | Thalamic literature |
| Impressed current density | J_imp | Nunez & Srinivasan |
| Gap junction conductance | g_j | Landisman |
| Connexin 36 | Cx36 | Protein symbol |
| LFP standing wave spatial frequency | k_LFP | Field notation |

---

## Key References

| Reference | DOI | Role |
|-----------|-----|------|
| Hodgkin & Huxley 1952 | 10.1113/jphysiol.1952.sp004764 | HH equations — L1 cell model |
| Rall 1969 | 10.1016/S0006-3495(69)86435-9 | Cable theory — J(x,t) |
| Steriade et al. 1993 | 10.1126/science.8235588 | Spindle circuit, I_T/I_h |
| McCormick & Huguenard 1992 | 10.1152/jn.1992.68.4.1384 | Thalamic relay cell params |
| Landisman et al. 2002 | 10.1523/JNEUROSCI.22-03-01002.2002 | TRN gap junctions |
| Landisman & Connors 2005 | 10.1126/science.1114655 | Gap junction modulation by cortex |
| **Hales 2014** | 10.1142/S021963521440013X [VERIFY] | **METHOD ANCHOR — endogenous EM field** |
| McFadden 2002a | J. Conscious. Stud. 9(4) | CEMI — EM field as causal integrator |
| McFadden 2002b | J. Conscious. Stud. 9(8) | CEMI theory full statement |
| **Lehar 2003 (BBS)** | 10.1017/S0140525X03000098 [VERIFY] | **THEORY ANCHOR — harmonic resonance** |
| Lehar 2003 (book) | ISBN 978-0805838886 | World in Your Head |
| Pribram 1991 | ISBN 978-0898599954 | Holonomic brain — theoretical context |
| Nunez & Srinivasan 2006 | ISBN 978-0195050387 | Quasi-static neural EM |
| Gray & Singer 1989 | 10.1073/pnas.86.5.1698 | Stimulus-specific gamma |
| Linden et al. 2010 | 10.1371/journal.pcbi.1000972 [VERIFY] | LFP from cable populations |
| Shiu et al. 2024 | TBD [VERIFY] | SCALE ANCHOR — FlyWire |

---

_Generated: 2026-03-16 | Confidence: MEDIUM | Web search unavailable — verify DOIs before use in derivations_
