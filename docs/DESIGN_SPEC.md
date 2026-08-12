# Final V2 Electrical Design Specification

This document is the frozen electrical baseline for the KiCad redesign. PCB layout must conform to this specification unless a later commit explicitly changes it.

## 1. System purpose

Weak optical signal measurement front-end for an STM32G431-based instrument. The optical source is expected to be modulated at 1 kHz; the MCU samples at 20 kS/s and performs digital phase-sensitive/FFT processing.

## 2. External interface

### J1 — MCU / supply interface

| Pin | Net | Function |
|---|---|---|
| 1 | +5V_IN | Raw +5 V input |
| 2 | AGND | Analog ground |
| 3 | ADC_OUT | Analog output to STM32 ADC |

### J2 — calibration input

| Pin | Net | Function |
|---|---|---|
| 1 | CAL_IN | External calibration voltage |
| 2 | AGND | Calibration return |

## 3. Power chain

- `+5V_IN -> FB1 -> +5V_FILTER`
- `FB1 = BLM18AG601SN1D`, 600 Ω @ 100 MHz
- `U1 = LP5907MFX-3.3/NOPB`
- U1 pin mapping:
  - Pin 1 IN -> +5V_FILTER
  - Pin 2 GND -> AGND
  - Pin 3 EN -> +5V_FILTER
  - Pin 4 NC
  - Pin 5 OUT -> AVDD_3V3

Decoupling:

- C1 = 4.7 µF, +5V_IN to AGND
- C2 = 100 nF, +5V_IN to AGND
- C3 = 4.7 µF, +5V_FILTER to AGND
- C4 = 100 nF, +5V_FILTER to AGND
- C5 = 4.7 µF, AVDD_3V3 to AGND
- C6 = 100 nF, AVDD_3V3 to AGND

## 4. Reference and bias

- `U2 = REF3312AIDBZR`
- U2 pin mapping:
  - Pin 1 IN -> AVDD_3V3
  - Pin 2 OUT -> VREF_RAW
  - Pin 3 GND -> AGND
- C7 = 100 nF, AVDD_3V3 to AGND
- C8 = 1 µF, VREF_RAW to AGND
- R1 = 10 kΩ, 0.1%, `VREF_RAW -> VBIAS_1V25`
- C9 = 1 µF, `VBIAS_1V25 -> AGND`

Nominal bias target: approximately 1.25 V.

## 5. Photodiode and TIA

### D1 — BPW34

- Pin 1 / Anode -> AGND
- Pin 2 / Cathode -> SUM

### U3 — OPA381AIDGKR

Pin mapping:

| Pin | Function | Net |
|---|---|---|
| 1 | NC | NC |
| 2 | -IN | SUM |
| 3 | +IN | VBIAS_1V25 |
| 4 | V- | AGND |
| 5 | NC | NC |
| 6 | OUT | TIA_OUT |
| 7 | V+ | AVDD_3V3 |
| 8 | NC | NC |

Local decoupling:

- C10 = 100 nF, AVDD_3V3 to AGND, placed immediately adjacent to U3 supply pins
- C11 = 1 µF, AVDD_3V3 to AGND, local bulk decoupling

Feedback:

- RF1 = 1 MΩ, 0.1%, SUM to TIA_OUT
- CF1 = 15 pF, C0G, SUM to TIA_OUT
- Exactly one feedback branch; no gain-switch network on the Final V2 board

## 6. ADC output filter

- R2 = 1 kΩ, 0.1%, `TIA_OUT -> ADC_OUT`
- C12 = 33 nF, `ADC_OUT -> AGND`
- Target RC corner is approximately 4.8 kHz

## 7. Calibration network

- TP_CAL -> CAL_IN
- RCAL = 100 MΩ, 1%, `CAL_IN -> CAL_R_NODE`
- SJCAL normally open:
  - Pad 1 -> CAL_R_NODE
  - Pad 2 -> SUM
- When SJCAL is closed:
  - `I_CAL = (V_CAL - VBIAS) / RCAL`
- CAL_IN and RCAL must remain electrically isolated from SUM while SJCAL is open

## 8. Test points

| Ref | Net | Nominal pad size |
|---|---|---|
| TP1 | +5V_IN | 1.0 mm |
| TP2 | +5V_FILTER | 1.0 mm |
| TP3 | AVDD_3V3 | 1.0 mm |
| TP4 | VREF_RAW | 1.0 mm |
| TP5 | VBIAS_1V25 | 1.0 mm |
| TP6 | SUM | 0.8 mm |
| TP7 | TIA_OUT | 1.2 mm |
| TP8 | ADC_OUT | 1.2 mm |
| TP9 | AGND | 1.2 mm |
| TP_CAL | CAL_IN | 1.0 mm |

TP6 silkscreen: `SUM_HIZ`.

## 9. PCB constraints

- 2 layers
- 45 mm x 30 mm nominal board size
- 1.6 mm FR-4
- 1 oz copper
- SUM is the highest-priority sensitive node
- D1, U3, RF1 and CF1 form one compact analog cluster
- Minimize SUM copper area and parasitic capacitance
- Keep ADC_OUT, CAL_IN and long AGND current paths away from SUM
- RF1 and CF1 must be physically adjacent to U3
- C10 must form the smallest practical U3 supply-decoupling loop
- Bottom layer should preferably provide a continuous AGND reference plane except where intentionally excluded around the SUM high-impedance region
- VBIAS guard geometry may be used around SUM if it can be implemented without increasing SUM capacitance or compromising routing

## 10. Production intent

Target fabrication/assembly route: JLCPCB / JLCEDA-compatible manufacturing output derived from KiCad Gerber, drill, BOM and pick-and-place files.

## 11. Reference-project policy

The TU Darmstadt `PDH_photodiode` project is used only as an engineering/layout reference. Its wideband ±5 V circuit, different photodiode, amplifier and gain are not copied electrically. Final V2 remains a 3.3 V single-supply OPA381/BPW34 design described by this specification.
