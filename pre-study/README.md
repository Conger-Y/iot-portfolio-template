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

---

## Datasheet Notes

### VL53L0X (ToF distance sensor)
- Physical quantity & principle:
- Supply voltage:
- Interface / I²C address:
- Range / resolution / accuracy / response time:
- Temperature dependence:
- Conditions / limitations warned about:

### MPR121 (capacitive touch sensor)
- Physical quantity & principle:
- Supply voltage:
- Interface / I²C address:
- Range / resolution / accuracy / response time:
- Temperature dependence:
- Conditions / limitations warned about:

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
