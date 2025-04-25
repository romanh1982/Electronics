
# 📘 Understanding Temperature-Induced Gain Drift in Differential Analog Front-Ends

## 🧩 Problem Statement

In a differential analog front-end, significant **gain drift** (~6dB) was observed with temperature, especially when using **clamping diodes** (e.g., BAT54SW-Q) in the signal path.

Despite BAT54SW-Q showing lower reverse leakage (I_R) than BAS16LTH in datasheets, **real-world measurements and simulations showed that BAS16LTH performs much better**, and **BAS116QA performs best**.

---

## 🔍 Key Observations

- **BAT54SW-Q (Schottky)**: Gain drift of ~6dB from -40°C to +100°C
- **BAS16LTH (PN Diode)**: Minor gain drift (~0.13dB)
- **BAS116 (Low-Leakage Diode)**: Almost no gain drift (<0.002dB)
- **SPICE Simulation** with `.TEMP` correctly shows this thermal effect
- **Manual stepping of diode capacitance (C_j)** had **much less effect**

---

## 🧠 Root Cause

- **Schottky diodes** (like BAT54SW-Q) have:
  - High and **nonlinear junction capacitance** (~10pF)
  - **Soft reverse conduction onset**
  - Low **reverse dynamic impedance** at high temperature
- While I_R may be lower on the datasheet, **Schottky behavior causes early signal shunting** at higher temps

- **PN diodes** (like BAS16LTH and BAS116):
  - Have **higher reverse breakdown impedance**
  - Better high-Z behavior even if datasheet I_R is higher
  - Much **more stable capacitance and blocking behavior**

---

## ✅ Lessons Learned

| Diode         | Capacitance | I_R (max)   | Thermal Gain Drift | Notes |
|---------------|-------------|-------------|---------------------|-------|
| **BAT54SW-Q** | ~10pF      | 2µA        | ~6dB               | Schottky, soft reverse |
| **BAS16LTH**  | 0.36–2pF   | 550µA      | ~0.13dB            | Good balance |
| **BAS116**    | ~1.5pF     | 3pA–5nA     | <0.002dB           | Best choice for stability |

---

## 🔧 Solutions

1. **Replace Schottky (BAT54) with PN diode (BAS16LTH or BAS116)**
2. Use `.TEMP` in SPICE to capture temperature drift:
   ```spice
   .TEMP -40 25 100
   ```
3. Consider adding **parallel resistors (~5–10kΩ)** to dominate the RC divider and reduce temperature sensitivity
4. Ensure **layout symmetry** for matched differential behavior

---

## 📘 Glossary

- **C_j**: Junction capacitance of a diode (forms a voltage divider with input R)
- **I_R**: Reverse leakage current
- **.TEMP**: SPICE directive to simulate temperature effects
- **Schottky Diode**: Metal–semiconductor junction; fast but with higher leakage and soft reverse behavior
- **PN Diode**: Traditional diode with better blocking behavior

---

## 🧪 Simulation Insights

- Only `.TEMP` properly triggers model-level temperature scaling (I_S, RS, C_J)
- Stepping C_j manually does not capture the nonlinear, temperature-dependent leakage effects
