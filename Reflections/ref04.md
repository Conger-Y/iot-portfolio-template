# Reflection 04 — Liquid Level Measurement Across Multiple Fluids

## What Was Difficult & Unexpected Failure Modes
Testing four sensing principles across freshwater, seawater, diesel, and simulated wastewater revealed how strongly fluid properties can break a sensor that works fine on a dry bench:
* **Electrode polarization & drift:** Even when powering the `HD-38` resistive needle probe in pulses via `GPIO25` to reduce electrolysis, holding a constant `100%` freshwater level still caused the ADC reading to drift downward by `67%` (`711` down to `238`). In seawater, high ionic conductivity saturated the probe immediately and accelerated trace oxidation.
* **Optical transparency & ultrasonic dead-zones:** In clear water, the `VL53L0X` ToF laser passed straight through the liquid or refracted away, producing `17 of 19` invalid outputs at full depth. Meanwhile, the ultrasonic sensor hit its near-field dead-zone around `18–20 cm` when the cup was full, and in the diesel test an unguided cork float tilted against the tank wall (`6.63 cm` wide spread) after refilling.

## Fun Exploration & Counter-Intuitive Results
Two experiments turned initial setbacks into memorable insights:
* **Turbidity helping optical sensing:** While the `VL53L0X` laser failed in clear water, switching to opaque, turbid simulated wastewater caused suspended particles to scatter the infrared beam cleanly at the air-liquid interface—yielding `19/19` valid readings with a tight `7 mm` span at full depth.
* **Vessel geometry & outlier filtering:** Weighing a tapered cup on the `HX711` load cell proved that half liquid depth corresponded to only `40.3%` of full volume (`153.6 g` vs. `381.0 g`) because the cup widens toward the top, and using the median (`388.95 g`) cleanly rejected a mechanical placement outlier (`353.3 g`). Simulating tank rocking (`simulate_rocking.py`) also visually demonstrated why sloshing waves scatter direct ultrasonic echoes.

## Take-Away, Advice & Pod Collaboration
1. **There is no universal level sensor:** Always match the sensing physics to the fluid—never use conductive probes in non-conductive hydrocarbons (diesel) or bio-fouling wastewater, and never rely on optical ToF in transparent liquids.
2. **Design mechanical damping into the system:** For marine or mobile tanks subject to sloshing, mount non-contact ultrasonic/radar sensors above their minimum dead-zone inside a perforated stilling pipe (baffle tube), or use external load cells when zero fluid contact is required.
3. **Pod teamwork:** Splitting data logging, Python curve-fitting/rocking simulations, and physical test runs across four fluids allowed us to cross-compare all four modalities systematically within one session.
