# CMOS Inverter — Switching Threshold Analysis & Transistor Sizing Evolution

**Author:** Vinay Upreti | NIT Uttarakhand | B.Tech ECE (Analog IC Design)  
**Technology:** gpdk090 (90nm CMOS) | Tool: Cadence Virtuoso + Spectre  
**VDD:** 1.8V | **Date:** April 2026

---

## Overview

This document presents the complete analytical and experimental derivation of the CMOS inverter switching threshold (V_M), followed by a systematic transistor sizing evolution driven entirely by physical reasoning. The approach is telescopic — each sizing decision is motivated by an observable physical discrepancy, not by empirical iteration.

The central thesis:

> The condition for V_M = V_DD/2 and the condition for equal propagation delays are not independent optimizations. Both are governed by a single constraint: **k_n = k_p**. This convergence is not coincidental — it reflects the same underlying physics viewed from two different performance metrics.

---

## Part 1 — Transistor Sizing Evolution

### 1.1 — Initial Sizing: Why W_n = W_p = 120n

The design process begins with the smallest defensible assumption: **minimum symmetric sizing**.

**Rationale:**

In the absence of prior knowledge about the technology's mobility ratio, the most intellectually honest starting point is one that imposes no asymmetry between the pull-up and pull-down networks. Minimum width (120nm) at minimum length (100nm) in gpdk090 satisfies this criterion.

The implicit prediction at this stage:

```
Since µ_n > µ_p universally in CMOS:
k_n = µ_n × Cox × (W/L) > k_p = µ_p × Cox × (W/L)
at equal W/L

→ NMOS pull-down intrinsically stronger than PMOS pull-up
→ V_M expected to shift LEFT of V_DD/2
→ i.e. V_M < 900mV
```

**Simulation result at W_n = W_p = 120n, L = 100n:**

| Metric | Predicted | Observed | Explanation |
|--------|-----------|----------|-------------|
| V_M | < 900mV | 900mV exactly | Short-channel effects and V_th balance in gpdk090 compensate mobility mismatch at minimum sizing |
| VTC shape | Asymmetric | Symmetric | Equal sizing accidentally balanced at this node |
| Output swing | Rail-to-rail | Rail-to-rail | CMOS static behavior confirmed |

This result, while initially counterintuitive, establishes a critical baseline: **equal sizing at minimum dimensions yields symmetric VTC in gpdk090**. All subsequent deviations are measured against this reference.

---

### 1.2 — PMOS Width Sweep: Quantifying V_M Sensitivity

**Motivation:**

Having established the baseline, the next objective is to characterise how V_M responds to changes in transistor strength. PMOS is chosen as the sweep variable because it is the conventionally weaker device (due to µ_p < µ_n) and represents the primary control knob for V_M adjustment in standard CMOS design.

**Physical prediction before sweep:**

```
W_p↑ → more parallel conduction channels
      → k_p↑ (intrinsic transconductance increases)
      → I_d↑ for fixed V_SG
      → R_on,p = 1/k_p(V_SG - |V_tp|) DECREASES
         (consequence of k_p increase, not the cause)
      → V_out = V_DD × R_on,p/(R_on,p + R_on,n)
         → V_out↑ for same V_in
      → VTC body shifts upward and leftward
      → V_M decreases
```

**Critical clarification on causality:**

The resistor-divider model tempts the interpretation that "R_on,p decreases therefore current increases." This inverts the causal chain. The correct sequence is:

```
W_p↑ → k_p↑ → I_d↑ → R_on,p↓
```

Width increase is the physical cause. R_on reduction is the measurable effect — analogous to parallel resistors where adding a branch reduces total resistance as a consequence of the additional current path.

**Simulation observations — PMOS parametric sweep (W_p: 120n → 500n):**

| W_p | V_M direction | Physical explanation |
|-----|--------------|---------------------|
| 120n (baseline) | 900mV | Equal strengths |
| Increasing | Shifts LEFT | PMOS increasingly dominates pull-up |
| 500n | ~400mV | PMOS overwhelms NMOS |

**Key observation:** The V_M shift is non-linear with W_p. This is analytically expected — V_M depends on √(k_n/k_p), introducing a square-root compression that produces diminishing returns as W_p increases.

---

### 1.3 — NMOS Width Sweep: Validating the Tug-of-War Model

**Motivation:**

To fully characterise the symmetry of the sizing problem, the NMOS sweep is performed independently. This validates whether both transistor types produce the same direction of V_M shift, and more importantly, whether the physical model — pull-up vs pull-down tug of war — is consistent.

**Physical prediction:**

```
W_n↑ → k_n↑ → I_d↑ → R_on,n↓
      → V_out = V_DD × R_on,n/(R_on,p + R_on,n) DECREASES
         (opposite direction to PMOS case)
      → V_out↓ for same V_in
      → VTC body shifts downward and leftward
      → V_M decreases
```

**Simulation observations — NMOS parametric sweep:**

| Parameter | PMOS W↑ | NMOS W↑ |
|-----------|---------|---------|
| Network strengthened | Pull-UP | Pull-DOWN |
| V_out for same V_in | Increases | Decreases |
| VTC body movement | Upward + left | Downward + left |
| V_M direction | Decreases | Decreases |
| Physical mechanism | PMOS wins tug of war | NMOS wins tug of war |

Both sweeps shift V_M left — but through opposite V_out trajectories. This confirms that V_M is governed by the **ratio of transistor strengths**, not by absolute sizing of either device independently.

---

### 1.4 — Exact W_p Extraction: Finding V_M = V_DD/2

**Motivation:**

V_M = V_DD/2 maximises and equalises both noise margins (NM_L and NM_H). From the PMOS sweep data, a parametric extraction of the exact W_p that centres V_M at 900mV is performed.

**Method:** Cross-function analysis — plot V_M vs W_p, extract the intersection with the V_DD/2 = 900mV horizontal.

**Result:**

```
W_p = 285.42n → V_M = 899.91mV ≈ 900mV ✓

Non-linear (concave) V_M vs W_p curve confirms
the √(k_n/k_p) dependence in the V_M formula
```

**Technology parameter extracted:**

$$\frac{W_p}{W_n} = \frac{285.42\text{n}}{120\text{n}} = 2.38$$

Since this ratio equals µ_n/µ_p at the sizing condition k_n = k_p:

$$\boxed{\frac{\mu_n}{\mu_p} = 2.38 \text{ in gpdk090 (90nm)}}$$

This is not a textbook value. It is a technology-specific parameter extracted experimentally — a meaningful result that characterises the process node directly.

---

### 1.5 — Baseline Transient: Observing the Delay Asymmetry

**Motivation:**

Before optimising delay, it is necessary to first observe and quantify the delay asymmetry at the V_M-balanced sizing (W_p = 285.42n). This establishes the pre-optimisation baseline for timing.

**Observation at W_p = 285.42n, W_n = 120n:**

```
Output fall (NMOS-governed) → faster
Output rise (PMOS-governed) → slower
→ t_pLH > t_pHL asymmetry confirmed

Even at VM-balanced sizing, dynamic delay
asymmetry persists because charging/discharging
C_L depends on peak currents, not just DC ratios.
```

This motivates the delay-specific parametric analysis in Stage 1.6.

---

### 1.6 — Propagation Delay Balancing: The Convergence

**Motivation:**

The delay asymmetry observed in Stage 1.5 prompts the question: what sizing balances t_pHL and t_pLH? And critically — is this the same sizing as V_M = V_DD/2, or a different optimisation?

**Analytical prediction:**

```
t_pHL ∝ C_L / k_n   (NMOS discharges output)
t_pLH ∝ C_L / k_p   (PMOS charges output)

For t_pHL = t_pLH:
C_L/k_n = C_L/k_p
→ k_n = k_p

This is IDENTICAL to the V_M = V_DD/2 condition.
Prediction: both optimisations converge at same W_p.
```

**Simulation result:**

```
t_p,rise (decreasing with W_p) crosses
t_p,delay (increasing with W_p) at:

W_p = 285.4629n → t_p = 8.87034ps

Compare with V_M balance point: W_p = 285.42n
Difference: 0.04n ≈ negligible
```

**The fundamental convergence:**

| Optimisation Target | W_p Required | Governing Condition |
|--------------------|-------------|-------------------|
| V_M = V_DD/2 | 285.42n | k_n = k_p |
| t_pHL = t_pLH | 285.46n | k_n = k_p |
| **Difference** | **0.04n ≈ 0** | **Same condition** |

> This is not a numerical coincidence. Both metrics are expressions of the same underlying physical balance: equal transconductance parameters. The telescopic approach predicts this analytically before simulation — the simulation merely confirms it.

---

### 1.7 — Noise Margin Extraction: Quantifying Immunity

**Method:** Voltage gain plot (dV_out/dV_in vs V_in) — markers at unity-gain points define V_IL and V_IH.

**Extracted values:**

| Parameter | Expression | Value |
|-----------|-----------|-------|
| NM_L | V_IL − V_OL | **701.589mV** |
| NM_H | V_OH − V_IH | **650.05mV** |
| Peak gain at V_M | \|dV_out/dV_in\|_max | **≈ 5.4** |
| Symmetry | NM_H / NM_L | **0.926 ≈ 1** |

Near-symmetric noise margins confirm that W_p = 285.42n successfully centres V_M. The small residual asymmetry is attributable to threshold voltage mismatch between PMOS and NMOS devices.

---

## Part 2 — Analytical V_M Derivation

### 2.1 — Boundary Conditions at V_M

At the switching threshold, by definition:

$$V_{in} = V_{out} = V_M$$

Both devices operate in saturation simultaneously, as both V_DS,n > V_GS,n − V_tn and V_SD,p > V_SG,p − |V_tp| are satisfied at this operating point.

---

### 2.2 — Current Equations in Saturation

**NMOS:**
$$I_{Dn} = \frac{k_n}{2}(V_M - V_{tn})^2, \quad k_n = \mu_n C_{ox} \frac{W_n}{L_n}$$

**PMOS:**
$$I_{Dp} = \frac{k_p}{2}(V_{DD} - V_M - |V_{tp}|)^2, \quad k_p = \mu_p C_{ox} \frac{W_p}{L_p}$$

---

### 2.3 — KCL Balance Condition

Continuity of current through the series-connected devices:

$$I_{Dp} = I_{Dn}$$

$$k_p(V_{DD} - V_M - |V_{tp}|)^2 = k_n(V_M - V_{tn})^2$$

---

### 2.4 — Algebraic Solution

Taking the positive square root of both sides:

$$\sqrt{k_p}(V_{DD} - V_M - |V_{tp}|) = \sqrt{k_n}(V_M - V_{tn})$$

Expanding and collecting V_M terms:

$$\sqrt{k_p}(V_{DD} - |V_{tp}|) + \sqrt{k_n} \cdot V_{tn} = V_M(\sqrt{k_n} + \sqrt{k_p})$$

$$\boxed{V_M = \frac{V_{DD} - |V_{tp}| + V_{tn}\sqrt{\dfrac{k_n}{k_p}}}{1 + \sqrt{\dfrac{k_n}{k_p}}}}$$

---

### 2.5 — Condition for V_M = V_DD/2

Substituting V_M = V_DD/2 and applying |V_tp| ≈ V_tn:

$$\sqrt{\frac{k_n}{k_p}} = 1 \implies k_n = k_p$$

Substituting the expressions for k:

$$\mu_n C_{ox} \frac{W_n}{L} = \mu_p C_{ox} \frac{W_p}{L}$$

$$\boxed{\frac{W_p}{W_n} = \frac{\mu_n}{\mu_p} = 2.38 \text{ (gpdk090, experimentally confirmed)}}$$

---

### 2.6 — Propagation Delay Under the Same Condition

$$t_{pHL} \propto \frac{C_L}{k_n}, \quad t_{pLH} \propto \frac{C_L}{k_p}$$

Setting t_pHL = t_pLH:

$$\frac{C_L}{k_n} = \frac{C_L}{k_p} \implies k_n = k_p$$

**The condition is identical.** Both V_M = V_DD/2 and t_pHL = t_pLH emerge from k_n = k_p — confirmed experimentally at W_p = 285.46n (Δ = 0.04n from 285.42n).

---

## Part 3 — Sizing Evolution Summary

```
ITERATION 0 — Initial Sizing
W_n = W_p = 120n, L = 100n
Motivation : Minimum symmetric — no prior assumptions
Result     : V_M = 900mV (baseline established)
Gap found  : Delay asymmetry t_pLH > t_pHL persists

ITERATION 1 — PMOS Sweep
W_p: 120n → 500n, W_n fixed at 120n
Motivation : Characterise V_M sensitivity to pull-up strength
Result     : V_M shifts LEFT monotonically with W_p
Insight    : k_p controls tug-of-war ratio, not R_on directly

ITERATION 2 — NMOS Sweep
W_n: swept parametrically, W_p fixed
Motivation : Validate symmetry of tug-of-war model
Result     : V_M also shifts LEFT with W_n
Insight    : V_M governed by k_n/k_p ratio exclusively

ITERATION 3 — Exact W_p Extraction
Method     : Cross-function V_M vs W_p → intersection at V_DD/2
Result     : W_p = 285.42n → V_M = 899.91mV ✓
Extracted  : µ_n/µ_p = 2.38 in gpdk090

ITERATION 4 — Delay Verification
Method     : t_p,rise and t_p,delay vs W_p crossover
Result     : Crossover at W_p = 285.46n (Δ = 0.04n)
Insight    : k_n = k_p governs both metrics simultaneously ✓

FINAL SIZING
W_n = 120n, W_p = 285.42n, L = 100n
V_M  = 899.91mV ≈ V_DD/2          ✓
t_p  = 8.87ps (balanced)           ✓
NM_L = 701.589mV, NM_H = 650.05mV ✓
```

---

## Part 4 — Key Conclusions

**C1 — Technology Parameter Extraction**  
µ_n/µ_p = 2.38 in gpdk090 (90nm) — experimentally derived from simulation, not assumed from literature.

**C2 — Causality in Transistor Sizing**  
Width increase causes transconductance increase. R_on reduction is a consequence. Confusing effect for cause leads to incorrect reasoning about sizing decisions.

**C3 — Metric Convergence**  
V_M centering and propagation delay balancing converge at the same sizing because both are constrained by k_n = k_p. This is a fundamental property of CMOS inverter physics.

**C4 — Non-linearity of V_M Response**  
V_M varies as √(k_n/k_p) with sizing — diminishing returns make linear interpolation for sizing estimates unreliable.

**C5 — Role as Foundation**  
The switching threshold concept established here — V_M as the balance point of transistor strengths — directly extends to bistable latch stability (Stage 2), SRAM cell read/write margins (Stage 3), and sense amplifier regeneration threshold (Stage 4). The inverter is the conceptual foundation of the entire design hierarchy.

---

## References

- Razavi, B. — *Design of Analog CMOS Integrated Circuits*, 2nd Ed.
- Weste, N. & Harris, D. — *CMOS VLSI Design*, 4th Ed.
- Seevinck, E. — *Analysis and Synthesis of Static CMOS Circuits*
- gpdk090 Process Design Kit Documentation

---

*CMOS Inverter — Stage 1 of Telescopic SRAM Design Project*  
*NIT Uttarakhand | B.Tech ECE | Analog IC Design | 2026*
