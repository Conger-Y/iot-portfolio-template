# Reflection 03 — Sensor Characterization, Calibration & Uncertainty

## What Was Difficult & Where We Struggled
Controlling physical test conditions and wiring accuracy proved much harder than writing the firmware:
* **Wiring alignment:** During Task 3b, our multimeter read nothing until we discovered the `3.3 V` red jumper wire was shifted by a single breadboard row into an empty socket.
* **Environmental interference:** In unshielded LDR distance sweeps, ambient room daylight created an illuminance floor that flattened resistance readings at larger lamp distances, making a simple distance-based proxy unreliable for true lux calibration.
* **Incomplete boundary check:** In the DS18B20 ice-bath trial (`8.7563 °C` mean), we ran out of ice before verifying the bath temperature right at the probe, leaving the `0 °C` calibration check incomplete.

## Fun Exploration & Quantitative Surprises
Several experiments turned abstract metrology terms into concrete numbers:
* **Mechanical repeatability:** Holding a potentiometer shaft still gave a `1 mV` spread, whereas turning it away and returning it to a visual mark 10 times widened the spread to `124 mV`—showing how human alignment and mechanical play dominate manual adjustments.
* **Filter trade-offs:** Injecting a synthetic `+500` count spike into 500 LDR samples showed that a 5-sample median filter completely erased the outlier (`0` deviation, `~43 ms` lag), whereas a 5-sample moving average smeared the spike across five consecutive readings and an EMA (`alpha = 0.2`) added `~85 ms` of phase lag.

## Take-Away, Feedback & Quality Discipline
1. **Never apply unverified offsets:** Even though our DS18B20 read `+1.43 °C` higher than a room reference thermometer, we intentionally left the firmware uncorrected because the reference thermometer itself was uncalibrated.
2. **Use hysteresis for threshold decisions:** Adding a `2.0 °C` to `2.5 °C` hysteresis band on a frost alarm cut false state switches from 6 down to 2 without delaying the initial trigger.
3. **Let the uncertainty budget guide improvements:** Combining error sources for our calibrated potentiometer (`1.472 ± 0.035 V`, $k = 2$) showed that calibration model discrepancy (`16.2 mV`) accounted for over `85%` of total variance—proving that adding more calibration points across the range matters far more than extra software filtering.
