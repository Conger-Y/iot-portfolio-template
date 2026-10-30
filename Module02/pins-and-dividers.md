** Task 1: Simple pins, PWM, UART, and a voltage divider


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
