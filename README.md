# Byter
A basic 8-bit, bit serial CPU in 74HCxx Logic.

![image](https://github.com/user-attachments/assets/f8e88638-3eb9-4316-be43-8a8c72021261)


Bit Serial Arithmetic is sufficiently obscure to most - you have to start with the basics.

When I started this journey during the first COVID lockdown - it took many iterations to get the basic hardware to work using the "Digital" simulator.

First you need a full adder, then extend it to a complete ALU.
 
You add your Accumulator and B shift registers and make a simple adder -subtractor.

You need the Carry Flipflop and the Carry injection and suppression logic.

Finally you need a a clock sequencer to make it all work correctly.


This repo - will takes you through the basics:

1. INSTRUCTION DECODER

We start with the instruction Decoder.  U1 (74HC138) decodes the 3-bit instruction in IR6:4 into one of the 8 fundamental instruction groups:

LOAD

AND

OR

XOR

ADD

SUB

STORE

JUMP

The didode matrix and U2 (octal inverter 74HC540) cinvert these instructions into control signals that will control the ALU.

2. ALU

The ALU is made entirely from basic gates. One quand 2-input XOR and three quand 2-input NANDs.  U10:U13.

U10, 11 and 12 make up the ALU, and U13 deals with the Carry suppress and injection logic.

U10 and U11 form a full adder with the means to invert either of the two inputs A and B.

U12 creates a 2 input miltiplexer that chooses either the AND term or the XOR term from the full adder.

U13 provides a means to inject a carry when it is needed and suppress the carry chain during logical operations.

3. The Clock Sequencer.

The clock is split up into a series of timing pulses. There are several ways to achieve this. I use a 4 bit counter and a SR-latch.

4. Registers.

I show 3 registers, The Accumulator AC, The B-Register and an Output Register. All are 8-bits and they are co-ordinated by the timing pulses and the gated clock stream GCLK.







