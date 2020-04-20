# BLDC-motor-controller-design
Hardware design, with additional code overview, of a typical drone BLDC motor speed controller(ESC).
Sofware part runs using STM32F407 microcontroller. Hardware was designed to handle 30A continously. Current version runs with 12V, so 3S pack, with absolute maximum around 14V. Circuit was designed to be very versatile, so with small modifications it can work with much higher voltages. One of my goals was not to use any specialized gate driver IC, again, to make it more universal. That's why I used only 4 NE555 and other common electronic components. 
The whole project is not licensed, so you should treat it mainly as an inspiration source.

## Code overview
I will not give you the exact code I used, but I can provide an overview of the code and it's working principle, which can be a small guide to replicate the same algorithm.

### 1. Current chopping
To be able to controll motor's current, lower mosfet are being closed and opened with a frequence of about 30kHz, with variable duty cycle. This is implemented by sending a PWM signal to lower mosfets gates. 

### 2. BEMF sensing
Electromotive force from a floating phase is used to determine current rotor position. The controller should be able to tell when a zero crossing between phase and center point occurs. We are using ADC conversion. First conversion is being taken from a floating phase, second from a virtual center point. Two measurements are then compared, to determine if a zero crossing occured. 

To avoid electrical noise, that is present on both voltages time courses, conversions have to be called in a specific moment. Two measurements are always called at the beginning of a lower mosfets on pulse, after voltages settle a bit (maybe 1-2us after state change). Next two are taken in the second part of the current chopping PWM period. Precise moment of this measurement depends on a current PWM duty cycle, because any measurents must be taken when there are no significant transients on a voltage time course, so before, or some time after mosfet's state changes. 
	
The measurents are taken two times a period, to provide sufficient resolution of a zero crossing detection. Imagine you look for zero crossing only once a 50us, but your signal has a period of 500us. It won't be accurate and the motor won't spin very smoothly (if at all). That's why PWM frequency is a bit higher than needed, and we are checking if zero crossing occured two times a period. This resolution is a limiting factor for max speed of the motor, which in my case was about 16 000 rpm. It also requires you to use a fast microcontroler (mine could sample at 2.4Msps with a single ADC unit, and it had 3 of those). 
	
Also, first pair of measurements in every step is not taken into account when it comes to detecting zero crossing, because we assume that the voltage is not yet settled after commutation step transition. 

### 3. Whole algorithm
Whole main algorithm is interrupt driven, mainly because most tasks are strictly real-time. Main while loop has a state machine, that determines current state of the controller (for example: "startup" state, "running"). Step timing is done with a separate timer overflow interrupt. To check if a zero crossing had occured, readings are compared in an ADC conversion complete interrutp. Another timer is utilized to generate 3 PWM signals. Additionaly, it's overflow interrupt calls first conversions, and it's fourth channel is used to time second ADC conversion's start (compare interrupt).

The controller also has a PI speed regulator, that uses alpha-beta filter to smooth speed readings (they vary quite a bit, because of the zero crossing detection resolution).

## Hardware
Hardware part consists of 6 main power transistors and their driving circuitry, and two auxiliary systems, providing voltage higher and lower than supply voltage. 
 ### 1. Power MOSFETs
 There are six n-channel MOSGFETs, three that are above load (voltage-wise) and another 3, which sources are connected directly to ground. I used 6 n-channels, because these are better suited for high current - low dissipation aplications (comparing to p-mosfets, n-channels made from the same sized die have much lower on-resistance). The circuit is calculated to be able to conduct 30 A on DC input. After gluing some small radiator, this current can be aplied continously. Using n-channels as upper mosfets has one disadvantage - to turn the transistor on, you need to apply to it's gate voltage higher than main supply. On the other hand - lower mosfets are pretty straight forward to controll - just aplly any reasonable, positive voltage to it's base. 
Lower mosfets gates driving uses voltage lower than power source (for example 8 V), and upper mosfets use voltage higher than supply (like Vcc+6V). 
