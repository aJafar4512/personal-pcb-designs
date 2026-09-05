# Design and points to improve

## 1. Design considerations

### Connector
An AMPSEAL automotive grade 8 pin connector, by TE Connectivity used for both the 12V power rail (assumed to be from the vehicle's low voltage bus) and CAN differential signal to save space and reduce complexity. 2 pins used for 12V, 2 for CAN_H and CAN_L and 3 pins in between these two for ground (0V) for maximum shielding between the sensitive differential traces and the potentially noisy power. One pin left unconnected. 

### CAN integration
This board was originally an expansion on my previous project involving an IMU and RF setup. I learnt after that design that individual ECUs in vehicles rarely communicate via wireless signal; instead, the CAN bus has become standard across automotive applications to connect electronics within a larger system together. 
This boards builds on that knowledge with a CAN transceiver receiving CAN_H and CAN_L (differential signal) from the connector. The differential signal is passed through a split termination group, with two 60Ω resistors and a capacitor to filter out common mode noise before reaching the transceiver. It is a Texas Instruments SN65HVD232QD powered by 3.3V from the buck converter instead of the more usual 5V for CAN transceivers; this prevents us needing two regulator stages. It is also Q100 qualified for automotive applications.

### Protection and regulation
The 12V rail is first connected to a fuse to protect against current surges, followed by a transient voltage suppression diode, primarily against sudden electrostatic discharge. A Schottky diode from here provides reverse polarity protection. The output from the latter now passes into the buck converter; this was designed using TI's Webench Power Designer platform, which designs regulators based off the requirements and suggests all components needed. This gives us our 3.3V rail to power components. 

### Microcontroller Unit (MCU)
The STM32F303K8T6 by ST Microelectronics was chosen for it's minimal footprint, simple pinout setup and integrated CAN controller. An external high speed oscillator is also used for accurate clocks at 24 MHz. Two pairs of decoupling capacitor + bulk capacitor are placed near the MCU so it can function as intended. It is placed centralized so as to keep routing distances as short as possible to key components like the two sensors. The MCU has serial wire debug inbuilt, so 4 copper pads are placed on the edge for the feature to be accessed by connecting debug setups to the pads.  

### Proximity sensor
Functions by measuring the intensity of reflected infrared (IR) light, and communicates with the MCU via I2C. Model number VCNL4035X01-GS08 by Vishay.
It's used to report the height of the board above the ground through time; in certain corners a car at high speed may tilt to one side (barrel roll), so one or two corners may be closer to the ground than the others.  

### Accelerometer
The H3LIS331DL by ST Microelectronics is a high-g 3 axis accelerometer. It is used to measure when the board may suddenly jerk or vibrate, such as a vehicle with this board passing over a bump or uneven road surface. As the board would be located near all 4 wheels, it can help diagnose issues with the chassis in case the response from the accelerometers is dissimilar across the 4 nodes. It communicates with the MCU via SPI. 

### General routing and board layers
Due to the large AMPSEAL connector there was plenty of vertical space for components so routing was not difficult as a whole. A large 3.3V trace was used to keep current within safe margins (even if vias were required on occasion). The ground polygon on the top layer effectively shields the CAN differential signal against the 12V rail and then a similar polygon shields the 3.3V rail and some other components (like the oscillator Y1) against the same 12V rail. The board could have been made more easily with a 4 layer stackup where shielding against electromagnetic interference can be enhanced with a SIG-GND-GND-SIG stackup. However, to ensure a routing challenge I used a 2 layer stackup where as much as possible of the bottom layer was filled with GND polygon pour. It may not be perfect, but the few signals routed on the lower layer are low speed and short, and orthogonal routing as been implemented, so effects on signal integrity can be perceived as minimal. 

## 2. Room for improvement

### Automotive classifications
Devices used in automotive applications require a certain robustness and typically have a certification called AEC-Q100. In this design, only the CAN transceiver and AMPSEAL connector fulfill this requirement. Finding AEC-Q100 certified components which would work in a design demands additional time and resources, which is something I need to work on going forward. 

### The proximity sensor
The proximity sensor can be considered a placeholder due to the mismatch between it's intended use and application in this design. IR reflectivity is not reliable for measuring distance to black, hot tarmac so the sensor would not give any helpful information in an automotive application; a time of flight or hall effect sensor would be far more suitable.
