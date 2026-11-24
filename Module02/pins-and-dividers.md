# Module 2

## 🎯 Learning Goals

Through hands-on I²C bus wiring, low-level register parsing, multi-device bus sharing, and logic analyzer debugging across Tasks 5–7 and Task 9 on the ESP32, this module achieves the following learning outcomes:

1. **I²C Bus Wiring & Address Scanning (Task 5):** Configure the ESP32 I²C hardware interface (`SDA = GPIO 21`, `SCL = GPIO 22`) on a breadboard, initialize serial diagnostics (`115200` baud), and execute an I²C bus scanner to verify physical connections and discover slave device addresses (`0x4F`).
2. **Low-Level Register Parsing vs. High-Level Libraries (Task 6):**
   * Communicate directly with the LM75 temperature sensor (`0x4F`) using the native Arduino `Wire` library to read raw bytes from pointer register `0x00`, manually handling byte order and scaling factors.
   * Cross-validate manual register calculations ($22.969\ ^\circ\text{C}$) against a dedicated LM75 library (`Temp0.getTemperature()`, $22.516\ ^\circ\text{C}$) and confirm dynamic sensor response ($22.516\ ^\circ\text{C} \rightarrow 26.266\ ^\circ\text{C}$) through physical heat application.
3. **Multi-Device Bus Sharing & Master-Slave Addressing (Task 7):**
   * Wire multiple I²C slave devices—the LM75 sensor (`0x4F`) and an I²C LCD display (`0x27`)—in parallel onto the same shared `SDA` and `SCL` lines without using extra GPIO pins.
   * Apply Master-Slave addressing principles to read real-time temperature data from one slave and display it on another slave across a single shared bus without data collisions.
4. **Digital Waveform Capture & Hardware Fault Debugging (Task 9):**
   * Deploy and operate a web-based logic analyzer (`hepter/web-logic-analyzer`) to inspect digital bus signals alongside basic pin, PWM, UART, and voltage-divider behavior.
   * Systematically diagnose and resolve breadboard power-distribution faults by identifying and clearing a `3V3`-to-`D19` short circuit that prevented the logic analyzer board from powering up.




## Task 5：Wire the Bus and Scan It
 

 <img width="2049" height="1280" alt="1b1839e628146707adc860389781dbc3" src="https://github.com/user-attachments/assets/4665ac84-e277-4892-85a4-0f31b9183414" />
 
Hardware Connection: The I²C sensor is connected to the ESP32 development board using jumper wires. The 5V and GND pins of the sensor are connected to the 3V3 and GND pins of the ESP32, respectively, to provide power supply. The SCL and SDA pins of the sensor are connected to the GPIO 22 and GPIO 21 pins of the ESP32, respectively, establishing the I²C data communication channel. All hardware components are mounted on a breadboard.

Software Configuration and Execution: The I²C scanner program sketch_oct30a.ino was written and uploaded via the Arduino IDE. The program initializes the serial communication (at a baud rate of 115200) and the I²C bus (specifying pins 21 and 22).

Test Results: After uploading the code, the Serial Monitor was opened. The system successfully printed the startup message and began scanning the I²C bus. The scan results indicated that one I²C device was successfully detected at the address 0x4F. This confirms that the hardware wiring is correct and the ESP32 is able to successfully communicate with the sensor via the I²C protocol.
#### Source code is available [[here](https://github.com/liusihan54/iot-portfolio-templateSihan/tree/main/Module02)](URL)
## Task 6: Read Through the Library, Then Read the Register
Register
<img width="1280" height="800" alt="d9ffb1252003348cba3e53a42e9509d0" src="https://github.com/user-attachments/assets/b92d0430-e85a-4e0a-b009-edd2e5593da7" />

To understand the underlying principles without relying on third-party libraries, an AI-generated code was executed. This code directly utilizes the native Wire library to initiate communication with the sensor at address 0x4F and locates the register storing the temperature data (address 0x00).

Library
<img width="1280" height="800" alt="7238bdef095a67c00386a94fca33ff06" src="https://github.com/user-attachments/assets/ee430748-4a05-4dc2-9473-e4df3b899582" />
I used a dedicated third-party library for the LM75 sensor. By initializing the sensor's I²C address to 0x4F and calling the Temp0.getTemperature() function, the processed temperature data was directly output.

Physical Verification: During the program's execution, I intentionally pressed the LM75 sensor with my fingers to physically apply heat. Observing the Serial Monitor, the temperature readings responded sensitively and immediately, with the data gradually climbing from a room temperature of 22.516 °C to 26.266 °C.

Comparison & Conclusion
<img width="1280" height="800" alt="0e9ce468300ff6462e156a4fc02d0c00" src="https://github.com/user-attachments/assets/9d8cec8a-559a-4dfa-b79f-4550595eb177" />
Comparing the "temperature value calculated by manually parsing the register's raw bytes (e.g., 22.969 °C)" with the "temperature value directly output using the LM75 library (e.g., 22.516 °C)", both the numerical range and the changing trend are perfectly consistent.
Conclusion: This proves that my understanding of this I²C sensor's data type, byte order, and scaling ratio is completely correct.

#### Source code is available [[here](https://github.com/liusihan54/iot-portfolio-templateSihan/tree/main/Module02)](URL)
## Task 7 : Two Devices on One Bus


<img width="1280" height="800" alt="ae3e69ed8e2ddc04dec56448c1f0b7b1" src="https://github.com/user-attachments/assets/475ad011-a635-44ca-b0a1-39a60b6aef29" />

A second I²C device was added to the existing bus. The SDA and SCL pins of both devices were connected in parallel to the same SCL (GPIO 22) and SDA (GPIO 21) pins on the ESP32 without using any extra data pins. After powering both devices and running the I²C scanner, the Serial Monitor successfully detected two distinct addresses simultaneously: 0x27 and 0x4F. This confirms that both physical devices are successfully mounted on the same bus.


I²C uses a Master-Slave architecture. The ESP32 acts as the master, and the sensors act as slaves. Each slave device on the bus has a unique hardware address (e.g., 0x27 and 0x4F). When the master initiates communication, it broadcasts the target device's address over the bus. Only the device with the matching address will respond and process the data, while all other devices ignore the communication and remain silent. This addressing mechanism ensures no data collision occurs on the shared SCL and SDA lines.
<img width="2304" height="4096" alt="050ac5a3816e0619b81b4d9b5c0e759e" src="https://github.com/user-attachments/assets/a4ada9d4-bedd-48a3-8b79-e5197a3514d8" />

Operation and Observation: An AI-written comprehensive Arduino sketch was executed, which utilizes the libraries for both the temperature sensor (0x4F) and the LCD display (0x27). The program successfully reads the temperature data from the LM75 and transmits this data in real-time to be displayed on the LCD screen, using the exact same pair of I²C pins (SDA=21 / SCL=22)







<img width="1280" height="800" alt="0e9ce468300ff6462e156a4fc02d0c00" src="https://github.com/user-attachments/assets/2c3bbbed-cb50-4212-9634-f9d3534b3b49" />** Task 1: Simple pins, PWM, UART, and a voltage divider

#### Source code is available [[here](https://github.com/liusihan54/iot-portfolio-templateSihan/tree/main/Module02)](URL)
## Task 9 : Build and Choose Our Scope — instrument sprint

我们通过 candidate： hepter/web-logic-analyzer 成功将图形显示出来
 <img width="2049" height="1280" alt="a55c5f8cf0187de61daf92e4053af284" src="https://github.com/user-attachments/assets/95a85e80-a0db-4131-bc37-97670243196f" />

#### Source code is available [[here](https://github.com/liusihan54/iot-portfolio-templateSihan/tree/main/Module02)](URL)



