
# Task 1: The LDR's Transfer Function — Resistance vs. Light

## 1. Circuit Setup & Voltage Reference

The voltage divider was set up in the following orientation:

$$3.3\text{V} \longrightarrow R_{\text{fixed}} (10\text{ k}\Omega) \longrightarrow \text{Node (ADC Pin D15)} \longrightarrow \text{LDR} \longrightarrow \text{GND}$$

In this configuration, as light intensity increases, $R_{\text{LDR}}$ decreases, causing the voltage across the LDR ($V_{\text{out}}$) to decrease.

The exact formulas used to calculate $R_{\text{LDR}}$ from $V_{\text{out}}$ are:

$$V_{\text{out}} = V_s \cdot \frac{R_{\text{LDR}}}{R_{\text{LDR}} + R_{\text{fixed}}}$$

$$R_{\text{LDR}} = R_{\text{fixed}} \cdot \frac{V_{\text{out}}}{V_s - V_{\text{out}}}$$

- **Supply Voltage ($V_s$):** Measured with multimeter as $3.30\text{ V}$ (assumed/verified).
- **Fixed Resistor ($R_{\text{fixed}}$):** Nominal $10\text{ k}\Omega$.
- **ADC Logging:** ESP32 Pin D15 (GPIO15).

---

## 2. Experimental Measurement & Data Table

Measurements were taken under shielded conditions across 6 different distances $d$ using a stable light source. Relative illuminance was estimated using the inverse-square law $E_r \propto (15/d)^2$ with $d_0 = 15\text{ cm}$ as the reference $E_r = 1.0$.

| 距离 $d$ (cm) | $V_{\text{out}}$ (V) | 相对照度估计 $E_r = (15/d)^2$ | $R_{\text{LDR}}$ (k$\Omega$) |
| :--- | :---: | :---: | :---: |
| 15 | 1.60 | 1.0000 | 10.625 |
| 20 | 1.46 | 0.5625 | 12.603 |
| 25 | 1.36 | 0.3600 | 14.265 |
| 30 | 1.3467 (3-run mean) | 0.2500 | 14.505 |
| 40 | 1.31 | 0.1406 | 15.191 |
| 50 | 1.27 | 0.0900 | 15.984 |

---

## 3. Data Fitting & Log-Log Model Analysis

To extract the power-law relationship $R \propto E^{-\gamma}$, a log-log transformation was applied:

$$\log(R_{\text{LDR}}) = -\gamma \cdot \log(E_r) + C$$

- **Log-Log Power Fit:** $R = 11.307 \cdot E^{-0.159}$
- **Extracted Power Exponent ($-\gamma$):** $-0.159$
- **Linear Fit Comparison:** $R = 16.139 - 5.684 \cdot E$
- **Power Model RMSE:** $0.581\text{ k}\Omega$
- **Linear Model RMSE:** $0.248\text{ k}\Omega$

### Observation & Discussion
Although the linear

<img width="1792" height="1280" alt="42146eb338b89306977f803ebb65b23a" src="https://github.com/user-attachments/assets/2d4bbfc1-1b00-4c01-849e-fd43ac720179" />




# Task 2: The DS18B20 — Resolution, Accuracy, and Response Time

## 1. Datasheet Specifications & Key Parameters

- **Interface:** 1-Wire protocol (requires a $4.7\text{ k}\Omega$ pull-up resistor to $3.3\text{V}$).
- **Factory Accuracy:** $\pm 0.5^\circ\text{C}$ across the $-10^\circ\text{C}$ to $+85^\circ\text{C}$ range.
- **Configurable Resolutions & Theoretical Conversion Times:**
  - **9-bit:** Step size $= 0.5^\circ\text{C}$, conversion time $\approx 93.75\text{ ms}$
  - **10-bit:** Step size $= 0.25^\circ\text{C}$, conversion time $\approx 187.5\text{ ms}$
  - **11-bit:** Step size $= 0.125^\circ\text{C}$, conversion time $\approx 375\text{ ms}$
  - **12-bit:** Step size $= 0.0625^\circ\text{C}$, conversion time $\approx 750\text{ ms}$

---

## 2. Empirical Resolution & Conversion Observations

Empirical test results using the ESP32 and DallasTemperature library:
- **9-bit:** e.g., $20.5000^\circ\text{C}$ (Step size $= 0.5000^\circ\text{C}$)
- **10-bit:** e.g., $20.7500^\circ\text{C}$ (Step size $= 0.2500^\circ\text{C}$)
- **11-bit:** e.g., $21.0000^\circ\text{C}$ (Step size $= 0.1250^\circ\text{C}$)
- **12-bit:** e.g., $21.1250^\circ\text{C}$ (Step size $= 0.0625^\circ\text{C}$)

### Resolution vs. Accuracy for Frost Decision
- **Absolute Accuracy ($\pm 0.5^\circ\text{C}$):** Limits the reliability of absolute threshold decisions. For example, a reading of $+0.3^\circ\text{C}$ could physically be $-0.2^\circ\text{C}$.
- **Quantization Resolution ($0.0625^\circ\text{C}$):** Does not overcome intrinsic sensor inaccuracy, but enables precise tracking of fine thermal dynamics and rate of temperature drop before reaching freezing temperatures.

---

## 3. Dynamic Response Time Analysis (Cold-to-Warm Water Step)

The sensor probe was rapidly transitioned from cold water to warm water at 12-bit resolution to measure its thermal step response.

- **Initial Steady State ($T_{\text{cold}}$):** $\approx 12.1^\circ\text{C}$
- **Final Steady State ($T_{\text{warm}}$):** $\approx 45.3^\circ\text{C}$
- **Total Temperature Step ($\Delta T$):** $33.2^\circ\text{C}$
- **63.2% Response Threshold ($T_\tau$):** $12.1 + (33.2 \times 0.632) \approx 33.1^\circ\text{C}$
- **Calculated Time Constant ($\tau$):** Derived from time elapsed to reach $T_\tau$.
- **Settling Time ($4 \sim 5\tau$):** Time taken to reach within 95%–99% of final value ($45.3^\circ\text{C}$).

### Air vs. Water Response Comparison
Thermal response in still air is significantly slower than in stirred water. This is because water has a substantially higher thermal conductivity and volumetric heat capacity than air, resulting in a much higher convective heat transfer coefficient ($h_{\text{water}} \gg h_{\text{air}}$).

---

## 4. Unit-to-Unit Check & Reference Thermometer Comparison

Both the DS18B20 probe and a reference thermometer were placed close together in still air and allowed to settle.

- **Sample Count:** 10 consecutive readings (sampled every 3 seconds)
- **DS18B20 Mean Reading:** $22.431^\circ\text{C}$
- **DS18B20 Reading Spread:** $0.0625^\circ\text{C}$ ($22.3750^\circ\text{C} \sim 22.4375^\circ\text{C}$)
- **Reference Thermometer Reading:** $21.0^\circ\text{C}$
- **Mean Difference ($T_{\text{DS18B20}} - T_{\text{ref}}$):** $\approx +1.43^\circ\text{C}$

### Decision on Offset Correction
The DS18B20 demonstrated exceptional short-term repeatability with an extremely low noise spread ($0.0625^\circ\text{C}$). Although an average difference of $+1.43^\circ\text{C}$ was observed relative to the reference thermometer, **this difference was not applied as an offset correction**. Because the absolute accuracy of the reference thermometer itself was unverified, applying this correction risks introducing an uncalibrated systematic error.






### Task 3a: Two-Point Calibration of the Potentiometer–ADC Chain

**Experimental setup**

A 10 kΩ potentiometer was connected between 3.3 V and GND, with its wiper connected to ESP32 GPIO34. A multimeter measured the wiper voltage relative to GND and served as the reference. The ADC used 12-bit resolution and 11 dB attenuation. Each reported ADC value was the average of 100 samples.

**Calibration and validation data**

| Point | Multimeter voltage | Mean ADC reading | Purpose |
|---|---:|---:|---|
| Low point | 0.53 V | 525.31 | Calibration |
| High point | 2.72 V | 3180.32 | Calibration |
| Third point | 1.50 V | 1667.45 | Independent validation |

The third point was excluded from the calibration fit.

**Calibration equation**

\[
V_{\mathrm{cal}}=0.53+
\frac{ADC-525.31}{3180.32-525.31}\times(2.72-0.53)
\]

**Validation results**

Error is defined as calculated voltage minus reference voltage.

| Conversion method | Calculated voltage | Error at the validation point |
|---|---:|---:|
| Simple conversion: `ADC × 3.3 / 4095` | 1.344 V | −0.156 V |
| Two-point calibration | 1.472 V | −0.028 V |

The simple conversion assumes that the full-scale ADC code corresponds to 3.3 V and is used only as a comparison baseline.

After implementing the calibration equation, the serial monitor displayed approximately **1.472–1.473 V**, consistent with the calculation, while the multimeter reference was **1.50 V**.

**Conclusion and limitations**

Two-point calibration reduced the voltage error at the independent validation point. A residual error of approximately −27 to −28 mV remained.

This calibration corrected the ADC measurement chain’s conversion relationship. The remaining error may include ADC nonlinearity, reference-meter error, and changes in measurement conditions. This experiment does not isolate these contributions or establish accuracy across the entire voltage range.

**Data-quality note**

The initial pairing of 0.50 V with ADC 650.50 was not confirmed by simultaneous measurements. It was replaced by the newly measured pairing of 0.53 V with ADC 525.31. The final serial-monitor screenshot was retained as evidence of the implemented calibration and validation result.
