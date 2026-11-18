# Module 05 — LEDs & Light as Output

**Board:** MH-ET LIVE ESP32 MiniKit (`mhetesp32minikit`)  
**Framework & Toolchain:** Arduino framework via PlatformIO CLI (`pio`) / Arduino IDE, Serial monitor at `115200` baud

## Repository Structure

```text
Module05/
├── README.md          # Lab report, measurements, power budget, and study log
├── task1-led/         # Task 1: Single-channel PWM fade on GPIO 16
├── task2-rgb/         # Task 2: Common-cathode RGB LED serial REPL (GPIO 16, 17, 18)
└── task3-strip/       # Task 3: 8-pixel WS2812B addressable strip via NeoPixelBusLg (GPIO 16)
```

---

## Task 1 — Single LED PWM Dimming (`task1-led/`)

### Hardware & Firmware Setup
* **Wiring:** A single indicator LED is connected from **GPIO 16** through a current-limiting resistor to **GND**.
* **Implementation:** `src/main.cpp` configures `GPIO 16` as an output and uses `analogWrite(LED_PIN, duty)` to sweep the 8-bit PWM duty cycle from `0` to `255` ($20\text{ ms}$ per step, $\sim 5.1\text{ s}$ fade-in), holds full brightness for $1000\text{ ms}$, sweeps back down from `255` to `0` ($\sim 5.1\text{ s}$ fade-out), and pauses for $1000\text{ ms}$ before repeating.

### Key Concept: Duty Cycle
Pulse-width modulation (PWM) switches the GPIO output rapidly between $0\text{ V}$ and $3.3\text{ V}$. The **duty cycle** (`0` to `255`, corresponding to $0\%$ to $100\%$ on-time per period) determines the average current delivered to the LED and therefore its perceived brightness.

---

## Task 2 — RGB LED Mixing & Channel Current Measurement (`task2-rgb/`)

### Hardware & Firmware Setup
* **Wiring:** A common-cathode RGB LED module (`COMMON_ANODE = false`) with **onboard** $150\ \Omega$ **SMD current-limiting resistors** (marked `151`) is connected as follows:
  * **Red channel (`R`)** $\rightarrow$ `GPIO 16`
  * **Green channel (`G`)** $\rightarrow$ `GPIO 17`
  * **Blue channel (`B`)** $\rightarrow$ `GPIO 18`
  * **Common cathode (`-`)** $\rightarrow$ `GND`
* **Firmware (`src/main.cpp`):** Reads newline-terminated ASCII commands (`R G B`) at `115200` baud, validates field count (`sscanf` returning `3` integers with no trailing characters) and buffer length (`64` bytes max), clamps values to `0`–`255` via `constrain()`, applies PWM duty cycles via `analogWrite()`, and echoes the applied RGB tuple or prints an error message on malformed input.
* **Functional & Serial Validation:**
  


https://github.com/user-attachments/assets/75f189c8-21aa-4c46-b497-48240b801f66



<img width="2065" height="1280" alt="4d50b694f6729c049802195ec928b1ec" src="https://github.com/user-attachments/assets/ffb5cb38-bf41-4a6c-aa4d-552259a47604" />

  * **Valid color inputs:** Verified primary colors (`255 0 0` echoing `Applied RGB: 255 0 0` for red, `0 255 0` for green, `0 0 255` for blue) as well as additive color mixing on the RGB module.
  * **Invalid input handling:** Verified that non-numeric or malformed strings such as `abc` are rejected without altering the LED state, returning `ERROR: Enter three integers, e.g. 255 0 0` over Serial.

### Channel Voltage & Current Measurements
Each color channel was driven individually at full duty cycle (`PWM = 255`), and the voltage drop $U$ across its onboard $150\ \Omega$ resistor was measured with a UNI-T UT131C multimeter. Channel current was calculated using Ohm's law ($I = U / 150\ \Omega$):

| Channel | Onboard Resistor | Measured Resistor Voltage ($U$) | Calculated Current ($I = U / R$) |
| :--- | ---: | ---: | ---: |
| **Red** | $150\ \Omega$ | $1.07\text{ V}$ | **$7.13\text{ mA}$** |
| **Green** | $150\ \Omega$ | $0.63\text{ V}$ | **$4.20\text{ mA}$** |
| **Blue** | $150\ \Omega$ | $0.50\text{ V}$ | **$3.33\text{ mA}$** |
<img width="1280" height="2275" alt="822c05fa6a0811fa4690c0dfd2cdb64e" src="https://github.com/user-attachments/assets/5bb3cfde-098a-4287-bb4d-829b6cc7e3af" />
<img width="2400" height="1080" alt="ab6b3099ef6a2288e6ca4466eea5e4c5" src="https://github.com/user-attachments/assets/2101c879-a908-4997-9a51-5aabf847920b" />

### Analysis of Channel Imbalance
Although all three channels use identical $150\ \Omega$ resistors on a $3.3\text{ V}$ GPIO rail, the red channel draws more than twice the current of the blue channel. Because an LED clamps its forward voltage ($V_f \approx 1.9\text{ V}$ for red, $\approx 2.1\text{ V}$ for green, and $\approx 2.8\text{ V}$ to $3.2\text{ V}$ for blue), the remaining voltage dropped across the series resistor ($U_R = V_{\text{GPIO}} - V_f$) is largest for red and smallest for blue, directly explaining the measured current difference.

---

## Task 3 — Addressable WS2812B Strip & Power Budget (`task3-strip/`)

### Aim & Setup
We controlled an eight-pixel WS2812B LED strip with the ESP32 MiniKit. The strip's `DIN` input was connected to **GPIO 16**, its power input ($5\text{ V}$) to the USB-powered `VIN`/`5V` rail, and its ground (`GND`) to ESP32 `GND`.

In `src/main.cpp`, we instantiated `NeoPixelBusLg<NeoGrbFeature, NeoEsp32Rmt0Ws2812xMethod, NeoGammaNullMethod>` using **GRB** color order, **ESP32 hardware RMT Channel 0** (offloading the $800\text{ kHz}$ NRZ timing waveform to dedicated peripheral hardware so Wi-Fi/BLE interrupts do not corrupt frames), `NeoGammaNullMethod` (no extra gamma curve), and a luminance limit of **`80 / 255`** ($\approx 31.4\%$) via `strip.SetLuminance(80)`.

### Control Test
The serial REPL (`115200` baud) accepted commands in the format `index R G B` (`index` `0`–`7`, `R/G/B` `0`–`255`) as well as `clear`:
* Sending `0 255 0 0` illuminated pixel `0` in red (`OK: LED 0 = 255 0 0`), and `clear` turned all eight pixels off.
* Successive commands updated individual pixels while retaining colors already assigned to other pixels, allowing all eight pixels to be set uniformly (`0 255 0 0` through `7 255 0 0` for all-red; `0 255 255 255` through `7 255 255 255` for all-white) or in multi-color patterns.
* Out-of-range indices (e.g., `9 255 255 255`) and malformed strings were rejected with descriptive error messages.
<img width="1024" height="1820" alt="640296a53d383575c97c5c7c07273b7b" src="https://github.com/user-attachments/assets/75d128a5-f5bb-469e-b46c-f0028332e94d" />

### Current Measurement (8-Pixel Strip)
We connected the UNI-T UT131C multimeter in series with the strip's positive $5\text{ V}$ supply line on its **DC `10 A`** range and measured current at luminance `80 / 255` (power estimated assuming $V_{\text{supply}} = 5\text{ V}$):
<img width="1280" height="2275" alt="2c476fe8992c46ff4d2b2764aa213eee" src="https://github.com/user-attachments/assets/91db4c84-9507-4128-b107-ae81962e372f" />
<img width="1280" height="2275" alt="cb9f911e80154b0d17be970d44c66367" src="https://github.com/user-attachments/assets/86d8091f-36a3-42fd-ae23-9765645fe65b" />


| Condition | Luminance Setting | Measured Strip Current | Estimated Strip Power ($5\text{ V}$) |
| :--- | ---: | ---: | ---: |
| **Eight pixels red** (`255 0 0`) | `80 / 255` | $\approx 0.03\text{ A}$ ($30\text{ mA}$) | $\approx 0.15\text{ W}$ |
| **Eight pixels white** (`255 255 255`) | `80 / 255` | $\approx 0.09\text{ A}$ ($90\text{ mA}$) | $\approx 0.45\text{ W}$ |

* **Color effect:** Full white activates all three internal sub-LEDs (red, green, and blue) per pixel, drawing approximately three times the current of the single-channel red setting.
* **Measurement scope & precision:** These readings reflect only the LED strip branch (excluding the ESP32's own operating current). Using the `10 A` multimeter range limits resolution to $0.01\text{ A}$ ($10\text{ mA}$).

### Power Budget Scaling (8 to 144 Pixels)
Using the worst-case design rule of **$60\text{ mA}$ per WS2812B pixel** at full-brightness white (`255 255 255` at `255 / 255` luminance) on a $5\text{ V}$ supply, alongside the scaled budget at our firmware's `80 / 255` luminance cap:

| Number of Pixels | Full-White Current ($60\text{ mA/px}$) | Full-White Power ($5\text{ V}$) | Scaled Current at `80 / 255` ($\approx 31.4\%$) | Recommended Power Supply Strategy |
| ---: | ---: | ---: | ---: | :--- |
| **8** | $0.48\text{ A}$ | $2.4\text{ W}$ | $\approx 0.15\text{ A}$ | Safe on a standard USB 2.0 port ($500\text{ mA}$ limit) when luminance-capped; full white approaches the USB 2.0 ceiling once ESP32 current ($\sim 100\text{ mA}$ to $250\text{ mA}$) is added. |
| **30** | $1.80\text{ A}$ | $9.0\text{ W}$ | $\approx 0.56\text{ A}$ | Exceeds USB 2.0 ($500\text{ mA}$); requires a dedicated $5\text{ V} \ge 2\text{ A}$ supply (or USB 3.0 / USB-C only when strictly brightness-limited). |
| **60** | $3.60\text{ A}$ | $18.0\text{ W}$ | $\approx 1.13\text{ A}$ | Must use an external $5\text{ V} \ge 4\text{ A}$ regulated power supply wired directly to the strip with common `GND` to the ESP32. |
| **144** | $8.64\text{ A}$ | $43.2\text{ W}$ | $\approx 2.71\text{ A}$ | Must use an external $5\text{ V} \ge 10\text{ A}$ supply with **power injection** at both ends of the strip to prevent voltage droop and color shift. |

### Conclusion
The prototype reliably controlled individual pixels and cleared the strip via serial commands. Comparing red and white current draw demonstrated how active sub-channels scale consumption, while comparing measured (`80 / 255`) and worst-case (`255 / 255`) budgets showed why firmware luminance limits, wire gauge, and controller overhead must all be factored into power supply selection.

---

## Study Log & Portfolio Evidence (Tasks 4 & 5)

### Task Summary Log
* **Task 1 (`task1-led/`):** Built a PWM fade loop on `GPIO 16` (`analogWrite`) and verified that varying the duty cycle smoothly modulates average LED current and perceived brightness.
* **Task 2 (`task2-rgb/`):** Scaffolded a common-cathode RGB serial REPL (`GPIO 16/17/18`), verified serial validation (applying valid triples like `255 0 0` and rejecting malformed inputs like `abc` with `ERROR: Enter three integers, e.g. 255 0 0`), and measured channel currents across the onboard $150\ \Omega$ resistors at `PWM = 255`: **Red =** $7.13\text{ mA}$, **Green =** $4.20\text{ mA}$, **Blue =** $3.33\text{ mA}$.
* **Task 3 (`task3-strip/`):** Implemented an 8-pixel WS2812B serial controller (`index R G B` and `clear`) on `GPIO 16` using `NeoPixelBusLg` (`NeoEsp32Rmt0Ws2812xMethod`, `SetLuminance(80)`). Measured strip current on the $5\text{ V}$ line was $\approx 0.03\text{ A}$ **(all red)** and $\approx 0.09\text{ A}$ **(all white)**.

### AI Scaffolding & Hardware Verification Note
* **What the AI agent got wrong:** During Task 3 scaffolding, the agent initially instantiated the base `NeoPixelBus` class while calling `strip.SetLuminance(80)`, which caused a compilation error because luminance scaling is only implemented in `NeoPixelBusLg`.
* **Fix & verification:** We updated the firmware to instantiate `NeoPixelBusLg<NeoGrbFeature, NeoEsp32Rmt0Ws2812xMethod, NeoGammaNullMethod>`, verified clean compilation with `pio run -t compiledb && pio run`, and validated pinouts (`GPIO 16`), color order (`GRB`), and current draw directly on hardware with a multimeter.

### Deeper Question: Addressable Strip vs. Single Indicator LED
* **Power budget:** A single indicator LED draws $\sim 3\text{ mA}$ to $7\text{ mA}$ from a GPIO pin, well within microcontroller limits. An addressable strip places dozens or hundreds of RGB driver ICs in parallel across the $5\text{ V}$ rail (up to $60\text{ mA}$ per pixel), easily exceeding USB port ($500\text{ mA}$) and onboard trace ratings unless budgeted and externally powered.
* **Data-level & timing considerations:** A single LED is driven by a simple static or low-frequency PWM voltage level. In contrast, WS2812B pixels decode a strict $800\text{ kHz}$ single-wire NRZ serial protocol (requiring hardware RMT/I2S offloading on the ESP32 to survive radio interrupts) and expect a logic-high threshold around $0.7 \times V_{DD}$ ($\approx 3.5\text{ V}$ when powered at $5\text{ V}$), making $3.3\text{ V}$ data signal integrity, short leads, and series damping resistors critical on longer runs.
