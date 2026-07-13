# CMOS Inverter Characterisation — gpdk090

**An engineering design notebook. Stage 1 of a 4-stage telescopic design project ending at a 6T SRAM cell with sense amplifier analysis.**

![Tool](https://img.shields.io/badge/Tool-Cadence%20Virtuoso-red)
![Tech](https://img.shields.io/badge/Technology-gpdk090%2090nm-blue)
![Sim](https://img.shields.io/badge/Simulator-Spectre-green)
![VDD](https://img.shields.io/badge/VDD-1.8V-orange)
![Stage](https://img.shields.io/badge/Stage-1%20of%204-lightgrey)

> A circuit designer who cannot predict a simulation result before running it does not yet understand the circuit.

This repository is not a collection of simulation screenshots. It is a chronological record of how my understanding of the CMOS inverter evolved — including the predictions that failed, the causal reasoning I had to correct, and the point at which the simulations stopped surprising me.

---

## Why This Project Exists

I wanted to reach a specific state before touching anything more complex than an inverter: the state where I could write down what a simulation would show *before* running it, and be right for the right reasons.

The inverter is the smallest circuit where that discipline can be practised honestly. It contains the essential physics of everything that follows:

```
Inverter switching threshold (VM)
        │  cross-couple two inverters
        ▼
Bistable latch — two stable states — a memory element
        │  add access transistors
        ▼
6T SRAM cell — controllable read and write
        │  add regenerative amplification
        ▼
Sense amplifier — rail-to-rail output from a mV differential
```

If I cannot explain why the inverter switches where it does, I cannot explain read disturb in SRAM or regeneration in a sense amplifier. So the inverter came first, and it was not allowed to be quick.

---

## Project Objectives

1. Establish a defensible baseline with no prior assumptions about the technology.
2. Understand — causally, not just numerically — how each transistor's width moves the switching threshold VM.
3. Find the exact sizing that centres VM at VDD/2, and extract the technology's real µn/µp from that sizing.
4. Determine whether the sizing that balances VM also balances propagation delay, and understand why or why not.
5. Quantify noise immunity at the final sizing.
6. Leave this stage only when every one of these behaviours is predictable before simulation.

---

## Engineering Workflow

Every experiment in this notebook follows the same loop:

```
Question
   ↓
Current mental model
   ↓
Prediction (written down before simulation)
   ↓
Intermediate verification (sanity checks, hand analysis)
   ↓
Simulation
   ↓
Contradiction, if any
   ↓
Missing physics identified
   ↓
Updated mental model
   ↓
Engineering insight
   ↓
Next question
```

The contradictions are kept in the record deliberately. They are where the learning happened.

---

## Technology

| Parameter | Value |
|-----------|-------|
| Process | gpdk090 (90 nm CMOS, generic PDK) |
| Tool | Cadence Virtuoso |
| Simulator | Spectre |
| VDD | 1.8 V |
| Minimum L | 100 nm |
| Minimum W | 120 nm |
| Devices | `nmos1v`, `pmos1v` |
| Input stimulus | Pulse 0 → 1.8 V, tr = 1 ns (transient); DC sweep 0 → 1.8 V (VTC) |

One number in this table is missing on purpose: µn/µp. Textbooks give values between 2 and 3. I did not want a textbook value — extracting the real ratio for this technology became one of the objectives, and it appears in Chapter 4.

---

## Design Evolution

### Chapter 1 — Finding a Neutral Starting Point

**Question.** If I know nothing about this technology's mobility ratio, what is the most defensible starting point?

**Mental model at this stage.** The inverter is a resistive divider whose two "resistances" are set by transistor geometry. Since µn > µp in every CMOS process I had read about, equal geometry should mean unequal strength.

**Prediction.** With Wn = Wp = 120n and L = 100n, kn > kp, the NMOS pull-down dominates, and VM lands *below* VDD/2 — somewhere left of 900 mV.

The honest move was to impose no sizing assumption at all. Equal minimum geometry means every asymmetry that appears later is the technology speaking, not my prior.

*Why this experiment?* To establish the reference against which every later deviation is measured.

![Inverter schematic — Wn = Wp = 120n, L = 100n](inverter_schematic.png)

*Fig 1 — Schematic. PM0: pmos1v W=120n L=100n, NM0: nmos1v W=120n L=100n, VDD = 1.8 V.*

Before trusting any VTC number I verified the setup dynamically: a transient run had to show clean rail-to-rail complementary switching, otherwise the DC sweep would be characterising a broken testbench.

![Baseline VTC with marker at VM](vtc_symmetric.png)

*Fig 2 — Transient (left): clean 0 ↔ 1.8 V switching. VTC (right): marker M1 reads 900.0 mV → 900.0 mV.*

**Observation.** VM = 900 mV. Exactly VDD/2. My prediction was wrong.

**Missing physics.** I had assumed the mobility difference would dominate at minimum dimensions. At 90 nm it does not act alone: short-channel effects and the PDK's threshold-voltage calibration partially compensate the mobility imbalance at minimum sizing. Equal minimum geometry in gpdk090 is accidentally symmetric.

**Updated mental model.** The mobility asymmetry is real but latent at minimum size. It should reappear once the widths move away from the minimum — which is exactly what the next two chapters test.

#### Engineering Notebook — Chapter 1

- **Question I can answer now:** what VM is at equal minimum sizing in gpdk090, and why the naive µn > µp argument fails there.
- **Mistake I corrected:** assuming long-channel intuition transfers unmodified to a 90 nm PDK.
- **Mental model I built:** VM as the equilibrium of a tug-of-war between pull-up and pull-down strength.
- **General principle:** a wrong prediction with a clean testbench is data, not failure. Write the prediction down first or this data is lost.
- **Connection forward:** this 900 mV baseline is the reference for every sweep that follows.

---

### Chapter 2 — Understanding Pull-Up Strength

**Question.** How sensitive is VM to PMOS width, and in which direction does it move?

**Mental model.** In the transition region both devices conduct; the output sits where their currents balance. In divider terms: Vout ≈ VDD · Ron,n / (Ron,p + Ron,n).

**Prediction.** Widening the PMOS strengthens the pull-up, lifts Vout for every input, and drags the VTC crossing point — VM — to the *left*. (Left, not right: with a stronger pull-up, less input voltage is needed before the NMOS starts losing the fight at the crossing condition Vout = Vin.)

*Why this experiment?* One variable at a time. Wn stays fixed at 120n; Wp sweeps 120n → 500n in 10 steps.

![PMOS parametric sweep](pmos_sweep.png)

*Fig 3 — VTC family for Wp = 120n → 500n, Wn fixed. The curves translate monotonically; the crossing with the Vin = Vout diagonal moves left.*

**Observation.** The VTC body shifts up and left, monotonically. VM decreases with Wp. The direction matched the prediction — but when I tried to articulate *why* to myself, the argument came out wrong.

**Contradiction — in the reasoning, not the plot.** My first verbal explanation was: "Ron of the PMOS decreases, therefore its current increases." The simulation could not falsify this, because the numbers come out the same. But the causality is inverted.

**Missing physics — the causal order.**

```
Wp↑  →  more parallel conduction channels
     →  kp↑              (intrinsic transconductance parameter)
     →  Id↑ at same VSG  (more current capability)
     →  Ron,p↓           ← CONSEQUENCE, not cause
     →  Vout↑ for same Vin
     →  VTC shifts up and left
     →  VM↓
```

Width is the physical cause. Ron is only how we summarise the effect — like adding resistors in parallel: the total drops as a *consequence* of the added path, not the other way round. The distinction sounds pedantic until a sizing change behaves unusually and the inverted chain predicts the wrong direction.

**Observation on the spacing.** The VM shift per step shrinks as Wp grows. Not a plotting artifact — VM depends on √(kn/kp), so doubling Wp does not double the effect. Linear interpolation for sizing estimates is unreliable here. This mattered in Chapter 4.

#### Engineering Notebook — Chapter 2

- **Question I can answer now:** the direction, mechanism, and saturation behaviour of VM versus Wp.
- **Mistake I corrected:** stating the causal chain backwards ("Ron↓ therefore Id↑").
- **Mental model I built:** W → k → Id → Ron, strictly in that order.
- **General principle:** a correct numerical answer can hide an inverted causal model. Simulation checks numbers, not reasoning.
- **Connection forward:** if the pull-up sweep is understood, the pull-down sweep becomes a prediction test.

---

### Chapter 3 — Understanding Pull-Down Strength

**Question.** Does widening the NMOS move VM the same way as widening the PMOS, or the opposite way?

**Prediction — written before the run, kept verbatim.** My reasoning at the time: Wn↑ → Id↑, and since Vgs is fixed, Vds must increase, so Vout rises and the curve shifts up-left. I predicted VM shifts left with Vout moving *upward*.

![NMOS parametric sweep](nmos_sweep.png)

*Fig 4 — VTC family for increasing Wn. The curves shift left, as with the PMOS sweep — but the body moves DOWN, not up.*

**Observation.** VM shifts left — direction correct. But the VTC body moved *down*. Half my prediction was right, and the half that was wrong was the half carrying the physics.

**Missing physics.** A stronger NMOS pulls the output *down* harder. My "Vds increases" step was not physics at all — it was the answer I expected, dressed up as a derivation.

```
Wn↑  →  kn↑  →  Id↑  →  Ron,n↓        ← consequence, as before
     →  Vout = VDD · Ron,n / (Ron,p + Ron,n)
        Ron,n↓  →  Vout↓ for same Vin  ← OPPOSITE to the PMOS case
     →  VTC shifts down and left
     →  VM↓
```

**Updated mental model — the synthesis.** Both sweeps push VM left, but through opposite Vout trajectories:

| | PMOS W↑ | NMOS W↑ |
|---|---|---|
| Network strengthened | Pull-up | Pull-down |
| Vout for same Vin | Increases | Decreases |
| VTC body movement | Up + left | Down + left |
| VM | Decreases | Decreases |

VM is governed by the *ratio* of the two strengths, never by either device alone. The tug-of-war model survived its first real test.

#### Engineering Notebook — Chapter 3

- **Question I can answer now:** how each device's width moves the VTC, including the Vout trajectory, not just the VM direction.
- **Mistake I corrected:** predicting Vout rises when the pull-down strengthens.
- **Mental model I built:** VM as a pure ratio property; absolute sizes only matter through their ratio.
- **General principle:** getting the final direction right for the wrong reason is more dangerous than being wrong — check the intermediate quantities, not just the endpoint.
- **Connection forward:** if VM is a ratio property, there exists one exact ratio that centres it. Finding it is the next question.

---

### Chapter 4 — Searching for the Switching Threshold

**Question.** What exact Wp puts VM at precisely VDD/2 — and what does that number reveal about the technology?

**Mental model.** From the analytical derivation (worked in full in [`vm_derivation.md`](vm_derivation.md)): VM = VDD/2 requires kn = kp. Since k = µ·Cox·(W/L) and both devices share L:

```
kn = kp   →   Wp/Wn = µn/µp
```

So the balancing width is not just a design number. It is a *measurement of the process itself*.

**Prediction.** The VM-vs-Wp curve will be concave (the √(kn/kp) compression from Chapter 2), and its crossing with 900 mV will land at a Wp/Wn ratio in the 2–3 range that textbooks quote for µn/µp.

![VM versus Wp cross-function](vm_vs_wp.png)

*Fig 5 — VM as a function of Wp. Marker M2: Wp = 285.42n → VM = 899.91 mV. The curve is visibly concave — diminishing returns, as predicted.*

**Observation.**

| Finding | Value |
|---|---|
| Wp for VM = 900 mV | **285.42n** |
| Confirmed VM | 899.91 mV |
| Wp/Wn | **2.38** |
| **µn/µp in gpdk090** | **2.38** |

$$\frac{W_p}{W_n} = \frac{285.42\,\text{n}}{120\,\text{n}} = 2.38 \implies \frac{\mu_n}{\mu_p} = 2.38$$

Not a datasheet value. Extracted from the switching behaviour of one inverter.

**A tension worth recording.** This result sat uncomfortably next to Chapter 1, where equal sizing also gave VM ≈ 900 mV. Both cannot follow from the same long-channel equation. The reconciliation is the one Chapter 1 hinted at: at minimum dimensions, second-order effects mask the mobility imbalance; once Wp moves away from minimum, the classical √(kn/kp) dependence takes over and governs the curve in Fig 5. The extraction is taken from the sweep region where the analytical model actually applies. Long-channel formulas at a 90 nm node are a regime, not a law.

#### Engineering Notebook — Chapter 4

- **Question I can answer now:** the exact sizing condition for a centred VM, and the real µn/µp of this PDK.
- **Mistake I corrected:** treating the analytical VM equation as valid across the whole sizing range rather than in its regime.
- **Mental model I built:** a well-chosen circuit measurement doubles as a process characterisation.
- **General principle:** when two of your own results appear to conflict, the boundary between them usually marks where a model's assumptions break.
- **Connection forward:** VM is a DC quantity. Whether this sizing also fixes the *dynamic* behaviour is a separate question — and I did not assume the answer.

---

### Chapter 5 — Understanding Dynamic Behaviour

**Question.** We have centred VM. Does the transient response balance at the same sizing?

Before answering, I had to close a gap in my own vocabulary. Mid-way through this work I forced myself to re-state, from scratch: *what is VM, actually?* The answer I settled on: VM is the input voltage at which Vout = Vin — the point where the inverter is exactly halfway through switching, both devices in saturation, gain at its peak. That is precisely what marker M1 in Fig 2 was showing. It seems basic. It was worth doing — Chapter 7 and the entire SRAM stage rest on this definition.

**Mental model.** VM centering is a static condition — it balances the operating point. Propagation delay is dynamic — it depends on how fast each device charges or discharges the load capacitance. Related, because both trace back to kn and kp. But I could not yet prove they were the *same* condition, so the prediction stayed cautious: some asymmetry may remain.

![Transient at Wp = 285.42n](baseline_transient.png)

*Fig 6 — Transient at the VM-balanced sizing. The output fall (NMOS discharging) is visibly faster than the rise (PMOS charging): tpLH > tpHL.*

**Observation.** Even at VM-balanced sizing the edges are not symmetric in this run. At this point something became genuinely interesting: is the residual asymmetry telling me the delay-balance sizing differs from the VM sizing — or is it measurement and load detail on top of the same underlying balance point? Only a proper delay-vs-Wp sweep could separate the two.

#### Engineering Notebook — Chapter 5

- **Question I can answer now:** why VM centering does not automatically guarantee edge symmetry in a given transient measurement.
- **Mistake I corrected:** conflating a DC equilibrium with a dynamic one without proof.
- **Mental model I built:** rise governed by PMOS charging CL, fall governed by NMOS discharging CL — two separate races over the same capacitor.
- **General principle:** when a definition feels "too basic to revisit," revisit it. Cheap insurance.
- **Connection forward:** the delay sweep will either split the two optimisations apart or collapse them into one.

---

### Chapter 6 — Balancing Propagation Delay

**Question.** What Wp equalises tpHL and tpLH — and is it the same Wp that centred VM, or a different one?

**Prediction — the strongest of the project, derived before the run:**

```
tpHL ∝ CL / kn     (NMOS discharges the output node)
tpLH ∝ CL / kp     (PMOS charges the output node)

tpHL = tpLH   →   CL/kn = CL/kp   →   kn = kp
```

kn = kp is *identical* to the VM = VDD/2 condition from Chapter 4. So the prediction was specific: the tp_rise and tp_fall curves must cross at Wp ≈ 285n, the same value the VM extraction produced.

![Delay crossover sweep](delay_crossover.png)

*Fig 7 — tp_rise (falling with Wp) against tp_fall (rising with Wp). Marker M2 at the crossover: Wp = 285.4629n, tp = 8.87 ps.*

**Observation.**

| Optimisation | Wp required | Governing condition |
|---|---|---|
| VM = VDD/2 | 285.42n | kn = kp |
| tpHL = tpLH | 285.46n | kn = kp |
| **Difference** | **0.04n ≈ 0** | **same condition** |

The prediction held to within 0.04 nm — noise. And this time it held for the reason I expected, which had not been true earlier in the project.

**Engineering insight.** VM centering and delay balancing are not two optimisations. They are two observable consequences of one physical condition, kn = kp. The question "should I size for threshold or for speed?" is, in this technology, a false dichotomy — one sizing, two benefits, one root cause. This is the kind of statement a simulator alone never produces; it only falls out when the prediction is derived first and the simulation is used as the judge.

#### Engineering Notebook — Chapter 6

- **Question I can answer now:** why DC and dynamic balance converge, not merely that they do.
- **Mistake I corrected:** none in this chapter — which is itself the data point. The model had become predictive.
- **Mental model I built:** apparently distinct specifications can be projections of a single underlying parameter; find the parameter.
- **General principle:** the goal of characterisation is to make the next simulation boring.
- **Connection forward:** with sizing frozen at Wn = 120n, Wp = 285.42n, the last open quantity is noise immunity.

---

### Chapter 7 — Quantifying Noise Immunity

**Question.** At the final sizing, how much noise can this inverter reject before a logic level becomes ambiguous?

**Method and prediction.** VIL and VIH are defined by the unity-gain points of the VTC (|dVout/dVin| = 1). With kn = kp, I expected NML and NMH to come out nearly equal — near-perfect symmetry would be the final confirmation that the sizing is right; any residual asymmetry would have to be explained, not waved away.

![Gain plot and noise margin extraction](noise_margin.png)

*Fig 8 — dVout/dVin versus Vin. Unity-gain markers: VIL = 701.589 mV, VIH = 1.14995 V. Peak gain ≈ −5.4 at VM = 900 mV.*

**Observation.**

| Parameter | Expression | Value |
|---|---|---|
| NML | VIL − VOL | **701.6 mV** |
| NMH | VOH − VIH | **650.0 mV** |
| Symmetry | NMH / NML | **0.926** |
| Peak gain at VM | \|dVout/dVin\|max | **≈ 5.4** |

**Engineering insight.** The margins are nearly symmetric — 0.926 against an ideal 1.0 — confirming the sizing. The residual 51 mV asymmetry is not sizing error: kn = kp balances the *strengths*, but the threshold voltages |Vtp| and Vtn of this PDK are not perfectly matched, and no width choice can correct a Vt mismatch. Recognising which imperfections width can fix and which it cannot is itself a sizing skill.

#### Engineering Notebook — Chapter 7

- **Question I can answer now:** the quantitative noise immunity of the final design, and the physical origin of its residual asymmetry.
- **Mistake I corrected:** the early instinct that "one more sizing tweak" can fix any asymmetry.
- **Mental model I built:** W controls strength ratio; Vt mismatch is a separate, width-immune axis.
- **General principle:** know which knob controls which error term before turning knobs.
- **Connection forward:** these noise margins become the vocabulary for static noise margin (SNM) analysis of the SRAM cell in Stage 3.

---

## Final Engineering Insights

Final design: **Wn = 120n, Wp = 285.42n, L = 100n, VDD = 1.8 V.**

| Metric | Value |
|---|---|
| VM | 900 mV = VDD/2 |
| Balanced tp | 8.87 ps |
| NML / NMH | 701.6 mV / 650.0 mV |
| Peak gain at VM | ≈ 5.4 |
| µn/µp (extracted) | 2.38 |

What actually changed between the first chapter and the last:

1. **Causality became non-negotiable.** W → k → Id → Ron, in that order, always. Both of my early reasoning failures were causality inversions, and both produced correct-looking numbers.
2. **VM stopped being a point on a curve and became a competition.** Every later structure — latch metastability, SRAM read disturb, sense-amp regeneration — is the same tug-of-war wearing different clothes.
3. **One condition, many faces.** kn = kp centres VM *and* balances delay *and* symmetrises noise margins. Specifications that look independent often are not.
4. **Models have regimes.** The long-channel VM equation described the sweep region and failed at minimum dimensions. Knowing where a model stops working proved more useful than the model itself.
5. **By Chapter 6, the simulations had become confirmations.** That transition — not any single number — was the actual deliverable of this stage.

---

## Repository Structure

```
.
├── README.md                  ← this notebook
├── vm_derivation.md           ← full analytical derivation of VM from first principles
├── inverter_schematic.png     ← Ch.1  schematic, Wn = Wp = 120n
├── vtc_symmetric.png          ← Ch.1  baseline VTC, VM = 900 mV marker
├── pmos_sweep.png             ← Ch.2  VTC family vs Wp
├── nmos_sweep.png             ← Ch.3  VTC family vs Wn
├── vm_vs_wp.png               ← Ch.4  VM(Wp) cross-function, 285.42n extraction
├── baseline_transient.png     ← Ch.5  edge asymmetry at VM-balanced sizing
├── delay_crossover.png        ← Ch.6  tp_rise / tp_fall crossover, 8.87 ps
├── noise_margin.png           ← Ch.7  gain plot, VIL / VIH markers
└── LICENSE
```

---

## References

- B. Razavi, *Design of Analog CMOS Integrated Circuits*, 2nd ed., McGraw-Hill.
- N. Weste and D. Harris, *CMOS VLSI Design: A Circuits and Systems Perspective*, 4th ed., Addison-Wesley.
- J. Rabaey, A. Chandrakasan, B. Nikolić, *Digital Integrated Circuits: A Design Perspective*, 2nd ed.
- Cadence gpdk090 Process Design Kit documentation.

---

## Future Work

| Stage | Topic | What it inherits from this stage | Status |
|---|---|---|---|
| **1 — Inverter** | **VM, delay, noise margins** | **Foundation** | **This repository** |
| 2 — Bistable latch | Cross-coupled feedback | VM becomes the metastability point | Planned |
| 3 — 6T SRAM cell | Write / read / hold SNM | VM becomes the read-disturb threshold | Planned |
| 4 — Sense amplifier | Comparative SA analysis | VM becomes the regeneration trigger | Planned |

The switching threshold is not an isolated inverter concept. The same tug-of-war between pull-up and pull-down decides stability in every stage above. The open question carried into Stage 2: when two of these inverters are cross-coupled, the point I have been calling VM stops being a threshold and becomes the boundary between two memories. What happens exactly *at* that boundary is where the next notebook begins.

---

*NIT Uttarakhand · B.Tech ECE · Analog IC Design · Cadence Virtuoso / Spectre · gpdk090 · April 2026*
