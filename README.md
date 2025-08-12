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
#Program
ORG 000H
MOV DPTR,#LUT              // moves the addres of LUT to DPTR
MOV P1,#00000000B          // Sets P1 as an output port
MOV P0,#00000000B          // Sets P0 as an output port
MAIN: MOV R6,#14D
      SETB P3.5
      MOV TMOD,#01100001B  // Sets Timer1 as Mode2 counter & Timer0 as Mode1 timer
      MOV TL1,#00000000B   //loads initial value to TL1
      MOV TH1,#00000000B   //loads initial value to TH1
      SETB TR1             // starts timer(counter) 1
BACK: MOV TH0,#00000000B   //loads initial value to TH0
      MOV TL0,#00000000B   //loads initial value to TL0
      SETB TR0             //starts timer 0
HERE: JNB TF0,HERE         // checks for Timer 0 roll over
      CLR TR0              // stops Timer0
      CLR TF0              // clears Timer Flag 0
      DJNZ R6,BACK
      CLR TR1              // stops Timer(counter)1
      CLR TF0              // clears Timer Flag 0
      CLR TF1              // clears Timer Flag 1
      ACALL DLOOP          // Calls subroutine DLOOP for displaying the count
      SJMP MAIN            // jumps back to the main loop
DLOOP: MOV R5,#100D
BACK1: MOV A,TL1           // loads the current count to the accumulator
       MOV B,#100D
       DIV AB              // isolates the first digit of the count
       SETB P1.0
       ACALL DISPLAY       // converts the 1st digit to 7 seg pattern
       MOV P0,A            // puts the pattern to Port 0
       ACALL DELAY         // 1mS delay
       ACALL DELAY
       MOV A,B
       MOV B,#10D
       DIV AB              // isolates the secong digit of the count
       CLR P1.0
       SETB P1.1
       ACALL DISPLAY       // converts the 2nd digit to 7 seg pattern
       MOV P0,A
       ACALL DELAY
       ACALL DELAY
       MOV A,B             // moves the last digit of the count to accumulator
       CLR P1.1
       SETB P1.2
       ACALL DISPLAY       // converts the 3rd digit to 7 seg pattern
       MOV P0,A
       ACALL DELAY
       ACALL DELAY
       CLR P1.2
       DJNZ R5,BACK1       // repeats the subroutine DLOOP 100 times
       RET

DELAY: MOV R7,#250D        // 1mS delay
 DEL1: DJNZ R7,DEL1
       RET

DISPLAY: MOVC A,@A+DPTR    // gets 7 seg digit drive pattern for current value in A
         CPL A             //  (See Note 1)
         RET
LUT: DB 3FH                // Look up table (LUT) starts here
     DB 06H
     DB 5BH
     DB 4FH
     DB 66H
     DB 6DH
     DB 7DH
     DB 07H
     DB 7FH
     DB 6FH
END

#NOTES 
1) The LUT used here was made for a common cathode seven segment display (used in previous projects) and here we are using a common anode display. The instruction CPL A will just complement the digit drive pattern in accumulator so that it becomes suitable for the common anode display. This is done just because to save my time but not a text book method. The correct way is to make a dedicated LUT for common anode configuration and aviod the extra CPL A instruction.
2) LM324 is a quad opamp and only one opamp inside it is used here. I used LM324 just because that was the only single supply opamp with me at the time. You can use any single supply opamp that matches our supply voltage(5V). You can even use a dual supply opamp (like the popular 741) in single supply mode (+V pin connected to positive supply and -V pin connected to ground) but i wont recommend it unless you have an oscilloscope. Dual supply opamps configured in single supply mode will not give results like a dedicated single supply opamp in the same situation.
