# Reflection 02 — Buses & I²C: Multi-Device Communication & Logic Analyzer

## What Was Good
It was very satisfying to see the I²C Master-Slave architecture work cleanly in practice. First, reading raw bytes directly from the LM75 temperature register (`0x00`) via the native `Wire` library (`22.969 °C`) matched the third-party library output (`22.516 °C`), and warming the chip with my fingers produced an immediate, smooth climb to `26.266 °C`. Second, wiring both the LM75 sensor (`0x4F`) and the I²C LCD (`0x27`) in parallel on the exact same `SDA` (`GPIO 21`) and `SCL` (`GPIO 22`) pins worked without address collisions, letting the ESP32 read temperature and display it live on the screen over just two data lines.

## Show Stoppers & Fun Debugging Exploration
During Task 9 ("Build and Choose Our Scope"), we hit a complete show stopper: the Blick logic analyzer board refused to power on at all once wired to the ESP32 on the breadboard. Tracing every jumper wire carefully revealed an unintended wire bridging the `3V3` supply rail directly to pin `D19`, creating a direct power short. Removing that single misplaced jumper brought the board back to life immediately, and seeing real digital bus waveforms appear on `hepter/web-logic-analyzer` made the hardware debugging effort pay off.

## Take-Away & Advice
1. **Inspect power rails first:** When a board fails to power up after breadboard changes, check for accidental `3V3`-to-GND or `3V3`-to-GPIO shorts before suspecting software or broken hardware.
2. **Look beneath libraries:** Reading a sensor's raw register once verifies byte order and scaling factors, turning third-party libraries from black boxes into tools you can actually debug.
