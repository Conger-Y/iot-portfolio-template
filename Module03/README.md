
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
<img width="1280" height="799" alt="a3075b3d98aafcfc2e634419550ab8b7" src="https://github.com/user-attachments/assets/7e2a13e8-3817-44c3-8f10-2756046f932c" />

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
<img width="1280" height="799" alt="a3075b3d98aafcfc2e634419550ab8b7" src="https://github.com/user-attachments/assets/80e7f876-d568-4e1d-8f2f-d105de6fd9fd" />

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
<img width="1280" height="800" alt="06e50db1c1781a416911b4ac23742310" src="https://github.com/user-attachments/assets/a1236af3-c56d-4f58-871f-17f83f33609d" />
<img width="1280" height="800" alt="9a56dca9d9561b17b3ac9066e79e6b38" src="https://github.com/user-attachments/assets/2989ae1c-c5ef-4137-80e4-891c4d0057f8" />

---

## 4. Unit-to-Unit Check & Reference Thermometer Comparison

Both the DS18B20 probe and a reference thermometer were placed close together in still air and allowed to settle.

- **Sample Count:** 10 consecutive readings (sampled every 3 seconds)
- **DS18B20 Mean Reading:** $22.431^\circ\text{C}$
- **DS18B20 Reading Spread:** $0.0625^\circ\text{C}$ ($22.3750^\circ\text{C} \sim 22.4375^\circ\text{C}$)
- **Reference Thermometer Reading:** $21.0^\circ\text{C}$
- **Mean Difference ($T_{\text{DS18B20}} - T_{\text{ref}}$):** $\approx +1.43^\circ\text{C}$
<img width="1278" height="795" alt="ae85d828997d04cab043571b4a983fd7" src="https://github.com/user-attachments/assets/bc3ed5c5-fd1b-45f3-87ed-33687c9b6c33" />

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


### Task 3b: Two-Point Calibration and Validation of the LDR

**Method**

The original, unshielded LDR measurements were used. The 15 cm and 50 cm measurements were selected as calibration points. The 25 cm measurement was reserved for independent validation and was excluded from both fits.

Supply voltage was assumed to be 3.30 V, and the fixed resistor was nominally 10 kΩ.

LDR resistance was calculated using:

\[
R_{\mathrm{LDR}}=10\,\mathrm{k}\Omega
\frac{3.30-V_{\mathrm{out}}}{V_{\mathrm{out}}}
\]

Relative illuminance was approximated from distance:

\[
E_r=\left(\frac{15}{d}\right)^2
\]

Here, \(d\) is in centimetres. This is a distance-based proxy, not a calibrated lux measurement.

**Data**

| Purpose | Distance | Vout | Relative illuminance proxy | LDR resistance |
|---|---:|---:|---:|---:|
| Calibration point 1 | 15 cm | 1.60 V | 1.000 | 10.625 kΩ |
| Calibration point 2 | 50 cm | 1.27 V | 0.090 | 15.984 kΩ |
| Independent validation | 25 cm | 1.36 V | 0.360 | 14.265 kΩ |

**Power-law calibration**

Fitting the two calibration points in logarithmic space gave:

\[
R_{\mathrm{power}}=10.625E_r^{-0.1696}
\quad[\mathrm{k}\Omega]
\]

Thus, the fitted exponent was \(\gamma=0.1696\).

**Linear calibration**

A straight line through the same two calibration points gave:

\[
R_{\mathrm{linear}}=16.514-5.889E_r
\quad[\mathrm{k}\Omega]
\]

**Third-point validation**

At 25 cm, the voltage-derived resistance was 14.265 kΩ. Error is defined as predicted resistance minus voltage-derived resistance.

| Model | Predicted resistance | Error | Relative error |
|---|---:|---:|---:|
| Power law | 12.635 kΩ | −1.630 kΩ | −11.42% |
| Linear | 14.394 kΩ | +0.129 kΩ | +0.91% |

The linear model predicted this independent validation point more closely than the power-law model.

**Uncertainty statement**

The validation result was approximately 14.26 kΩ. Ambient-light contamination of the illuminance estimate and illumination changes caused by positioning or shadowing were considered the two largest likely limitations; their contributions were not quantified, so no numerical expanded uncertainty is claimed.

**Interpretation and calibration decision**

The lamp-off output was approximately 1.30 V, close to the readings at the larger distances. Ambient daylight therefore substantially affected the measurements, and the distance-based illuminance proxy did not represent total illumination.

The smaller linear-model error at one validation point does not establish that the LDR is physically linear or that the model will generalize to other conditions. Likewise, the fitted power-law exponent should not be treated as a reliable intrinsic sensor parameter.

Both models were retained for comparison, but neither was accepted as a reliable lux calibration. No firmware correction was applied. Reliable light calibration would require controlled illumination or measured illuminance and a quantified uncertainty budget.



### Task 3C:  The digital sensor (DS18B20)
An initial ice-bath trial produced 10 readings with a mean of 8.7563°C. The bath temperature at the probe was not verified. With no ice left for a repeat trial, the 0°C validation remained incomplete. The readings were retained as preliminary data, and no offset correction was applied based on this trial.








### Task 4: Hysteresis and repeatability — where the loop is real

**LDR — light up and down**



The lamp and LDR were kept fixed while a card was moved through five marked positions. Position 1 blocked the most light, and position 5 was fully unblocked. The output voltage was measured at each position while increasing and then decreasing illumination.

| Position            | Increasing illumination (V) | Decreasing illumination (V) | Absolute difference (V) |
| ------------------- | --------------------------: | --------------------------: | ----------------------: |
| 1 — most blocked    |                       0.328 |                       0.865 |                   0.537 |
| 2                   |                       1.433 |                       1.287 |                   0.146 |
| 3                   |                       1.634 |                       1.278 |                   0.356 |
| 4                   |                       1.735 |                       1.625 |                   0.110 |
| 5 — fully unblocked |                       1.632 |                       1.623 |                   0.009 |

The largest difference was **0.537 V** at position 1, the start and end of the sweep. Among the three intermediate positions, the largest difference was **0.356 V** at position 3. A separate fully unblocked reading of **1.715 V** was higher than both position 5 readings from the sweep.

The first sweep showed that the readings differed between the two directions at the same marked positions. Hysteresis, response lag, and variations in card placement or ambient light could all contribute. A slower sweep with more time for the readings to settle is still pending.


**Potentiometer — mechanical repeatability**


**Objective**

To measure how consistently a potentiometer returns to the same output voltage when its shaft is manually turned back to a marked position.

**Setup and procedure**

A 10 kΩ potentiometer was connected across the ESP32’s 3.3 V supply and ground. A digital multimeter in DC voltage mode measured the wiper voltage relative to ground.

The potentiometer was adjusted to 1.527 V, and its shaft position was marked. Three voltage readings were taken without moving the shaft. The shaft was then turned away from the mark and returned to it ten times, approaching the mark from the same direction each time. The voltage was recorded after releasing the shaft. The mark was used to set the position for each trial.

**Raw data**

| Reading | Shaft held fixed (V) | Returned to the mark (V) |
| ------: | -------------------: | -----------------------: |
|       1 |                1.527 |                    1.559 |
|       2 |                1.527 |                    1.509 |
|       3 |                1.526 |                    1.439 |
|       4 |                    — |                    1.563 |
|       5 |                    — |                    1.506 |
|       6 |                    — |                    1.456 |
|       7 |                    — |                    1.528 |
|       8 |                    — |                    1.563 |
|       9 |                    — |                    1.538 |
|      10 |                    — |                    1.504 |

**Results**

The range was calculated as the maximum reading minus the minimum reading.

| Condition            | Samples | Mean voltage (V) | Minimum (V) | Maximum (V) | Range (V) |
| -------------------- | ------: | ---------------: | ----------: | ----------: | --------: |
| Shaft held fixed     |       3 |            1.527 |       1.526 |       1.527 |     0.001 |
| Returned to the mark |      10 |            1.517 |       1.439 |       1.563 |     0.124 |

**Discussion and conclusion**

The voltage varied by only **1 mV** across the three fixed-position readings. After manual repositioning, the range increased to **124 mV**. The lowest return reading, 1.439 V, was **88 mV below** the initial marked-position reading of 1.527 V.

Returning the shaft to the same visual mark produced substantially more variation than was observed with the shaft held fixed. The 124 mV range describes the repeatability of the manual repositioning procedure. The measurements do not separate the effects of visual alignment and mechanical play.



**DS18B20 — repeatability and resolution**


The same DS18B20 probe was tested in room air at two resolution settings. Ten consecutive readings were recorded at each setting.

| Resolution | Valid readings | Reading step (°C) | Mean (°C) | Minimum (°C) | Maximum (°C) | Range (°C) |
| ---------- | -------------: | ----------------: | --------: | -----------: | -----------: | ---------: |
| 9 bit      |             10 |            0.5000 |   22.5000 |      22.5000 |      22.5000 |     0.0000 |
| 12 bit     |             10 |            0.0625 |   23.6938 |      23.6250 |      23.7500 |     0.1250 |

All ten readings at 9-bit resolution were 22.5000°C. At 12-bit resolution, the readings spanned 0.1250°C, equivalent to two 12-bit increments. The coarser 9-bit setting can report the same value even when the temperature changes slightly.

The two settings were tested in separate runs. The difference between their mean temperatures cannot be attributed to resolution alone. This experiment measured short-term repeatability for one probe. Comparison between probes was limited by equipment availability.



 ### Task 5: Noise and filtering — measure the cost



## 1. Sensor Noise and Filter Comparison

To evaluate noise reduction and robustness against synthetic outliers, 500 samples were logged from the LDR ($\sim 20\text{ ms}$ sampling interval) and processed through three filters:
- **5-sample Moving Average**
- **5-sample Median**
- **Exponential Moving Average (EMA, $\alpha = 0.2$)**

<img width="1732" height="1221" alt="be6f5245c4534b64205f03d782f14d25" src="https://github.com/user-attachments/assets/a4a441a5-d108-4151-be3f-60291834d534" />


### Synthetic Spike Response Analysis (Sample 470)
A synthetic $+500$ ADC count spike was injected into the LDR dataset to simulate brief optical or electrical transients:
- **Median Filter (5-sample):** Completely rejected the single-sample outlier ($0$ count deviation from baseline).
- **Moving Average (5-sample):** Attenuated the spike from $+500$ to $+100$ counts, but spread the disturbance across 5 consecutive output samples.
- **EMA ($\alpha = 0.2$):** Reduced peak impact to $+100$ counts, but introduced an exponential tail that degraded settling performance over $\sim 15\text{--}20$ samples.

---

## 2. Filter Lag and Response Time Trade-Off

Using a manual potentiometer ramp transition crossing the midpoint threshold ($1720.5\text{ ADC counts}$), the phase lag introduced by each filter was measured relative to the raw ADC signal:
<img width="1732" height="1171" alt="dc3408e89c75ad5b9a9b611981e730c6" src="https://github.com/user-attachments/assets/7b8b3ce4-6490-472a-9151-2dd7b8410284" />


- **Raw ADC Crossing Time:** $\approx 4.995\text{ s}$
- **Median (5) Crossing Time:** $\approx 5.038\text{ s}$ ($\Delta t \approx 43\text{ ms}$, $\sim 2$ samples delay)
- **Moving Average (5) Crossing Time:** $\approx 5.042\text{ s}$ ($\Delta t \approx 47\text{ ms}$, group delay $\frac{N-1}{2} = 2$ samples)
- **EMA ($\alpha = 0.2$) Crossing Time:** $\approx 5.080\text{ s}$ ($\Delta t \approx 85\text{ ms}$, largest phase lag)

> **Trade-Off Summary:** Stronger noise smoothing comes at the direct cost of increased phase lag. A 5-sample median filter offers the best compromise by providing instantaneous outlier rejection with minimal group delay.

---

## 3. Threshold Decision for Frost Alarm (Hysteresis Evaluation)

A simulated temperature sequence was used to compare two alarm rules as the temperature fluctuated around 2.0°C:
- **Single-Threshold Alarm:** Switched on at or below 2.0°C and switched off above 2.0°C.
- **Hysteresis Alarm:** Used the same activation threshold ($\le 2.0^\circ\text{C}$) but stayed on until the temperature reached $2.5^\circ\text{C}$.

### Experimental Findings & Selection
The single-threshold alarm changed state **six times**, while the hysteresis alarm changed state only **twice**. Both activated on the first reading below 2.0°C. 

I would choose hysteresis for the frost alarm because it drastically reduced repeated switching (chattering) near the decision boundary while preserving the exact same activation threshold in this test.
 

 ### Task 6 (Stretcher): Uncertainty and Declarative Filtering


## 1. Voltage Measurement Uncertainty Budget

The potentiometer voltage was measured with the ESP32 after two-point calibration at an independent validation point, yielding an estimated value of **$1.472\text{ V}$**. The reference multimeter read **$1.50\text{ V}$**, resulting in a validation residual of approximately **$-28\text{ mV}$**.

### Uncertainty Budget Table

| Contribution | Estimate | Distribution | Standard uncertainty | Source |
| :--- | :---: | :---: | ---: | :--- |
| **Reference Multimeter** | $\pm 10.4\text{ mV}$ | Rectangular ($a/\sqrt{3}$) | $6.0\text{ mV}$ | PCW03A 9.999V DC range accuracy limits propagated through interpolation |
| **Short-term Repeatability** | $\pm 1.25\text{ mV}$ | Normal (SE of mean) | $0.72\text{ mV}$ | Standard error of the mean from 3 consecutive averaged ADC readings |
| **Model Discrepancy (Fit Residual)** | $\pm 28.0\text{ mV}$ | Rectangular ($a/\sqrt{3}$) | $16.2\text{ mV}$ | Single validation point residual treated as provisional half-width $a$ |
| **Nonlinearity / Quantization** | $< 0.8\text{ mV}$ | Rectangular ($a/\sqrt{3}$) | Negligible | ESP32 12-bit ADC quantization step ($\approx 0.8\text{ mV}$) |
| **Drift & Self-heating** | $< 1.0\text{ mV}$ | Bounded | Negligible | Short measurement duration in stable indoor environment |

### Combined and Expanded Uncertainty Calculation
Combining the standard uncertainty contributions in quadrature:

$$u_c = \sqrt{6.0^2 + 0.72^2 + 16.2^2} \approx \sqrt{36 + 0.5184 + 262.44} = 17.29\text{ mV} \approx 17\text{ mV}$$

Applying a coverage factor of **$k = 2$** (for an approximate $95\%$ confidence level):

$$U = k \cdot u_c = 2 \times 17.29\text{ mV} = 34.58\text{ mV} \approx 0.035\text{ V}$$

**Final Reported Voltage Result:**
$$\mathbf{V_{\text{pot}} = 1.472 \pm 0.035\text{ V} \quad (k = 2)}$$

---

## 2. Software Filter Performance Comparison

In the relatively stable section of the LDR recording:
- **Exponential Moving Average (EMA, $\alpha = 0.2$):** Reduced the sample standard deviation from **$5.368$ to $3.979\text{ ADC counts}$**. Its measured group delay during the potentiometer transition was approximately **$84\text{ ms}$**.
- **5-Point Median Filter:** Added approximately **$40\text{ ms}$** of phase lag and proved to be the most effective filter for suppressing artificial single-sample impulse spikes.

---

## 3. Dominant Contribution & Improvement Priority

- **Dominant Uncertainty Source:** The **Calibration model discrepancy ($16.2\text{ mV}$)** is by far the largest contribution to the uncertainty budget, accounting for over $85\%$ of the combined variance.
- **Improvement Strategy:** I would first check the calibration at **more independent voltage points across the range**. This would verify whether the observed $-28\text{ mV}$ discrepancy represents a localized nonlinearity or a systematic offset bias. While additional filtering reduces short-term noise, correcting the calibration model bias is required to reduce the overall uncertainty boundary.
