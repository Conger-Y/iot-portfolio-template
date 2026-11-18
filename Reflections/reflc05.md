# Reflection 05 — LEDs & Light as Output

## What Was Good & Hands-On Exploration
Seeing textbook diode physics and power calculations match real multimeter numbers on the bench was the highlight of this session:
* **RGB forward-voltage physics:** Measuring voltage drops directly across the onboard `150 Ω` SMD resistors (`151`) at `PWM = 255` showed Red drawing `7.13 mA` (`1.07 V`), Green `4.20 mA` (`0.63 V`), and Blue `3.33 mA` (`0.50 V`). Seeing Red draw more than twice the current of Blue made the impact of different LED forward voltages ($V_f$) immediately concrete.
* **Addressable strip scaling:** Driving the 8-pixel WS2812B strip via ESP32 hardware RMT Channel 0 (`NeoEsp32Rmt0Ws2812xMethod`) kept timing rock-solid, and measuring series current on the `5 V` line confirmed that all-white illumination (`~0.09 A` / `0.45 W`) draws three times the current of all-red (`~0.03 A` / `0.15 W`) at `80 / 255` luminance.

## Show Stoppers & AI Code Debugging
Using PlatformIO with Aider (`gemini-3-flash-preview`) accelerated project scaffolding across `task1-led`, `task2-rgb`, and `task3-strip`, but it also produced a classic AI API hallucination:
* **What the AI got wrong:** In Task 3, the initial generated code instantiated the base `NeoPixelBus` class while calling `strip.SetLuminance(80)`, triggering a C++ compilation failure because luminance scaling only exists in `NeoPixelBusLg`.
* **How we fixed it:** Feeding the PlatformIO compiler error back into Aider and specifying `NeoPixelBusLg<NeoGrbFeature, NeoEsp32Rmt0Ws2812xMethod, NeoGammaNullMethod>` resolved the build cleanly. We also hit a brief `COM` port drop after rewiring `GPIO 16` from Task 2 to Task 3, which was fixed by checking power rails and re-seating the USB cable.

## Take-Away, Advice & Pod Collaboration
1. **Cap luminance in firmware before plugging into USB:** Setting `SetLuminance(80)` kept our 8-LED strip safely around `90–150 mA`, whereas scaling up to `60` or `144` pixels at full white (`60 mA/pixel`, up to `8.64 A`) requires an external `5 V` supply and dual-end power injection.
2. **Verify AI-generated hardware code:** Treat LLM output as a fast first draft—always check library class names, color byte order (`GRB`), and serial input validation (such as rejecting `abc` on the RGB REPL or out-of-bounds indices on the strip).
3. **Pod collaboration:** Working together to hold multimeter probes on tiny SMD resistors while capturing serial logs and video evidence made our measurements much cleaner and easier to verify.
