# BLDC-motor-controller-design
Hardware design, with additional code overview, of a typical drone BLDC motor speed controller(ESC).
Sofware part was done with STM32F407 microcontroller. Hardware was designed to handle 30A continously. Current version runs with 12V, so 3S pack, with absolute maximum around 14V. Circuit was designed to be very versatile, so with small modifications it can work with much higher voltages. One of my goals was not to use any specialized gate driver IC, again, to make it more universal. That's why I used only 4 NE555 and other common electronic components. 
