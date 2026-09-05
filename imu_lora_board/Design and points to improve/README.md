# Design and points to improve

## 1. Design considerations
### Power and protection
9V is supplied via the 4 pin TE connectivity 1379165-7 power header. Two pins are used each for 9V and ground so as to distribute current and reduce heating issues. The 9V line is passed through a fuse for surge protection, TVS diode for transient voltage spikes such as in the case of sudden electrostatic discharge, and a Schottkey diode for reverse polarity protection. The output from the latter is passed through a buck converter for a clean 3.3V bus to power the board's components. The buck converter was designed using Texas Instruments' WeBench Power Designer platform. 

### Microcontroller unit (MCU)
The STM32F301K6 was chosen for its limited footprint, support for an external high speed oscillator (the 8 MHz WE-XTAL_12SMX in this case) and both UART and I2C for communication with peripherals. The pins for serial wire debug are routed to pads for a tag connect header for debugging purposes. 

### Inertial measurement unit (IMU) and shock sensor
The popular MPU-6050 was chosen for its simple setup and I2C communication. Contains a 3 axis gyroscope and accelerometer for up to 16g. The TE connectivity 1005940-1 piezo film sensor was used for it's high sensitivity to detect sudden jolts or shocks to the board. The analog output is passed through a low pass filter to reduce unwanted noise, connecting thereafter to the MCU's analog to digital converter pin. 

### RF module and antenna setup
The Microchip RN2903 was chosen as the RF transceiver module to output data from the sensors as it passes through the MCU. The RN2903 requires a simple dual UART point to point connection, and outputs the appropriate signal at 915 MHz, using the LoRa standard, to a chip antenna (Yageo ANT1204F005R0915A). This simplified the design as this "smart" transceiver has an onboard MCU and handles all the RF calculations and modulation so extra routing or additional MCU memory is not needed.  

### General routing and board design. 
By bundling all the power components around the connector, a tight layout could be formed for the large ground polygon and for transferring 9V effectively through the different protection stages and converter. Component placement ensured a large 3.3V polygon could be created which routed as near as possible to components which needed 3.3V, extending across 2 and a half sides of the board. A 2 layer stackup was determined to be sufficient and routing as much as possible on the upper layer to have as much ground polygon as possible on the lower layer was seen as an interesting challenge. The big gap in the ground polygon is probably visible in the bottom right corner; this is required for the chip antenna to effectively transmit it's signal. A couple of traces could be seen as potentially problematic, especially the long trace extending from the bottom of the MCU to the right side of the transceiver IC. However, this was regarded a tradeoff worth making; it passed under a 3.3V polygon, which is an acceptable compromise, and the traces related to serial wire debug which are rarely used during operation itself. It does, however, cut under the trace leading to the chip antenna, which is a larger problem that should have been fixed, as this trace is impedance controlled (at 50 ohms) as required for the antenna to function as intended. Although this trace is only for a reset pin, which is rarely used at all, the cut underneath the ground plane may cause problems for transmission. Besides this, there are a few low speed signals (UART and I2C) routed on the bottom layer and some 3.3V traces. 

## 2. Room for improvement

### Automotive classifications
This board was designed as a proof of concept primarily for automotive applications. However, none of the components are explicitly certified for automotive use. Using such components would require more time and research for part selection and probably also routing given the different size and shape of more bulky components due to their robustness. Further, no vehicle uses LoRa and other wireless communication standards to communicate movement information or general telemetry, preferring CAN to talk to other ECUs in a vehicle and the main controller over wire. However, this does not limit this PCB concept from other less intensive applications which involve movement and vibrations which need to be measured, such as drones. If the entire power protection and conversion setup is replaced by a fixed 3V battery it could be used for applications where plugged in power is not needed, like biological research. 

### Routing issues
The routing issues with the RF transceiver reset trace were brought up in the previous section. 
