# Module 4 – Liquid Level Measurement: Freshwater, Seawater, Diesel & Wastewater Tank Analysis
## 🎯 Learning Goals

Through multi-fluid experimental characterization, calibration modeling, and comparative sensor evaluation across Freshwater, Seawater, Diesel Fuel, and Wastewater tanks on the ESP32 and M5StickS3, this module achieves the following learning outcomes:

1. **Resistive Liquid Sensing, Pulsed Excitation & Electrode Drift (Freshwater & Seawater):**
   * Wire a resistive needle probe (`HD-38`) using pulsed GPIO power (`GPIO25`) to mitigate electrolysis and compare 2-point vs. 4-point inverse calibration models ($\text{ADC} = A + B/\text{level pct}$).
   * Quantify severe downward signal drift (`67%` drop under constant `100%` submersion) caused by electrode polarization, verify dry-baseline cycle repeatability (`~1.2%` variance), and explain why high ionic conductivity in seawater causes rapid saturation and accelerated trace corrosion.
2. **Non-Contact Ultrasonic Sensing & Sloshing Dynamics (Diesel Tank):**
   * Recognize why non-conductive hydrocarbons (diesel) require non-contact sensing and compare direct liquid-surface ultrasonic reflections against buoyant cork-float targets.
   * Evaluate dynamic tank rocking effects (`simulate_rocking.py`) and physical edge cases (unguided float tilting/sticking and near-field ultrasonic dead-zone saturation around `18–20 cm`).
3. **Optical Time-of-Flight (ToF) Scattering vs. Transparency Limits (`VL53L0X`):**
   * Compare `VL53L0X` laser ranging in clear water versus turbid simulated wastewater, demonstrating how infrared beam penetration and specular refraction cause `89%` invalid readings in clear water at full depth, whereas suspended particulates in turbid wastewater scatter the beam cleanly at the surface (`100%` valid readings, `7 mm` spread).
4. **Gravimetric Load Cell Sensing & Vessel Geometry Effects (`HX711`):**
   * Calibrate a strain-gauge load cell (`HX711`) to infer fluid mass and volume without chemical, acoustic, or optical contact, applying median filtering to reject mechanical placement outliers (`353.3 g`).
   * Account for non-cylindrical container geometry, verifying that half liquid depth in a tapered vessel corresponds to `40.3%` of full volume rather than `50%`.
5. **Engineering Trade-Offs & Maritime/Industrial Sensor Selection:**
   * Synthesize fluid properties (conductivity, turbidity, bio-fouling risk) and environmental dynamics (sloshing) into a structured selection matrix, justifying industrial best practices such as perforated stilling pipes (baffle tubes) for ultrasonic/radar tank gauges and external load cells for fouling-prone process vessels.
<img width="1280" height="1706" alt="aec9b1edacb80062622853c64adb15bc" src="https://github.com/user-attachments/assets/4b8b4100-4f5e-44a6-9d75-900e3cc77ad5" />

## Overview

This repository documents the experimental evaluation, calibration, and comparative analysis of low-cost liquid level sensing technologies (**Resistive Needle Probe HD-38 / HW-038**, **Ultrasonic Distance Sensor HC-SR04 / JSN-SR04T**, **Time-of-Flight Laser Sensor VL53L0X**, and **HX711 Load Cell**) across four different fluid mediums: **Freshwater**, **Seawater**, **Diesel Fuel**, and **Wastewater**.

The goal is to assess sensor linearity, signal drift, repeatability, environmental interference (sloshing/rocking), turbidity/optical scattering, and medium compatibility for maritime and industrial tank monitoring applications.

---

## 1. Freshwater Tank (Resistive Needle Probe – HD-38)

### 1.1 Hardware Setup & Calibration Model

* **Sensor**: Resistive needle probe (`HD-38` module)
  * Analog Output (`AO`) $\rightarrow$ `GPIO34`
  * Digital Output (`DO`) $\rightarrow$ `GPIO19`
* **Power Management**: Powered via `GPIO25` strictly during measurement intervals to minimize electrolysis and electrode corrosion.
* **Reference Baseline**: Needle tips touch the bottom of the container (`0%` level).
* **Calibration Models**:
  
  $$
  \text{ADC} = A + \frac{B}{\text{level pct}}
  $$

  * **2-Point Model** (calibrated at `100%` and `50%`): $A_2 = 119.0, B_2 = 4400.0$
  * **4-Point Fit** (calibrated at `100%`, `75%`, `50%`, `25%`): $A_4 = 131.8, B_4 = 3345.0$

### 1.2 Experimental Observations & Results

| Experiment Phase | Key Metric / Observation | Quantitative Result | 
| :--- | :--- | :--- | 
| **1. Slow Pouring** | Non-linear ADC drop & Digital (`DO`) threshold trigger | Dry `~4095` $\rightarrow$ Wet `~150–250`; `DO` flips `0` $\rightarrow$ `1` cleanly upon contact | 
| **2. Settling & Drift** | Continuous downward drift at constant `100%` water level | Starts at **`711`**, settles at **`238`** (**`67%` total drop**) | 
| **3. Repeatability** | Full $\rightarrow$ Empty $\rightarrow$ Full cycle stability | 1st Full: **`161 ± 2.7`** vs. 2nd Full: **`163 ± 2.0`** (~1.2% variance) | 
| **4. Sloshing / Agitation** | Surface wave disturbance at the end of pouring | Significant spread between `adc_min` and `adc_max` | 

#### Visual Results – Freshwater Calibration & Drift

<img width="782" height="507" alt="output" src="https://github.com/user-attachments/assets/98d33415-753e-41dc-bc03-387e070deb36" />

*Figure 1: Non-linear calibration curve (2-point vs. 4-point fit) and 67% downward signal drift under constant 100% submersion due to electrode polarization.*

<img width="849" height="468" alt="output1" src="https://github.com/user-attachments/assets/b1aba86d-cb49-454b-81a0-b58630701f09" />

*Figure 2: Full $\rightarrow$ Empty $\rightarrow$ Full repeatability cycle showing clean dry baseline recovery (`ADC > 2800`, peaking at `4095`).*

---

## 2. Seawater & Diesel Tank Analysis

### 2.1 Seawater Conductivity Effects (Resistive Probe)

* **Higher Ionic Conductivity**: Dissolved salts drastically lower fluid resistance compared to freshwater, pushing the resistive probe rapidly toward saturation (`ADC` drops steeper even at minimal submersion).
* **Accelerated Electrolysis**: Even with pulsed power (`GPIO25`), saltwater accelerates oxidation on exposed copper/nickel traces, degrading long-term baseline accuracy.

### 2.2 Diesel Tank Experiment (`Module04_Diesel_Tank`)

Because diesel is a **non-conductive hydrocarbon**, resistive probes (`HW-038` / `HD-38`) cannot conduct current through the fluid and are incompatible with fuel level sensing. Consequently, non-contact **Ultrasonic Sensing** (implemented on `ESP32` via `code/ultrasonic_reference_ESP32.ino`) was evaluated using two configurations:

1. **Direct Ultrasonic Reflection** (measuring distance directly to the diesel surface).
2. **Float-Assisted Ultrasonic Reflection** (measuring distance to a buoyant cork reflector floating on the diesel surface).

#### Diesel Experimental Results Summary

*(Data transcribed from `data/observed_screenshot_transcription.csv`)*

| Test Condition | Sensor / Setup | Observed Distance | Notes | 
| :--- | :--- | :--- | :--- | 
| **Direct Diesel Surface (Fill A)** | Ultrasonic (`ESP32`) | `7.22 cm` | Slight echo spread due to liquid surface absorption | 
| **Cork-Top Float (Fill A – Stable)** | Ultrasonic + Float | `4.56 cm` | Solid float provides a high-reflectivity target for stable measurement | 
| **Direct Diesel Surface (Fill B)** | Ultrasonic (`ESP32`) | `2.50 cm` | Higher liquid level reduces sensor-to-surface distance | 
| **Cork-Top Float (Fill B – Unstable)** | Ultrasonic + Float | `6.63 cm` (wide spread) | Unrestricted float tilted/stuck against tank wall after refilling | 
| **Simulated Rocking** | `simulate_rocking.py` | Periodic oscillation | Wave tilt deflects direct echoes; guided float dampens variance | 

#### Visual Results – Diesel Tank & Rocking Simulation

<img width="1506" height="818" alt="01_observed_distance_captures" src="https://github.com/user-attachments/assets/23ab540c-2a06-4d92-b5e4-411241e9e2c0" />

*Figure 3: Comparison of ultrasonic distance readings directly off the diesel surface vs. using a floating target across two filling cycles.*

<img width="1682" height="786" alt="03_simulated_rocking_time_series" src="https://github.com/user-attachments/assets/d51da01a-0dd8-4564-8b2a-93fdfa11d5f1" />

*Figure 4: Simulated tank rocking (`data/simulated_shaking_illustrative_only.csv`) illustrating measurement noise and echo variance under dynamic motion.*

---

## 3. Wastewater Holding Tank Analysis (Ultrasonic, ToF & Load Cell)

To address fouling, bio-film buildup, and suspended particulates inherent in blackwater/greywater holding tanks, three non-immersion sensing modalities were evaluated on simulated wastewater.

### 3.1 Ultrasonic Distance Sensor (`ESP32` – `sketch2sensors.ino`)
* **Hardware Setup**: Ultrasonic transducer wired to `TRIG_PIN = 25` and `ECHO_PIN = 26` on an `ESP32 Dev Module`, utilizing a 5-sample burst window (`distances[5]`) with threshold classification (`HIGH WATER` vs. `NORMAL`).

| Fill Condition | Observed Distance Range (cm) | Threshold / Stability Behavior |
| :--- | :--- | :--- |
| **Empty Cup** | `18.06 – 29.96 cm` (mostly `~28.59 cm`) | Wide acoustic beam reflects off tapered cup sidewalls when empty |
| **Half Depth (`50%`)** | `20.60 – 23.94 cm` (mostly `~21.88–22.19 cm`) | Stable surface return, clearly separated from empty baseline |
| **80% Depth (Static)** | `13.07 – 20.56 cm` (mostly `~18.01–19.60 cm`) | Triggers `HIGH WATER` alert; occasional boundary flips to `NORMAL` near `19.29–20.56 cm` |
| **80% Depth (Shaking)** | `16.14 – 18.06 cm` (mostly `~16.79–17.42 cm`) | Surface agitation causes slight distance scatter (`±1 cm`), remaining below alert threshold |
| **Full (`100%`)** | `18.35 – 22.19 cm` (mostly `~18.68–19.96 cm`) | Near-field minimum sensing distance limitation (`~18–20 cm` dead-zone saturation) |

### 3.2 Time-of-Flight (ToF) Optical Sensor (`VL53L0X` + `M5StickS3`)
* **Aim & Method**: Evaluated whether a `VL53L0X` ToF laser sensor (mounted `~10 cm` above the cup rim) can distinguish liquid depths in clear water vs. turbid simulated wastewater. Cardboard laid across the rim served as a solid-target control.

| Test Condition | Recorded Distances (Last 5 Numeric) | Summary & Validity |
| :--- | :--- | :--- |
| **Empty Cup** | `240, 234, 240, 239, 238 mm` | **Median `239 mm`**, span `6 mm` |
| **Cardboard at Cup Rim (Control)** | `106, 103, 104, 105, 102 mm` | **Median `104 mm`**, span `4 mm` (matches mounting height) |
| **Clear Water (~Half Depth)** | `237, 235, 242, 206, 226 mm` | **Median `235 mm`**, span `36 mm` (laser penetrates transparent water) |
| **Clear Water (Full)** | `173, 205 mm` | **17 of 19 outputs flagged invalid** (specular reflection / refraction loss) |
| **Simulated Wastewater (~Half Depth)** | `226, 229, 219, 226, 230 mm` | **Median `226 mm`**, span `11 mm` (`3 of 19` flagged invalid) |
| **Simulated Wastewater (Full)** | `135, 141, 138, 134, 137 mm` | **Median `137 mm`**, span `7 mm` (**19/19 valid**; `89 mm` drop from half depth) |

* **Key Finding**: Clear water transmits infrared laser light to the cup bottom or refracts away, causing `89%` invalid readings at full depth. Conversely, **turbid wastewater scatters the laser directly at the air-liquid interface**, yielding `100%` valid readings at full level with a tight `7 mm` spread.

### 3.3 Gravimetric Load Cell Sensing (`HX711` + `ESP32`)
* **Aim & Method**: A strain-gauge load cell interfaced via an `HX711` amplifier (`ESP32`) was tared and calibrated using a `20 g` reference weight to estimate fluid mass and volume ($\rho \approx 1\text{ g/mL}$) in a tapered cup without any optical, acoustic, or chemical contact.

| Fill Condition | Recorded Readings (g) | Representative Value (g) | Net Water Mass (g) | Estimated Volume |
| :--- | :--- | :--- | :--- | :--- |
| **Empty Cup (Baseline)** | `7.7, 7.7, 8.2, 8.3` | `7.95` (median) | `0.0 g` | `0 mL` |
| **Approximately Half Depth** | `161.9, 161.2` | `161.55` (mean) | `153.6 g` | `≈ 154 mL` (`40.3%` of full) |
| **Full Cup** | `388.4, 353.3, 389.5, 390.7` | `388.95` (median) | `381.0 g` | `≈ 381 mL` |

* **Key Finding**: At half depth, the tapered cup holds **`40.3%`** of full volume because the vessel widens toward the top. Using the **median** (`388.95 g`) effectively rejects mechanical placement outliers (`353.3 g`). For production accuracy, multi-point calibration near the full-load range (`300–400 g`) is recommended rather than extrapolating from a `20 g` weight.



#### Source code is available [[here](https://github.com/liusihan54/iot-portfolio-templateSihan/blob/main/Module04/README.md)](URL)
---

## 4. Comparative Sensor Selection Matrix

| Criterion | Resistive Probe (`HD-38` / `HW-038`) | Ultrasonic Sensor (`ESP32`) | ToF Laser (`VL53L0X`) | Load Cell (`HX711`) |
| :--- | :--- | :--- | :--- | :--- |
| **Freshwater** | Works (non-linear; `67%` drift) | **Recommended** (above dead-zone) | Poor (laser passes through clear water) | **Recommended** (immune to optics) |
| **Seawater** | Poor (rapid saturation & corrosion) | **Recommended** (non-contact) | Poor (clear water refraction loss) | **Recommended** (requires sturdy mount) |
| **Diesel / Fuel** | **Incompatible** (non-conductive) | **Recommended** (with guided float) | Moderate (depends on fuel opacity) | **Recommended** (density-calibrated) |
| **Wastewater** | Poor (bio-fouling bridges electrodes) | Good (watch `~18–20 cm` dead-zone) | **Effective** (turbidity reflects laser) | **Recommended** (zero fouling risk) |
| **Sloshing / Rocking** | High sensitivity (`adc_min/max` spikes) | Moderate (mitigated via stilling pipe) | Moderate (surface wave scatter) | Low (total mass remains constant) |

---

## 5. Industrial & Maritime Application Context

In real-world **maritime vessel fuel/ballast/blackwater monitoring** and **industrial fluid storage** (such as bulk diesel backup tanks, chemical dosing vessels, and municipal wastewater lift stations in **North Rhine-Westphalia, Germany**):

* **Resistive probes** are strictly restricted to simple binary high/low leak alarms in clean conductive water systems.
* **Non-contact ultrasonic or guided-wave radar sensors** housed inside a **perforated stilling pipe (baffle tube)** are the standard for fuel and ballast tanks—eliminating conductivity dependencies, preventing corrosion, and damping dynamic sloshing waves (while keeping sensors mounted above their minimum dead-zone).
* **Optical ToF sensors** are viable in opaque/turbid wastewater holding tanks where surface scum or suspended solids provide reliable diffuse reflection, whereas **external load cells (`HX711`)** offer completely non-invasive, fouling-proof inventory tracking for stationary process tanks.

---

## Repository Structure

```text
├── README.md
├── assets/                                   # Experiment plots & hardware photos
├── code/
│   ├── ultrasonic_reference_ESP32.ino        # Diesel ultrasonic distance measurement firmware
│   └── sketch2sensors.ino                    # Wastewater ultrasonic 5-sample threshold firmware
├── data/
│   ├── observed_screenshot_transcription.csv # Transcribed diesel test measurements
│   ├── simulate_rocking.py                   # Tank sloshing/rocking simulation script
│   └── simulated_shaking_illustrative_only.csv
└── notebooks/
    └── Module_4_Freshwater_Analysis.ipynb    # Freshwater calibration & drift analysis
```
