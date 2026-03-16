---
generated: 2026-03-16
domain: Thalamo-cortical EM manifold modeling
dimension: Computational Approaches
confidence: MEDIUM (web versions unverified — marked [VERIFY])
---

# Computational Approaches: Thalamo-Cortical EM Manifold

## 1. HH Cable Simulation (N = 100–10,000)

### Recommended Stack: Brian2 + Brian2CUDA

**Brian2** (Stimberg et al. 2019, eLife 47314 [VERIFY version]) is the primary choice:
- Python-native, HH cable equations with multi-compartment support
- Gap junction (electrical synapse) API via `Synapses` object
- **Brian2CUDA** provides NVIDIA GPU backend for ODE integration

NEURON simulator is superior for single-cell morphological fidelity but has **no GPU ODE integration path** — use NEURON for single-cell validation only.

### Timestep Constraint

HH Na⁺ activation gate m has time constant ~0.1 ms at peak.

- **Use dt = 0.025 ms** (exponential Euler or RK4)
- **Never dt > 0.05 ms** — aliases the spike
- Crank-Nicolson cable: Hines (1984) tridiagonal algorithm, second-order in space and time, unconditionally stable for passive cable; operator splitting for active membrane

### Gap Junction Solve

Electrical synapses require implicit joint matrix solve at each step:
- **Sparse topology (tree/ring):** O(N) per step — tractable
- **Dense all-to-all:** O(N²) per step — expensive at N > 1,000

Brian2 handles via the `Synapses` object with `connect()` pattern.

---

## 2. EM Field Computation from Neural Currents

### Quasi-Static Approximation (Valid for Neural Frequencies)

At neural frequencies (< 1 kHz), the quasi-static approximation to Maxwell's equations holds:
- Electric field satisfies ∇²φ = -ρ/ε
- Wavelength >> tissue dimensions
- **Full-wave FDTD is overkill by ~6 orders of magnitude** — do not use

### Volume Conductor (Green's Function Method)

For homogeneous isotropic medium (σ ≈ 0.33 S/m for brain tissue):

```
φ = (p · r̂) / (4πσr²)
```

Implementation:
1. Precompute the N_src × N_obs Green's function tensor **G** once
2. Each timestep: single matrix-vector multiply φ = **G** · **I_m**
3. Cost per timestep: O(N²) — trivial for N ≤ 1,000

### Scaling for Large N

| N | Method | VRAM | Cost |
|---|--------|------|------|
| ≤ 1,000 | Dense Green's function tensor | < 4 MB | Trivial |
| ~ 3,000 | Dense tensor hits GPU VRAM wall | ~4 GB float32 | FMM required |
| 10,000 | Fast Multipole Method (FMM) | Reduced | FMM + multi-GPU |

**FMM options:** Greengard & Rokhlin (1987); ExaFMM-t, PVFMM [VERIFY availability].

**FEM (FEniCSx):** Only needed if skull/CSF/gray matter boundaries matter. For isolated thalamic syncytium modeling, homogeneous medium is sufficient to first order.

### Note on Hales Method

The "Hales endogenous EM field method" referenced in the project context (Hales, Colin — ResearchGate 2014) should be retrieved directly by the user. The volume conductor framework above is the standard approach consistent with that literature. **Action required: retrieve and read the specific Hales paper before Phase 1 planning.**

---

## 3. 3D Sensory Field Representation

### Visual: Light Field

Represent as 4D plenoptic function L(x, y, θ, φ) or modern implicit representation:
- **3D Gaussian Splatting (3DGS)** (Kerbl et al. 2023): GPU-native, differentiable, renders >100 fps on A100. Python via `gsplat` library [VERIFY]
- **JAX-NeRF:** If differentiability through renderer is needed for learning
- Test stimuli: virtual point sources / virtual screens acceptable *inside* the 3D field model — must not reduce the field model itself to 2D

### Auditory: Spatial Audio Field

- **Ambisonics:** Spherical harmonic decomposition, orders 1–7 for adequate spatial resolution. Python: `sounddevice` + `scipy` for HRTF [VERIFY]
- **pyroomacoustics:** Room simulation [VERIFY]
- **JAX FFT:** For GPU-batched HRTF convolution

---

## 4. GPU-Accelerated Neural Simulation Frameworks

| Framework | HH Cable | Gap Junctions | EM Integration | Recommended |
|-----------|----------|---------------|----------------|-------------|
| **Brian2 + Brian2CUDA** | Yes (native) | Yes (Synapses) | No (add JAX layer) | **Primary** |
| NEURON | Yes (best fidelity) | Yes | No — no GPU ODE | Validation only |
| GeNN | Yes | Limited | No | Fallback |
| PyNN | Yes (abstracted) | Yes | No | Not recommended |
| JAX custom | Manual | Manual sparse | Natural | EM layer / fallback |

**GeNN** (GPU-enhanced Neuronal Networks, Yavuz et al. 2016 [VERIFY]) compiles C++/CUDA directly — strongest fallback if Brian2CUDA proves unstable for multi-compartment cables.

---

## 5. Resource Estimates

| N | Wall time (est.) | VRAM | Feasibility |
|---|-----------------|------|-------------|
| 100 | ~1 s | ~200 MB | **Trivially tractable — hard constraint satisfied** |
| 1,000 | ~20–30 s | ~2 GB | Tractable on single A100 |
| 10,000 | ~500–1000 s | ~10 GB (neural) + FMM | Requires multi-GPU or HPC |

**Note:** These are estimates from domain knowledge, not benchmarks. Phase 1 must validate N = 100 tractability before committing to larger sweeps.

---

## 6. FlyWire/Shiu 2024 Comparison

**Critical caveat:** FlyWire used point LIF neurons (~130k cells), **not HH cable**:
- LIF is ~100× cheaper per step
- LIF allows ~40× larger timestep
- Combined: ~4,000× cheaper than HH cable per simulated second

**FlyWire validates data pipeline architecture, not HH scalability.** N = 130,000 LIF ≠ N = 130,000 HH cable. The feasibility bound for HH cable is N ~ 1,000–10,000, not 130,000.

FlyWire is relevant as: (a) connectome architecture guide, (b) software pipeline inspiration, (c) long-term growth target — not as a justification for large-N HH simulation.

---

## 7. Recommended Phase Architecture

| Phase | Task | N | Tools |
|-------|------|---|-------|
| 1 | Single-cell HH validation | 1 | Brian2, CPU, compare to NEURON |
| 2 | N=100 pilot + EM pipeline | 100 | Brian2CUDA + JAX Green's function |
| 3 | Gap junction syncytium | 100–500 | Sweep g_gap, find synchrony threshold |
| 4 | N=1,000 scaling test | 1,000 | Single-GPU feasibility boundary |
| 5 | EM standing wave extraction | 1,000 | SVD/DMD on E-field time series |
| 6 | Sensory manifold encoding | 1,000 | 3DGS (visual) + Ambisonics (audio) |
| 7 | N=10,000 (if Phase 4 justifies) | 10,000 | FMM-based EM, multi-GPU |

---

## 8. Open Questions / Actions Required

- [ ] Retrieve and read specific Hales (2014) endogenous EM field paper — confirm EM computation method
- [ ] Benchmark Brian2CUDA multi-compartment cable stability (Phase 1)
- [ ] Verify FMM GPU library availability: ExaFMM-t, PVFMM [VERIFY]
- [ ] Verify current Brian2CUDA version and CUDA compatibility [VERIFY]
- [ ] Decide on JAX vs Brian2CUDA as primary GPU framework for EM layer

---

_Generated: 2026-03-16 | Confidence: MEDIUM (web search blocked; versions marked [VERIFY])_
