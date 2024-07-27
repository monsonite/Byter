# Byter
A basic 8-bit, bit serial CPU in 74HCxx Logic.

![image](https://github.com/user-attachments/assets/ec734f51-9816-43f2-b17a-963f365d8bd3)



Bit Serial Arithmetic is sufficiently obscure to most - you have to start with the basics.

When I started this journey during the first COVID lockdown - it took many iterations to get the basic hardware to work using the "Digital" simulator.

First you need a full adder, then extend it to a complete ALU.
 
You add your Accumulator and B shift registers and make a simple adder -subtractor.

You need the Carry Flipflop and the Carry injection and suppression logic.

Finally you need a a clock sequencer to make it all work correctly.


This repo - will takes you through the basics:

The following is NOT a full CPU, it is just an ALU testbed to ensure that the serial arithmetic process is working correctly.

1. INSTRUCTION DECODER

![image](https://github.com/user-attachments/assets/21a46953-8c0f-4d63-98e7-f3a0cae0e346)


We start with the instruction Decoder.  U1 (74HC138) decodes the 3-bit instruction in IR6:4 into one of the 8 fundamental instruction groups:

LOAD

AND

OR

XOR

ADD

SUB

STORE

JUMP

The didode matrix and U2 (octal inverter 74HC540) convert these instructions into control signals that will control the ALU.

The Instruction Decoder is entirely combinational logic, so could be reduced to basic gates or even coded into a ROM.

2. ALU

![image](https://github.com/user-attachments/assets/faf7353b-dc04-4214-82d2-fe1b1e18323d)

The ALU is entirely combinational logic made entirely from basic gates so could be coded into a ROM.

It has 12 inputs, and 5 outputs:

Ain   The bitstream from the Accumulator Shift Register

Bin   The bitstream from the B Shift Register

Cin   Any Carry generated from the previous bit-sum

InvA  Invert the A input for B-A subtraction operations

InvB  Invert the B input for A-B subtraction operations

Mode  Suppress Carry propagation for logical operations

SetC (T1)  Set the Carry on Timing Pulse T1

Sub   Asserted to indicate when a subtraction is performed

LDA (T0) Timing Pulse T0 initiates loading of the Accumulator

CLA  Clear Accumulator first, if a LOAD instruction is being performed

I0 Instruction input 0 - selects whether ALU output is the SUM function

I0 Instruction input  -  selects whether ALU output is the CARRY function

I1:I0   00    ZERO 

I1:I0   01    XOR

I1:I0   10    AND 

I1:I0   11    OR

OUTPUTS:

Fout  - The SUM Output of the ALU

Cout  - The CARRY output of the ALU

/CSET - Preset the Carry Flipflop

/CRES - Pre-clear the Carry Flipflop

/CLRACC - Clear the Accumulator




One quand 2-input XOR and three quand 2-input NANDs.  U10:U13.

U10, 11 and 12 make up the ALU, and U13 deals with the Carry suppress and injection logic.

U10 and U11 form a full adder with the means to invert either of the two inputs A and B.

U12 creates a 2 input multiplexer that chooses either the AND (CARRY) term or the XOR (SUM) term from the full adder.

U13 provides a means to inject a carry when it is needed and suppress the carry chain during logical operations.


3. The Clock Sequencer.

![image](https://github.com/user-attachments/assets/be8e1062-0d5c-4e0a-af4f-e911b3f7ff1c)

The clock sequencer is based around a 4-bit counter (74HC161) U8. When clocked, this counter will count from 0 to 15 and on the $0F count produces the RCO (Ripple Carry Out) signal. RCO is captured by one half of the dual flipflop U9 and is used to produce true and complementary timing pulses RUN and T0.

U7 (74HC00) is a clock gating circuit.  Two of the NAND gates are configured as a SR latch. RES goes low, and is used to reset the latch. This makes the output RST high and allows the clock signal CLK to pass to the counter U8. when RST goes high the counter is taken out of reset and begins to count upwards from 0 to 15.

RST is effectively a clock gating signal. When high it allows a burst of clocks to be sent to the various shift registers - thus synchronising their operations. The inverted form of RST is available from the output of the lower NAND gate. This can be used as a /CE or /SS (slave select) signal for enabling SPI peripherals


4. Registers.

I show 3 registers, The Accumulator AC, The B-Register and an Output Register. The output register just follows the Accumulator - but just latches its output result at the end of the machine cycle. All are 8-bits and they are co-ordinated by the timing pulses and the gated clock stream GCLK.


In the next part, I will show how other ICs can be used to replace U1, U2, U10,11,12,13.




