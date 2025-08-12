# Contactless digital tachometer using 8051.
A three digit contact less digital tachometer using 8051 microcontroller which can be used for measuring the revolutions/second of a rotating wheel, disc, shaft or anything like that is introduced in this project. The tachometer Â can measure up to a maximum of 255 rev/sec at an accuracy of 1 rev/sec. What you just need to do is to align the sensor close to the reflective strip Â (aluminium foil, white paper or some thing like that) glued on the rotating surface and the meter shows the rev/sec on the display. The circuit diagram of the digital tachometer is shown below.

<img width="568" height="241" alt="image" src="https://github.com/user-attachments/assets/ca2737dd-93bd-4366-8276-d7a49983d8db" />


The first section of the circuit is the optical pickup based on photo transistor Q4 and red LED D4. Every time the reflective stripe Â on the rotating object passes in front of Â the sensor assembly, the reflected light falls on the photo transistor which makes it conduct more and as a result Â its collector voltage drops towards zero. When viewed through an oscilloscope the collector waveform of the photo transistor Q4 (2N5777) would look like this:

<img width="269" height="134" alt="image" src="https://github.com/user-attachments/assets/8e89b9f1-4721-4388-a6c3-b5c324e9cb79" />

Next part is the signal conditioning unit based on the opamp LM324 (IC1). Only one opamp inside the quad LM324 is used here and it is wired as a comparator with reference voltage set at 3.5V (using resistors Â R16 and R17). The job of this comparator unit is to convert the spiky collector wave form into a neat square pulse train so that it can be applied to the microcontroller. Every time the collector voltage of the photo transistor goes below 3.5V, the output of the comparator goes to negative saturation and every time the collector voltage of the photo transistor goes above 3.5V, the comparator output goes to positive saturation resulting in a waveform like this:

Â  Â  Â  Â  Â 

<img width="270" height="128" alt="image" src="https://github.com/user-attachments/assets/d86f4ffb-37fa-45ef-84af-08f6c333d1c1" />

From the above two graphs you can see that the negative going edge of the waveform indicates the passage of the reflective patch across the sensor and that means one revolution. If you could some how measure the number of negative going edgesÂ occurringÂ in one second, then that’s the rev/sec of the rotating object and that’s what the microcontroller does here.

The 8051 microcontroller here does two jobs and they are:

1) Â Count the number of negative going pulses available at its T1 pin (pin15).

2) Do necessary mathematics and display the count on the 3 digit 7 segment display.

For the counting purpose both the timers of 8051 (Timer0 and Timer1) are used. Timer 1 is configured as an 8 bit auto reload counter for registering the number of incoming zero going pulses and Timer0 is configured as a 16 bit timer which generate the necessary 1 second time span for the Timer1 to count.


