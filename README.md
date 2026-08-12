# BPW34 + OPA381 Weak-Signal Photodetector Front-End

KiCad-based, version-controlled redesign of a weak optical signal front-end for physics competition work.

## Final V2 design target

- Photodiode: BPW34
- TIA: OPA381
- Supply input: +5 V
- Analog rail: LP5907-3.3 -> AVDD_3V3
- Reference: REF3312 -> VREF_RAW -> 10 kΩ -> VBIAS_1V25, with local 1 µF bypass
- TIA feedback: 1 MΩ || 15 pF
- ADC isolation/filter: 1 kΩ series + 33 nF to AGND
- Calibration injection: CAL_IN -> 100 MΩ -> normally-open SJCAL -> SUM
- MCU interface: +5V_IN / AGND / ADC_OUT to existing STM32G431 board
- Target modulation/detection: 1 kHz optical modulation, 20 kS/s ADC, digital lock-in / FFT in MCU

## Development policy

This repository replaces GUI-only iterative PCB editing. Every electrical/layout milestone must be committed so that schematic, PCB, and production data can be diffed and rolled back.

Planned milestones:

1. `v0.1-schematic` — clean KiCad schematic, ERC pass, footprint mapping reviewed
2. `v0.2-placement` — critical analog placement frozen
3. `v0.3-critical-routing` — SUM, feedback, VBIAS, OPA381 local decoupling completed
4. `v0.4-full-routing` — remaining power/ADC/CAL routing and ground strategy completed
5. `v1.0-production` — DRC pass, Gerber/BOM/PnP ready for fabrication

## Reference design

Layout principles will be compared against the open hardware project `TU-Darmstadt-APQ/PDH_photodiode`, especially its photodiode/TIA placement, feedback-loop minimization, local decoupling, and ground strategy. The final circuit here is not intended to be an electrical clone; it is redesigned around BPW34, OPA381, 3.3 V single-supply operation, 1.25 V bias, STM32 ADC interfacing, and the calibration network used in this project.

## Status

Repository initialized. Next step: freeze the V2 electrical specification and create the KiCad project skeleton before any PCB routing.
