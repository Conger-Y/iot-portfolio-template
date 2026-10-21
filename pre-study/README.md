# Pre-Study — Sensors and Actuators (Sensorik und Aktorik)

**Name:** Conger-Y
**Course:** Sensors and Actuators, WS 2026/27, HSBI Gütersloh


---

## Overview

This is my Module 0 (pre-study) portfolio entry. It contains my notes, answers to
Guiding Questions 1–8, the Wokwi mini-exercise evidence, and my datasheet notes.

---

## Wokwi Mini-Exercise

A potentiometer is connected to an analog input pin. The code reads the analog
value, prints it over Serial, and maps it to the LED brightness, so turning the
potentiometer changes the LED.

- Potentiometer → analog input, value printed via Serial, mapped to LED brightness.
- ADC resolution tested at: 8 / 10 / 12 bits
- Observations:

<img width="416" height="281" alt="image" src="https://github.com/user-attachments/assets/e0c3a2a7-2782-4c6c-b047-83603ed27a9d" />
<img width="415" height="220" alt="image" src="https://github.com/user-attachments/assets/f71ba0cd-80b7-40f9-904a-466fdd129e48" />
<img width="415" height="214" alt="image" src="https://github.com/user-attachments/assets/a4789363-0e2c-4d1f-8fdf-601eee458dd2" />
<img width="415" height="238" alt="image" src="https://github.com/user-attachments/assets/81a81d08-fe06-4962-832e-45910fa0f3d3" />

ADC resolution test:
- 8-bit  → raw range 0..255   (256 steps)
- 10-bit → raw range 0..1023  (1024 steps)
- 12-bit → raw range 0..4095  (4096 steps)

Observation: changing the resolution changed only the number of discrete steps
(the resolution), not the accuracy. When the `map()` upper bound no longer matched
the actual raw range, the output "jumped" or stopped reaching full brightness
---

## Datasheet Notes

### VL53L0X (ToF distance sensor)
- **Physical quantity & principle:** Absolute distance. Time-of-Flight (ToF) — a
  940 nm VCSEL laser fires invisible light pulses and the sensor times how long the
  photons take to return from the target (using a SPAD array). The reported range is
  independent of the target's colour or reflectance.
- **Supply voltage:** 2.6–3.5 V for the bare chip; breakout modules work from about
  2.6–5.5 V thanks to an onboard regulator.
- **Interface / I²C address:** I²C (plus an XSHUT shutdown pin and a GPIO1 interrupt
  pin). The 7-bit address is 0x29 (written as 0x52 in 8-bit form). It is programmable,
  but it resets to 0x29 on every power cycle.
- **Range / resolution / accuracy / response time:** About 30 mm to 2000 mm; accuracy
  roughly ±3%; response time under 30 ms. A configurable "timing budget" trades speed
  for longer range and better accuracy.
- **Temperature dependence:** Operating range about −20 to +70 °C.
- **Conditions / limitations:** Accuracy drops noticeably beyond ~1.5 m; readings are
  noisy on highly reflective surfaces; two identical sensors clash on the bus (same
  default 0x29) and must be re-addressed one by one using the XSHUT pin.

### MPR121 (capacitive touch sensor)
- **Physical quantity & principle:** Touch / proximity. Capacitive sensing — a finger
  near an electrode changes its capacitance; the chip charges the electrode with a
  constant current and measures the change in the time constant to detect a touch.
- **Supply voltage:** 1.71–3.6 V (modules without a regulator need 2.5–3.6 V). It is a
  3.3 V device — do not exceed 3.6 V.
- **Interface / I²C address:** I²C (plus an IRQ interrupt output). Default address 0x5A;
  it can be set to 0x5A, 0x5B, 0x5C or 0x5D through the ADDR pin, so up to 4 devices can
  share one bus.
- **Channels / response:** 12 electrode inputs plus 1 virtual channel for proximity
  detection; a configurable sample period (e.g. 16 ms) with touch/release thresholds
  and debounce.
- **Power:** 29 µA at a 16 ms sampling period; 3 µA in stop mode.
- **Conditions / limitations:** No onboard regulator (watch the 3.6 V limit); electrode
  wire length and material affect how easily a touch triggers, so thresholds must be
  tuned; 8 of the pins are shared as LED driver / GPIO.

---

## Measurement and Uncertainty — my understanding (NTC example)

1. **True value, error, uncertainty.** The true value is the real, objective physical
quantity — it is fixed. But when we measure it with an instrument, there is always some
error. For example, an NTC thermistor reading might be off by 0.3 °C, or maybe 0.5 °C —
we don't know exactly. So we use a range to include this error. Any temperature inside
this range can be trusted, and this range is the uncertainty.

2. **Accuracy vs. precision.** Accuracy is how far a measurement is from the target
(the true value), so it means how close each reading is to the true value. Precision
looks at a set of readings and asks how close they are to each other — very close means
high precision. High precision does not mean high accuracy: the readings can be stable
but still wrong.

3. **Resolution vs. accuracy.** Resolution is the scale, the smallest step the instrument
can tell apart. It does not care whether the result is correct — it is just how finely you
can observe. Accuracy is how big the gap is between the measured result and the real value.
A high resolution does not make the accuracy better — for example, the NTC's own self-heating
adds error, so you can never truly reach the physical limit.

4. **Systematic vs. random error.** Systematic error: if the NTC reads 0.5 °C too high every
time, the same fixed amount is added each time; it comes from the device itself and can be
fixed by calibration against a known temperature. Random error: if I read the same temperature
and get a slightly different result each time, going up and down, that is random error — it can
be smoothed by taking the average of several readings.

5. **Sampling and quantization.** Sampling means setting a time interval and reading the data
once every interval. Quantization means putting these continuous values into pre-defined boxes.
For example, if the NTC reads 2.5987 °C, there is only a box close to this number, not one
exactly equal to it — the difference that results is the quantization error.

---

## Actuators Primer — my understanding

**PWM / duty cycle.** PWM is like a switch that controls brightness using the pin, adjusting how
bright the LED or LED strip is. The duty cycle explains how: a higher duty-cycle ratio is brighter,
a lower ratio is dimmer. More generally, PWM simulates an in-between value by switching on and off
very fast, so it can also set a servo angle.

**LED current-limiting resistor.** An LED is a fragile component that cannot take a large current.
Once the voltage goes past the LED's forward (turn-on) voltage, the current rises very steeply, and
too much current burns it out. A current-limiting resistor holds the current at a safe value and
protects both the LED and the pin.

**Addressable LEDs (WS2812 / NeoPixel) vs plain RGB.** Addressable LEDs are a form of integration.
A plain RGB LED is controlled one by one. An addressable LED has a small built-in chip; all the LEDs
are connected in one chain on a single data line, and a data signal tells each chip what colour to
show. Once the number of LEDs gets large, a single pin cannot supply enough current, so a separate
power supply is needed.

**Motors: servo / DC / stepper.** A servo uses the PWM pulse width to control a precise angle (0–180°).
A DC motor just takes current and keeps spinning; an H-bridge driver controls its direction and speed,
but it has no idea where it is. A stepper motor uses a dedicated driver board to turn step by step, so
it can position precisely and still rotate continuously.

**Why motors and relays need a driver.** Motors and relays need large voltage and current that a pin
cannot provide. A driver sits in between and amplifies the current, so the pin is not damaged and the
motor still runs. The driver also isolates the microcontroller and blocks the reverse voltage spike when
a motor or relay switches off.

---

```mermaid
flowchart TD
    A["Physical quantity: body weight (force / gravity)"]
    A -->|"the load bends a metal beam"| B["Sensor (transducer): strain gauge"]
    B -->|"the strain gauge deforms, and its resistance changes"| C["Signal conditioning:<br>(amplifier, divider, filter, reference)"]
    C -->|"a bridge / voltage-divider circuit turns the resistance change into a voltage change"| D["ADC:<br>(sample, filter, calibrate)"]
    D -->|"converts the analog voltage into a digital value"| E["MCU:<br>(MQTT, Node-RED, dashboard)"]
    E -->|"uses a calibration curve to convert the digital value into kilograms"| F["Output:<br>the LCD shows 70 kg / a buzzer beeps"]
    E -->|"command (PWM / driver)"| G["System:<br>(acts on the world)"]

Physical quantity: body weight (force / gravity)
→  — the load bends a metal beam, the strain gauge deforms, and its resistance changes.
→ a bridge / voltage-divider circuit turns the resistance change into a voltage change.
→ converts the analog voltage into a digital value.
 uses a calibration curve to convert the digital value into kilograms.
 the LCD shows 70 kg / a buzzer beeps.

### 2. Wokwi observation
The full analog range had 4096 discrete steps at 12-bit resolution. After changing the ADC resolution
to 8-bit, it had 256 steps (10-bit → 1024). What changed was the resolution (the number of steps), not
the accuracy — a finer scale does not make the reading more correct.

### 3. Datasheet
VL53L0X: interface is I²C; 7-bit address 0x29 (0x52 in 8-bit form), programmable but resets to 0x29 on
power-up; stated accuracy about ±3%, with the condition that it degrades beyond ~1.5 m and on highly
reflective surfaces.

### 4. Accuracy vs. precision
- Precise but not accurate: a kitchen scale that always reads 50 g too high — the readings are very
  consistent, but consistently wrong.
- Accurate but not precise: an old thermometer whose readings scatter around the true temperature —
  each reading jitters, but the average is right.

### 5. I²C anticipation
On an I²C bus, all devices share the same clock line (SCL) and data line (SDA). This can cause a
communication conflict: when the microcontroller (master) sends a signal, every device receives it,
but the signal is meant for one specific device, so if several respond at once the data collides. The
address solves this — each device has its own address, the master calls an address, and only the matching
device responds while the others stay quiet. The address is basically an ID card on the shared bus. Real
example: two VL53L0X sensors both default to 0x29, so they clash on one bus and must be re-addressed one
by one via the XSHUT pin; an MPR121 can instead be set to 0x5A/0x5B/0x5C/0x5D via its ADDR pin, so four
can share one bus.

### 6. Actuator choice
Task: move a small door latch once every few minutes. **I would choose a servo.** A latch only needs to
turn to a position and back; a servo has built-in angle control, needs just one PWM line, and uses very
little power when it moves only occasionally.
- Stepper: could do it and position precisely, but it needs a dedicated driver board and more wiring — too much for such a simple task.
- DC motor: not suitable — it only spins, with no position feedback, so it cannot stop reliably at the open/closed position.
Justification: **torque** — a small latch needs little torque, a servo is enough; **control** — a servo's angle control is the simplest; **power** — it moves only occasionally, so average power is very low.

### 7. LED power math
A WS2812 strip has 30 LEDs, each drawing up to ~60 mA at full white.
- Total current = 30 × 60 mA = 1800 mA = **1.8 A**.
- The microcontroller's 5 V pin cannot supply this. A pin can only output a few tens to a couple of
  hundred mA, and 1.8 A is many times more — it would burn the pin. So the strip must be powered from a
  separate 5 V supply, while the microcontroller only provides the data line.

### 8. Project abstract — Directional Approach Alert
The idea started from playing PUBG, where I kept getting shot from a direction I could not see. The real
need behind it: when I wear headphones and focus on a game or on work, I cannot see or hear someone
approaching from behind or from the side. This project is a device that senses when a person approaches,
works out roughly which direction they come from, and gives a directional alert even while I have
headphones on.

How it works: several distance / presence sensors are placed around my seat, each facing a different
direction (behind, left, right). When one detects someone approaching, the system decides the rough
direction — an estimate, not an exact angle.

Output (actuator): a vibration motor on the mouse pad or seat buzzes on the matching side, so I feel which
direction the person is coming from. Optionally an addressable LED also shows the direction by colour.

Sensors and actuator I would start with:
- Sensor 1: a VL53L0X ToF distance sensor (I²C) to detect an approaching person.
- Sensor 2: a second distance or PIR presence sensor facing another direction.
- Actuator: a small vibration motor for the directional haptic alert.

This is a first direction and may change during the course.
