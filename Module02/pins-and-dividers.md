<img width="1280" height="800" alt="0e9ce468300ff6462e156a4fc02d0c00" src="https://github.com/user-attachments/assets/2c3bbbed-cb50-4212-9634-f9d3534b3b49" />** Task 1: Simple pins, PWM, UART, and a voltage divider


 Task 9
 <img width="2049" height="1280" alt="a55c5f8cf0187de61daf92e4053af284" src="https://github.com/user-attachments/assets/95a85e80-a0db-4131-bc37-97670243196f" />
原先一直困扰我们为什么blick用杜邦线接到eps32后blick板不亮，后面发现在面包板上3v3和D19用一根杜邦线连着导致blick短路，当这根线被撤掉，图像成功显示


 Task 3



Task 5
 
 已经通过找到I²C地址
 <img width="2049" height="1280" alt="1b1839e628146707adc860389781dbc3" src="https://github.com/user-attachments/assets/4665ac84-e277-4892-85a4-0f31b9183414" />
Hardware Connection: The I²C sensor is connected to the ESP32 development board using jumper wires. The 5V and GND pins of the sensor are connected to the 3V3 and GND pins of the ESP32, respectively, to provide power supply. The SCL and SDA pins of the sensor are connected to the GPIO 22 and GPIO 21 pins of the ESP32, respectively, establishing the I²C data communication channel. All hardware components are mounted on a breadboard.

Software Configuration and Execution: The I²C scanner program sketch_oct30a.ino was written and uploaded via the Arduino IDE. The program initializes the serial communication (at a baud rate of 115200) and the I²C bus (specifying pins 21 and 22).

Test Results: After uploading the code, the Serial Monitor was opened. The system successfully printed the startup message and began scanning the I²C bus. The scan results indicated that one I²C device was successfully detected at the address 0x4F. This confirms that the hardware wiring is correct and the ESP32 is able to successfully communicate with the sensor via the I²C protocol.

Task 6
Register
<img width="1280" height="800" alt="d9ffb1252003348cba3e53a42e9509d0" src="https://github.com/user-attachments/assets/b92d0430-e85a-4e0a-b009-edd2e5593da7" />


To understand the underlying principles without relying on third-party libraries, I directly used the native Wire library to initiate communication with the sensor at address 0x4F and located the register storing the temperature data (address 0x00).

Reading Raw Data: Successfully read 2 bytes of raw hexadecimal data, for instance, a high byte of 0x16 and a low byte of 0xF8.

Data Reconstruction Process: According to the data structure instructions in the LM75 datasheet, I performed bit concatenation and bit-shifting operations on the two read raw bytes. Then, by applying the resolution/scaling factor provided in the datasheet, I manually calculated the actual physical temperature (e.g., resulting in 22.969 °C).

Library
<img width="1280" height="800" alt="7238bdef095a67c00386a94fca33ff06" src="https://github.com/user-attachments/assets/ee430748-4a05-4dc2-9473-e4df3b899582" />
I used a dedicated third-party library for the LM75 sensor. By initializing the sensor's I²C address to 0x4F and calling the Temp0.getTemperature() function, the processed temperature data was directly output.

Physical Verification: During the program's execution, I intentionally pressed the LM75 sensor with my fingers to physically apply heat. Observing the Serial Monitor, the temperature readings responded sensitively and immediately, with the data gradually climbing from a room temperature of 22.516 °C to 26.266 °C.

Comparison & Conclusion
<img width="1280" height="800" alt="0e9ce468300ff6462e156a4fc02d0c00" src="https://github.com/user-attachments/assets/9d8cec8a-559a-4dfa-b79f-4550595eb177" />
Comparing the "temperature value calculated by manually parsing the register's raw bytes (e.g., 22.969 °C)" with the "temperature value directly output using the LM75 library (e.g., 22.516 °C)", both the numerical range and the changing trend are perfectly consistent.
Conclusion: This proves that my understanding of this I²C sensor's data type, byte order, and scaling ratio is completely correct.


Task 7
