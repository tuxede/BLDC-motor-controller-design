# BLDC-motor-controller-design
Hardware design, with additional code overview, of a typical drone BLDC motor speed controller(ESC).
Sofware part was done with STM32F407 microcontroller. Hardware was designed to handle 30A continously. Current version runs with 12V, so 3S pack, with absolute maximum around 14V. Circuit was designed to be very versatile, so with small modifications it can work with much higher voltages. One of my goals was not to use any specialized gate driver IC, again, to make it more universal. That's why I used only 4 NE555 and other common electronic components. 

# Code overview
I will not give you the exact code I used, but I can provide an overview of the code and it's working principle, which can be a small guide to replicate the same algorithm.

1. Current chopping
	To be able to controll motor's current, lower mosfet are being closed and opened with a frequence of about 30kHz, with variable duty cycle. This is implemented by sending a PWM signal to lower mosfets gates. 

2. BEMF sensing
	Electromotive force from a floating phase is used to determine current rotor position. The controller should be able to tell when a zero crossing between phase and center point occurs. We are using ADC conversion. First conversion is being taken from a floating phase, second from a virtual center point. Two measurements are then compared, to determine if a zero crossing occured.  
	To avoid electrical noise, that is present on both voltages time courses, conversions have to be called in a specific moment. Two measuremtn are always called at the beginning of a lower mosfets on pulse, after voltages settle a bit (maybe 1-2us after state change). Next two are taken in the second part of the current chopping PWM period. Precise moment of this measurement depends on a current PWM duty cycle, because any measurents must be taken when there are no significant transients on a voltage time course, so before, or some time after mosfet's state changes. 
	The measurents are taken two times a period, to provide sufficient resolution of a zero crossing detection. Imagine you look for zero crossing only once a 50us, but your signal has a period of 500us. It won't be accurate and the motor won't spin very smoothly (if at all). That's why PWM frequency is a bit higher than needed, and we are checking if zero crossing occured two times a period. 
	Also, first two measuremtns in every step are not taken into account when it comes to detecting zero crossing, because we assume that the voltage is not settled after commutation step transition. 
