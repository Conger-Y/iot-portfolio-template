# Pre-Study — Sensors and Actuators (Sensorik und Aktorik)

**Name:** Conger-Y
**Course:** Sensors and Actuators, WS 2026/27, HSBI Gütersloh
**Instructor:** Prof. Dr. Ulrich Norbisrath (Ulno)

---

## Overview

This is my Module 0 (pre-study) portfolio entry. It contains my notes,
answers to Guiding Questions 1–8, the Wokwi mini-exercise evidence, and
my datasheet notes.

---

## Evidence Index

- Wokwi simulation screenshots: see below (Section: Wokwi)
- Datasheet notes (VL53L0X, MPR121): see below (Section: Datasheets)
- Project abstract: see Guiding Question 8

---

## Wokwi Mini-Exercise

*(Screenshots of the circuit and serial output go here. Notes on ADC
resolution changes and step size / "jumping" output below.)*

- Potentiometer → analog input, value printed via Serial, mapped to LED brightness.
- ADC resolution tested at: 8 / 10 / 12 bits
- Observations:
  - (fill in: integer range and step size at each resolution)
  - (fill in: what happened when the step size was set too coarse)
<img width="416" height="281" alt="image" src="https://github.com/user-attachments/assets/e0c3a2a7-2782-4c6c-b047-83603ed27a9d" />
<img width="415" height="220" alt="image" src="https://github.com/user-attachments/assets/f71ba0cd-80b7-40f9-904a-466fdd129e48" />
<img width="415" height="214" alt="image" src="https://github.com/user-attachments/assets/a4789363-0e2c-4d1f-8fdf-601eee458dd2" />
<img width="415" height="238" alt="image" src="https://github.com/user-attachments/assets/81a81d08-fe06-4962-832e-45910fa0f3d3" />


---

## Datasheet Notes

### VL53L0X (ToF distance sensor)
- Physical quantity & principle: Absolute distance. Time-of-Flight — a 940 nm VCSEL laser fires invisible pulses and the sensor times the photons returning from the target (SPAD array). Range is independent of target colour/reflectance.
- Supply voltage: 2.6–3.5 V (bare chip); breakout modules 2.6–5.5 V via onboard regulator.
- Interface / I²C address: I²C (+ XSHUT shutdown, GPIO1 interrupt). 7-bit address 0x29 (written 0x52 in 8-bit form). Programmable but resets to 0x29 on every power cycle.
- Range / resolution / accuracy / response time: ~30 mm to 2000 mm; accuracy ≈ ±3%; response < 30 ms (configurable timing budget trades speed for range/accuracy).
- Temperature dependence: operating −20 to +70 °C.
- Conditions / limitations: accuracy drops significantly beyond ~1.5 m; noisy on highly reflective surfaces; two identical units clash on the bus (same default 0x29) and must be re-addressed via XSHUT.

### MPR121 (capacitive touch sensor)
- Physical quantity & principle: Touch / proximity. Capacitive sensing — a finger near an electrode changes capacitance; the chip uses a constant-current charge method and measures the change in time constant.
- Supply voltage: 1.71–3.6 V (modules without regulator: 2.5–3.6 V). 3.3 V device — do not exceed 3.6 V.
- Interface / I²C address: I²C (+ IRQ interrupt). Default 0x5A; selectable to 0x5A/0x5B/0x5C/0x5D via the ADDR pin → up to 4 devices on one bus.
- Channels / response: 12 electrode inputs + 1 virtual proximity channel; configurable sample period (e.g. 16 ms) with touch/release threshold and debounce.
- Power: 29 µA at 16 ms sampling; 3 µA in stop mode.
- Conditions / limitations: no onboard regulator (watch the 3.6 V limit); electrode wire length/material affects triggering, thresholds must be tuned; 8 pins are multiplexed as LED/GPIO.

---

## Guiding Questions

### 1. The chain
*(Draw sensor → conditioning → ADC → MCU → system for a device you own. Name each stage.)*

### 2. Wokwi observation
*(How many discrete steps did the full analog range have? After changing ADC resolution, the new number? What changed — resolution, accuracy, or both?)*

### 3. Datasheet
*(For one chosen sensor: interface, address if any, stated accuracy + conditions attached.)*

### 4. Accuracy vs. precision
*(One everyday example: precise but not accurate. One: accurate but not precise.)*

### 5. I²C anticipation
*(Two sensors share clock + data wires — what problem does that create, and what is the address for?)*

### 6. Actuator choice
*("Move a small door latch once every few minutes" — which actuator, which not? Justify by torque, control, power.)*

### 7. LED power math
*(30-LED WS2812 strip, ~60 mA each at full white — total current? Why can't the MCU 5 V pin supply it?)*

### 8. Project abstract
*(5–10 line scenario: who uses it and why, which sensors, which actuators. Note the two sensors + one actuator you'd start with.)*
