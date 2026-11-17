# Module 4 – Liquid Level Measurement: Freshwater, Seawater & Diesel Tank Analysis

![Project Banner](assets/hardware_setup_overview.jpg)

## Overview
This repository documents the experimental evaluation, calibration, and comparative analysis of low-cost liquid level sensing technologies (**Resistive Needle Probe HD-38 / HW-038** vs. **Ultrasonic Distance Sensor HC-SR04 / JSN-SR04T**) across three different fluid mediums: **Freshwater**, **Seawater**, and **Diesel Fuel**. 

The goal is to assess sensor linearity, signal drift, repeatability, environmental interference (sloshing/rocking), and medium compatibility for maritime and industrial tank monitoring applications.

---

## 1. Freshwater Tank (Resistive Needle Probe – HD-38)

### 1.1 Hardware Setup & Calibration Model
- **Sensor**: Resistive needle probe (`HD-38` module)
  - Analog Output (`AO`) $\rightarrow$ `GPIO34`
  - Digital Output (`DO`) $\rightarrow$ `GPIO19`
- **Power Management**: Powered via `GPIO25` strictly during measurement intervals to minimize electrolysis and electrode corrosion.
- **Reference Baseline**: Needle tips touch the bottom of the container (`0%` level).
- **Calibration Models** ($\text{ADC} = A + \frac{B}{\text{level\_pct}}$):
  - **2-Point Model** (calibrated at `100%` and `50%`): $A_2 = 119.0$, $B_2 = 4400.0$
  - **4-Point Fit** (calibrated at `100%`, `75%`, `50%`, `25%`): $A_4 = 131.8$, $B_4 = 3345.0$

### 1.2 Experimental Observations & Results

| Experiment Phase | Key Metric / Observation | Quantitative Result |
| :--- | :--- | :--- |
| **1. Slow Pouring** | Non-linear ADC drop & Digital (`DO`) threshold trigger | Dry `~4095` $\rightarrow$ Wet `~150–250`; `DO` flips `0` $\rightarrow$ `1` cleanly upon contact |
| **2. Settling & Drift** | Continuous downward drift at constant `100%` water level | Starts at **`711`**, settles at **`238`** (**`67%` total drop**) |
| **3. Repeatability** | Full $\rightarrow$ Empty $\rightarrow$ Full cycle stability | 1st Full: **`161 ± 2.7`** vs. 2nd Full: **`163 ± 2.0`** (~1.2% variance) |
| **4. Sloshing / Agitation** | Surface wave disturbance at the end of pouring | Significant spread between `adc_min` and `adc_max` |

#### Visual Results – Freshwater Calibration & Drift
![Freshwater Calibration and Drift](assets/freshwater_calibration_drift.png)
*Figure 1: Non-linear calibration curve (2-point vs. 4-point fit) and 67% downward signal drift under constant 100% submersion due to electrode polarization.*

![Freshwater Repeatability](assets/freshwater_repeatability_cycle.png)
*Figure 2: Full $\rightarrow$ Empty $\rightarrow$ Full repeatability cycle showing clean dry baseline recovery (`ADC > 2800`, peaking at `4095`).*

---

## 2. Seawater & Diesel Tank Analysis

### 2.1 Seawater Conductivity Effects (Resistive Probe)
- **Higher Ionic Conductivity**: Dissolved salts drastically lower fluid resistance compared to freshwater, pushing the resistive probe rapidly toward saturation (`ADC` drops steeper even at minimal submersion).
- **Accelerated Electrolysis**: Even with pulsed power (`GPIO25`), saltwater accelerates oxidation on exposed copper/nickel traces, degrading long-term baseline accuracy.

### 2.2 Diesel Tank Experiment (`Module04_Diesel_Tank`)
Because diesel is a **non-conductive hydrocarbon**, resistive probes (`HW-038` / `HD-38`) fail to detect liquid presence (`ADC` remains pinned near the dry baseline `~4095`). Consequently, non-contact **Ultrasonic Sensing** (implemented on `ESP32` via `code/ultrasonic_reference_ESP32.ino`) was evaluated using two configurations:
1. **Direct Ultrasonic Reflection** (measuring distance directly to the diesel surface).
2. **Float-Assisted Ultrasonic Reflection** (measuring distance to a buoyant reflector floating on the diesel surface).

#### Diesel Experimental Results Summary
*(Data transcribed from `data/observed_screenshot_transcription.csv`)*

| Test Condition | Sensor / Setup | Observed Behavior / Reading | Notes |
| :--- | :--- | :--- | :--- |
| **Resistive Baseline** | `HW-038` Needle Probe | `ADC ≈ 4095` (No response) | Diesel is electrically insulating; resistive sensing unusable |
| **Direct Diesel Surface** | Ultrasonic (`ESP32`) | `[Insert mm/cm reading]` | Low acoustic impedance & surface absorption can cause occasional echo loss |
| **Float-Assisted Target** | Ultrasonic + Float | `[Insert mm/cm reading]` | Solid float provides a high-reflectivity target for stable distance measurement |
| **Simulated Rocking** | `simulate_rocking.py` | Periodic distance oscillation | Wave tilt deflects direct echoes; float stabilizes acoustic return |

#### Visual Results – Diesel Tank & Rocking Simulation
![HW038 Diesel Serial Log](assets/HW038_probe_diesel_log.png)
*Figure 3: Serial output confirming that the resistive `HW-038` probe remains unresponsive (`~4095`) when submerged in non-conductive diesel.*

![Ultrasonic Direct vs Float](assets/ultrasonic_direct_vs_float.png)
*Figure 4: Comparison of ultrasonic distance readings directly off the diesel surface vs. using a floating target across two filling cycles.*

![Simulated Rocking Slosh](assets/simulated_rocking_curve.png)
*Figure 5: Simulated tank rocking (`data/simulated_shaking_illustrative_only.csv`) illustrating measurement noise and echo variance under dynamic motion.*

---

## 3. Comparative Sensor Selection Matrix

| Criterion | Resistive Probe (`HD-38` / `HW-038`) | Ultrasonic Sensor (`ESP32` Reference) |
| :--- | :--- | :--- |
| **Freshwater** | Works (non-linear; requires calibration & settling time) | **Recommended** (linear, no drift, non-contact) |
| **Seawater** | Poor long-term reliability (rapid saturation & corrosion) | **Recommended** (immune to salinity/conductivity) |
| **Diesel / Fuel** | **Incompatible** (non-conductive fluid) | **Recommended** (especially with a float reflector) |
| **Signal Drift** | High (`67%` drift observed in constant freshwater test) | Negligible (temperature-dependent only) |
| **Sloshing / Rocking** | High sensitivity (`adc_min`/`adc_max` spikes) | Moderate (mitigated via moving average & stilling well) |

---

## 4. Industrial & Maritime Application Context
In real-world **maritime vessel fuel/ballast monitoring** and **industrial fluid storage** (such as bulk diesel backup tanks and process water reservoirs in **North Rhine-Westphalia, Germany**):
- **Resistive probes** are strictly limited to simple binary high/low leak detection in conductive water systems.
- **Non-contact ultrasonic or guided-wave radar sensors** housed inside a **perforated stilling pipe (baffle tube)** are the industry standard—eliminating fluid conductivity dependencies, preventing electrode corrosion, and mechanically damping sloshing waves during vessel motion or pump refilling.

---

## Repository Structure
```text
├── README.md
├── assets/                                   # Experiment plots & hardware photos
├── code/
│   └── ultrasonic_reference_ESP32.ino        # ESP32 ultrasonic distance measurement firmware
├── data/
│   ├── observed_screenshot_transcription.csv # Transcribed diesel test measurements
│   ├── simulate_rocking.py                   # Tank sloshing/rocking simulation script
│   └── simulated_shaking_illustrative_only.csv
└── notebooks/
    └── Module_4_Freshwater_Analysis.ipynb    # Freshwater calibration & drift analysis
```