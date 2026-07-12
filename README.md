# CMOS Inverter — Telescopic Design Methodology

![Tool](https://img.shields.io/badge/Tool-Cadence%20Virtuoso-red)
![Tech](https://img.shields.io/badge/Technology-gpdk090%2090nm-blue)
![Sim](https://img.shields.io/badge/Simulator-Spectre-green)
![VDD](https://img.shields.io/badge/VDD-1.8V-orange)

---

## Design Philosophy

Most textbooks present CMOS inverter results directly.
This project takes the opposite approach — every
simulation result is **predicted from physical intuition
first**, then verified in Cadence Virtuoso.

Core question driving every step:
> "WHY does this happen physically —
> not just WHAT the formula says"

---

## Technology

| Parameter | Value |
|-----------|-------|
| Technology | gpdk090 (90nm CMOS) |
| Tool | Cadence Virtuoso + Spectre |
| VDD | 1.8V |
| Minimum L | 100nm |
| µn/µp ratio | 2.38 (extracted from simulation) |

---

## Design Flow

```
Step 1 → Symmetric sizing (Wn=Wp=120n)
         → VM = 900mV = VDD/2 confirmed ✓
         ↓
Step 2 → PMOS width sweep (120n to 500n)
         → VM shifts LEFT as Wp increases
         → Physical reason: Wp↑ → kp↑ → Ron,p↓ → Vout↑
         ↓
Step 3 → NMOS width sweep
         → VM shifts LEFT as Wn increases
         → Physical reason: Wn↑ → kn↑ → Ron,n↓ → Vout↓
         ↓
Step 4 → Extract exact Wp for VM = VDD/2
         → Wp = 285.42n confirmed
         → µn/µp = 2.38 extracted
         ↓
Step 5 → Propagation delay balancing
         → tp_rise = tp_fall at Wp = 285.46n
         → Same condition as VM balance!
         ↓
Step 6 → Noise margin extraction
         → NML = 701.589mV
         → NMH = 650.05mV
```

---

## Key Results

| Parameter | Value | Status |
|-----------|-------|--------|
| VM at symmetric sizing | 900mV = VDD/2 | ✅ |
| Wp for VM = VDD/2 | 285.42n | ✅ |
| µn/µp ratio extracted | 2.38 | ✅ |
| Delay balanced at Wp | 285.46n | ✅ |
| tp at balance point | 8.87ps | ✅ |
| NML | 701.589mV | ✅ |
| NMH | 650.05mV | ✅ |

---

## Core Insight

> VM centering and propagation delay balancing are
> governed by the **same condition: kn = kp**
>
> They are not two separate optimizations.
> They are two manifestations of one balance point.
>
> **One sizing — two benefits — one root cause.**
> Wp = 285.42n satisfies both simultaneously in gpdk090.

---

## Simulation Results

### 1. Inverter Schematic
![Schematic](simulations/results/inverter_schematic.png)

### 2. VTC — Symmetric Sizing (VM = 900mV)
![VTC](simulations/results/vtc_symmetric.png)

### 3. PMOS Parametric Sweep
![PMOS](simulations/results/pmos_sweep.png)

### 4. NMOS Parametric Sweep
![NMOS](simulations/results/nmos_sweep.png)

### 5. Wp Extraction for VM = VDD/2
![VM_WP](simulations/results/vm_vs_wp.png)

### 6. Propagation Delay Balancing
![Delay](simulations/results/delay_crossover.png)

### 7. Baseline Transient
![Transient](simulations/results/baseline_transient.png)

### 8. Noise Margins
![NM](simulations/results/noise_margin.png)

---

## Hand Calculation

Full derivation of VM from first principles:
[View VM Derivation](hand_calculations/vm_derivation.md)

### Summary of Derivation

At VM both PMOS and NMOS are in saturation.
Setting Ip = In:

```
kp(VDD - VM - |Vtp|)² = kn(VM - Vtn)²
```

Taking square root and solving:

```
VM = (VDD - |Vtp| + Vtn√(kn/kp)) / (1 + √(kn/kp))
```

For VM = VDD/2 → kn = kp → **Wp/Wn = µn/µp = 2.38**

![Notes](hand_calculations/vm_hand_calculation.jpg)

---

## Why Width is the CAUSE not Ron

Most students say:
> "Ron decreases therefore current increases"

**This is wrong — effect taken as cause.**

Correct chain:
```
Wp↑ → more parallel current channels
    → kp↑ (intrinsically stronger transistor)
    → Id↑ for same Vgs
    → Ron,p↓  ← THIS IS THE RESULT
    → Vout↑ for same Vin
    → VTC shifts left
    → VM decreases
```

Width increase is the CAUSE.
Ron decrease is the EFFECT.
Think of it like resistors in parallel.

---

## Part of Larger Telescopic Project

| Stage | Topic | Status |
|-------|-------|--------|
| **1 — Inverter** | **VM, Delay, Noise Margins** | **✅ This repo** |
| 2 — Bistable Latch | Cross-coupled feedback | 🔲 |
| 3 — 6T SRAM Cell | Write / Hold / Read SNM | 🔲 |
| 4 — Sense Amplifier | Comparative analysis | 🔲 |

Each stage builds physical intuition for the next.
The inverter VM concept directly becomes the
switching threshold concept in the SRAM cell.

---

## References

- Razavi B. — Design of Analog CMOS Integrated Circuits
- Weste & Harris — CMOS VLSI Design
- gpdk090 PDK Documentation

---

*NIT Uttarakhand | ECE | Analog IC Design | 2026*
