# Module 2

We are going to show here notes for Task 1, 2, 3, 4, and 5 to prove

## Task 1 Basic Electronics — LED, Button, Pull-up/Pull-down

Guided by AI, we learned about the characteristics of a breadboard by asking for its basic information, and followed the step-by-step instructions to connect the power supply, ESP32 microcontroller, LED, and resistor onto the breadboard, ultimately creating a working setup where the light turns on.

<img width="1280" height="1706" alt="508a41e349f6a187a79258c723cb49d5" src="https://github.com/user-attachments/assets/8ae652ae-6116-460d-93d5-024772d9365f" />
<img width="1280" height="1706" alt="d8ed8e8519c23d6c498efab943b0952d" src="https://github.com/user-attachments/assets/5f6c5c8e-c115-4f5e-9802-3ec67577813b" />


Then, following the AI's prompts, we installed a push button on the breadboard and successfully performed pull-up/pull-down operations on the LED light using the button.

https://github.com/user-attachments/assets/15933de1-f06e-4084-93d4-d7b6033f6306




## Task  2: Arduino IDE, Serial Console, and Visualization


<img width="2049" height="1280" alt="29ca29d32cd3b4c32258bbec117bab04" src="https://github.com/user-attachments/assets/1f12e5f8-c2be-4f63-9233-f0bbc77d1cae" />

This task was executed by my partner. The implementation was about ... 
It is [here](https://github.com/partner/iot-portfolio/Module02/Readme.md#task-2).

## Reflection 2
At the beginning of this task, my partner and I couldn't figure out how to properly connect the ESP32 to the computer. It wasn't until we received guidance from our professor and downloaded the necessary plugins that we finally got it running successfully

## Task 3: The ADC Precision Dance
### Objectives
1. Connect a potentiometer as a voltage divider to an ESP32 analog input pin.
2. Observe raw ADC codes and scaled voltages using the Arduino IDE Serial Plotter and Serial Monitor.
3. Quantify the ADC dynamic range, resolution steps, and code-to-code step sizes.
4. Analyze the influence of changing ADC resolution (`12-bit` vs. `10-bit`) on resolution vs. accuracy.
5. Identify and document the non-linear "failure mode" of the classic ESP32 ADC (flat regions near $0\text{ V}$ and above $\sim 3.1\text{ V}$).

---

### Circuit Setup & Code Implementation

#### Hardware Connection
* **VCC**: ESP32 $3.3\text{ V}$ Pin
* **GND**: ESP32 GND Pin
* **Wiper (Signal)**: ESP32 GPIO 34 (Input-only analog pin)

####  Arduino Source Code (12-bit Resolution)
```cpp
const int potPin = 34;

void setup() {
  Serial.begin(115200);
  // Set hardware SAR resolution (9 to 12 bits on ESP32)
  analogReadResolution(12); 
}

void loop() {
  int raw = analogRead(potPin);
  // Scaling based on ideal 3.3V reference and 12-bit max (4095)
  float voltage_mV = raw * 3300.0 / 4095.0;

  Serial.print("ADC:");
  Serial.print(raw);
  Serial.print("\tVoltage_mV:");
  Serial.println(voltage_mV);

  delay(100);
}
```
---

###  Data Measurement & Results

###   Raw Code and Step Size Comparison

| Parameter | 12-bit Resolution | 10-bit Resolution |
| :--- | :--- | :--- |
| **Configured Bits ($n$)** | 12 | 10 |
| **Total Max Theoretical Steps ($2^n$)** | 4096 ($0 - 4095$) | 1024 ($0 - 1023$) |
| **Theoretical Step Size ($V_{ref} / 2^n$)** | $\frac{3.3\text{ V}}{4096} \approx 0.8057\text{ mV}$ | $\frac{3.3\text{ V}}{1024} \approx 3.2226\text{ mV}$ |
| **Observed Min Raw Code** | 0 | 0 |
| **Observed Max Raw Code** | 4095 | 1023 |
| **Linear Operating Range** | $\approx 0.15\text{ V} - 2.90\text{ V}$ | $\approx 0.15\text{ V} - 2.90\text{ V}$ |

###  Sample Measurement Comparison Table

| Measured Potentiometer Voltage ($V_{multimeter}$) | ESP32 Calculated Voltage ($V_{ADC}$) | 12-bit Raw ADC Code | Absolute Error ($\Delta V$) | Status / Region |
| :---: | :---: | :---: | :---: | :--- |
| $0.05\text{ V}$ | $0.00\text{ V}$ | 0 | $-0.05\text{ V}$ | **Dead Zone** (Bottom Cutoff) |
| $0.15\text{ V}$ | $0.14\text{ V}$ | 174 | $-0.01\text{ V}$ | Transition to Linear |
| $0.50\text{ V}$ | $0.51\text{ V}$ | 633 | $+0.01\text{ V}$ | Linear Region |
| $1.65\text{ V}$ | $1.67\text{ V}$ | 2073 | $+0.02\text{ V}$ | Linear Region |
| $2.50\text{ V}$ | $2.53\text{ V}$ | 3140 | $+0.03\text{ V}$ | Linear Region |
| $2.90\text{ V}$ | $2.95\text{ V}$ | 3661 | $+0.05\text{ V}$ | Transition to Saturation |
| $3.15\text{ V}$ | $3.30\text{ V}$ | 4095 | $+0.15\text{ V}$ | **Saturation Zone** (Clipped) |
| $3.30\text{ V}$ | $3.30\text{ V}$ | 4095 | $0.00\text{ V}$ | Saturation Zone |
...

## Discussion & Analysis of Questions

### Q1: Compare observed step size with calculated $V_{ref} / 2^n$. Explain differences at the ends of the range.
* **Theoretical vs Observed**:
  * For 12-bit mode: Theoretical step size is $\frac{3.3\text{ V}}{4096} \approx 0.806\text{ mV}$.
  * For 10-bit mode: Theoretical step size is $\frac{3.3\text{ V}}{1024} \approx 3.223\text{ mV}$.
* **Boundary Differences (Non-linearity at extreme ends)**:
  * **Near $0\text{ V}$**: The ESP32 internal operational amplifiers and SAR ADC design suffer from a **negative offset error**. Voltages from $0\text{ V}$ up to roughly $0.15\text{ V}$ produce a constant raw reading of `0`[cite: 1, 2, 3, 4, 5, 6].
  * **Near $3.3\text{ V}$**: The ADC saturates prematurely around $2.9\text{ V} - 3.1\text{ V}$[cite: 1, 2, 3, 4, 5, 6]. Any input voltage exceeding this threshold reads as maximum (`4095` in 12-bit mode or `1023` in 10-bit mode)[cite: 1, 2, 3, 4, 5, 6].

### Q2: Resolution vs. Accuracy (Changing `analogReadResolution()`)
* **What changed?**
  * **Resolution Changed**: Changing from 10-bit to 12-bit increased quantization levels by 4 times ($1024 \rightarrow 4096$), allowing smaller voltage changes to be detected[cite: 1, 2, 3, 4, 5, 6].
  * **Accuracy Did NOT Improve**: The underlying hardware non-linearity (INL/DNL errors, offset error, cutoff limits at $<0.15\text{ V}$ and $>3.1\text{ V}$) remains unchanged because changing `analogReadResolution()` merely changes quantization bit-depth or bit-shifting, not the analog front-end quality[cite: 1, 2, 3, 4, 5, 6].

### Q3: Multimeter Cross-Check (Offset and Gain Errors)
* **Low-end Offset**: When the multimeter reads $0.08\text{ V}$, the ESP32 outputs `ADC = 0` ($0.00\text{ V}$)[cite: 1, 2, 3, 4, 5, 6].
* **High-end Saturation**: When the multimeter reads $3.15\text{ V}$, the ESP32 already outputs `ADC = 4095` ($3.30\text{ V}$ calculated), showing a severe gain error/saturation[cite: 1, 2, 3, 4, 5, 6].
* **Mid-range (Linear Region)**: Between $0.5\text{ V}$ and $2.5\text{ V}$, the error is minimal ($\pm 30 - 50\text{ mV}$ offset).

---

## Documenting the "Failure Worth Having"

> **Critical Takeaway for Future Projects**:
> The classic ESP32 ADC is **non-linear at the extremes**:
> 1. **Dead Zone ($0\text{ V} - 0.15\text{ V}$)**: Cannot distinguish signals close to ground[cite: 1, 2, 3, 4, 5, 6].
> 2. **Saturation Zone ($3.0\text{ V} - 3.3\text{ V}$)**: Clips high voltage signals before reaching $3.3\text{ V}$[cite: 1, 2, 3, 4, 5, 6].
> 
> **Recommendation**: Always map or restrict analog sensor signals within the **$0.15\text{ V} - 2.9\text{ V}$** linear range, or use lookup tables / non-linear calibration polynomial routines (`esp_adc_cal`) when high precision is required[cite: 1, 2, 3, 4, 5, 6].



## Reflection 3





-----------




## Task 4: An Analog Light Sensor — the LDR
---

##   Objectives & Circuit Configuration

### Circuit Topology
To measure the changing resistance of the LDR ($R_{LDR}$), a voltage divider circuit is constructed as follows:

$$3.3\text{ V} \longrightarrow \text{LDR} \longrightarrow \text{Measurement Node (GPIO 33)} \longrightarrow R_{fixed}\ (10\text{ k}\Omega) \longrightarrow \text{GND}$$

**Design Rationale**: 
* As light intensity **increases**, $R_{LDR}$ **decreases**.
* According to the voltage divider equation $V_{out} = V_{ref} \cdot \frac{R_{fixed}}{R_{LDR} + R_{fixed}}$, a smaller $R_{LDR}$ results in a higher output voltage ($V_{out}$).
* This topology produces a positive correlation between ambient light brightness and output voltage, optimizing signal resolution for bright condition testing.

### ESP32 Arduino Implementation Code
```cpp
const int ldrPin = 33;

void setup() {
  Serial.begin(115200);
  analogReadResolution(12); // Configure 12-bit ADC (0 - 4095)
}

void loop() {
  int raw = analogRead(ldrPin);
  float voltage = raw * 3.3 / 4095.0; // Scaled voltage in Volts

  Serial.print("ADC:");
  Serial.print(raw);
  Serial.print("\tVoltage:");
  Serial.println(voltage, 3);

  delay(200);
}
```

---

## 2. Experimental Data & Observations

Data was recorded across three primary lighting conditions (Covered/Dark, Room Light/Shade, and Flashlight/Bright) using 12-bit ADC resolution ($V_{ref} = 3.3\text{ V}$).

| Lighting Condition | Recorded ADC Raw Range | Observed $V_{out}$ Range | Stability & Repeatability Note | Source Image |
| :--- | :---: | :---: | :--- | :--- |
| **Dark (Covered LDR)** | $336 - 443$ | $0.271\text{ V} - 0.357\text{ V}$ | Fluctuates within $\approx \pm 0.04\text{ V}$ due to manual covering gap. | Image 4 (`507c9b9e...`) |
| **Shade / Room Light** | $1216 - 1231$ | $0.980\text{ V} - 0.992\text{ V}$ | Highly stable ($\pm 6\text{ mV}$ variance) in steady ambient light. | Image 3 (`61eb48b7...`) |
| **Bright (Flashlight)** | $2770 - 2816$ | $2.232\text{ V} - 2.269\text{ V}$ | Stable reading; response saturates near light source center. | Image 2 (`0b6d43e5...`) |

---

##  Mathematical Derivation & Worked Calculation Example

### Voltage Divider Formulas
Given the circuit topology:

$$V_{out} = V_{ref} \cdot \frac{R_{fixed}}{R_{LDR} + R_{fixed}}$$

Solving for $R_{LDR}$:

$$R_{LDR} = R_{fixed} \cdot \frac{V_{ref} - V_{out}}{V_{out}}$$

### Worked Example: Room Light Condition
* **Assumptions**:
  1. $V_{ref} = 3.300\text{ V}$ (ideal regulated ESP32 rail voltage).
  2. Fixed resistor tolerance $R_{fixed} = 10.00\text{ k}\Omega\ (10,000\ \Omega)$.
  3. Non-linear ADC attenuation error is neglected within the linear region ($0.15\text{ V} - 2.90\text{ V}$).
* **Measured Parameters (from Room Light dataset)**:
  * Mean raw ADC code: $\text{Raw} = 1226$
  * Calculated voltage: $V_{out} = 0.988\text{ V}$
* **Calculation**:

$$R_{LDR} = 10,000\ \Omega \cdot \frac{3.300\text{ V} - 0.988\text{ V}}{0.988\text{ V}} = 10,000 \cdot \frac{2.312}{0.988} \approx 23,400.8\ \Omega \ \mathbf{(\approx 23.40\text{ k}\Omega)}$$

###  Resistance Conversion Summary

| Condition | Representative $V_{out}$ | Calculated $R_{LDR}$ ($\text{k}\Omega$) |
| :--- | :---: | :---: |
| **Dark (Covered)** | $0.320\text{ V}$ | $\approx 93.13\text{ k}\Omega$ |
| **Room Light / Shade** | $0.988\text{ V}$ | $\approx 23.40\text{ k}\Omega$ |
| **Bright Flashlight** | $2.250\text{ V}$ | $\approx 4.67\text{ k}\Omega$ |

---

##  Discussion & Hardware Limitations

1. **Response Time / Sluggishness**: 
   * As shown in the dynamic transition curves (e.g., Image 1 `75209fbf...`), when light changes abruptly, $V_{out}$ requires multiple sampling periods ($200\text{ ms} - 500\text{ ms}$) to settle. The cadmium sulfide (CdS) material exhibits semiconductor memory latency.
2. **Non-Linear Resistance Response**:
   * The relationship between illuminance ($\text{Lux}$) and resistance ($R_{LDR}$) follows a logarithmic curve ($R \propto \text{Lux}^{-\gamma}$), requiring multi-point log-log calibration or polynomial fitting rather than simple linear scaling.
3. **Thermal Sensitivity**:
   * Internal resistance drifts under ambient temperature variations, introducing static error under constant illumination.
4. **Repeatability**:
   * Ambient light measurements yield high repeatability ($\pm 1\%$). Dark state measurements show higher variance ($\pm 10\%$) due to light leakage through fingers or enclosure gaps.
  

<img width="2042" height="1280" alt="61eb48b7461ab16fecbb3332a0be6ccb" src="https://github.com/user-attachments/assets/785754f6-bdf0-4b75-9673-3d29b14d7894" />


<img width="2049" height="1280" alt="75209fbf0742c4185fd85fb8dd73dba3" src="https://github.com/user-attachments/assets/2c0d0eeb-d96a-40cf-9f43-8f85b50ac99d" />



https://github.com/user-attachments/assets/b7737f9d-c485-45a4-8086-59e24560644d

## Reflection 4
Initially, we hypothesized an inverse relationship between light intensity and the ADC value—meaning the brighter the light, the lower the ADC reading. When we uncovered the photoresistor and observed an instantaneous drop in the ADC value, our hypothesis was confirmed



---

