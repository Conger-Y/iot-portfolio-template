# Reflection 1 in Module 1


## What Was Difficult & Interaction with Instructors
At the start of Task 2, my partner and I got stuck trying to establish a basic connection between the ESP32 and the computer, which blocked us from flashing code or viewing serial output. Reaching out to our professor quickly resolved the bottleneck: with his guidance, we installed the missing USB-to-UART drivers/plugins and got the toolchain running smoothly. It was a good reminder to verify driver and port recognition early before assuming the board or code is faulty.

## Take-Away & Advice
My biggest take-away from Task 3 ("The ADC Precision Dance") is that **higher resolution does not equal higher accuracy**. Switching `analogReadResolution()` from `10-bit` (`1024` steps) to `12-bit` (`4096` steps) quadrupled the theoretical step precision from `3.223 mV` down to `0.806 mV`, yet cross-checking against a multimeter revealed that the classic ESP32 ADC has a dead zone below `0.15 V` (always reading `0`) and saturates prematurely above `2.90 V – 3.15 V` (clipping at `4095`).  
**Advice:** Keep analog sensor signals within the `0.15 V – 2.90 V` linear window whenever possible, and never rely on raw `ADC * 3.3 / 4095` math near the supply rails without calibration.

## Interaction with Peers & Extra Exploration
My partner and I split the documentation workload effectively—my partner handled and hosted the Task 2 serial visualization write-up in our shared portfolio while I documented the breadboard, ADC, and LDR characterization tasks. We also used AI prompts step-by-step to verify breadboard bus layouts for pull-up/pull-down button wiring and tested our own hypothesis on the LDR voltage divider, watching the ADC reading drop instantaneously when uncovering the photoresistor.
