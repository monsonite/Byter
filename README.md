# Byter
A basic 8-bit, bit serial CPU in 74HCxx Logic.

![image](https://github.com/user-attachments/assets/ec734f51-9816-43f2-b17a-963f365d8bd3)

PART 1. Getting Startedon the basics.

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

The didode matrix and U2 (octal inverter 74HC540) converts these instructions into control signals that will control the ALU and carry logic.

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


Details:

One quand 2-input XOR and three quand 2-input NANDs.  U10:U13.

U10, 11 and 12 make up the ALU, and U13 deals with the Carry suppress and injection logic.

U10 and U11 form a full adder with the means to invert either of the two inputs A and B.

U12 creates a 2 input multiplexer that chooses either the AND (CARRY) term or the XOR (SUM) term from the full adder.

U13 provides a means to inject a carry when it is needed and suppress the carry chain during logical operations.


3. The Clock Sequencer.

![image](https://github.com/user-attachments/assets/be8e1062-0d5c-4e0a-af4f-e911b3f7ff1c)

The clock sequencer is based around a 4-bit counter (74HC161) U8. When clocked, this counter will count from 0 to 15 and on the $0F count produces the RCO (Ripple Carry Out) signal. RCO is captured by one half of the dual flipflop U9 and is used to produce true and complementary timing pulses RUN and T0.

U7 (74HC00) is a clock gating circuit.  Two of the NAND gates are configured as a SR latch. RES goes low, and is used to reset the latch. This makes the output RST high and allows the clock signal CLK to pass to the counter U8. when RST goes high the counter is taken out of reset and begins to count upwards from 0 to 15.

RST is effectively a clock gating signal. When high it allows a burst of clocks to be sent to the various shift registers - thus synchronising their operations. The inverted form of RST is available from the output of the lower NAND gate. This can be used as a /CE or /SS (slave select) signal for enabling SPI peripherals.

Carry Flipflop.

The other half of dual flipflop U9 is the "Carry Flipflop".  If a positive carry is generated during a bit-sum, it will be stored in U9B and presented to the Carry In (Cin) input of the ALU on the next clock cycle. The set and reset inputs of the flipflop are used to set or clear the carry, as required by the instruction being performed. For example a subtraction requires the carry to be set first at the start of the calculation but bitwise boolean logical operations require the carry to be suppressed for the duration of the operation so that it does not propagate between the bits and corrupt the operation.


4. Registers.

I show 3 registers, The Accumulator AC, The B-Register and an Output Register. The output register just follows the Accumulator - but just latches its output result at the end of the machine cycle. All are 8-bits and they are co-ordinated by the timing pulses and the gated clock stream GCLK.


In the next part, I will show how other ICs can be used to replace U2, U10, 11, 12, 13 thus reducing the package count.  

The Instruction decoding logic can be simplified and the Carry Suppress logic moved to the Carry Flipflop, rather than using additional NAND gates.

The clock sequencer counter may be set to generate different numbers of pulses in the clock burst. These van be used to control the Accumulator to provide left and right shift operations and also a nybble swap operation between the upper 4-bits and the lower 4-bits.


Part 2. Reducing package count - or getting more bang for your buck!

All combinational logic may ultimately be reduced to a ROM or other programmable logic such as PAL, GAL or FPGA.  These are all "blackbox" solutions, and not easy to follow as a newcomer to bit serial architechtures.

However, by carefully choosing devices that are more specialised, and contain more logic than basic 2-input gates, fairly large savings may be made in package count, and in some cases, these devices offer additional functionality, which previously would have required further packages.

I am initially going to focus on the Instruction Decoder and ALU. Between these sub-circuits, they use 6 packages, 1x 20 pin, 1x 16 pin and 4x 14-pin - plus an array of 15 diodes, which on a pcb would take up the same space as a pair of 14 pin DILs. 

We will keep U1, the 74HC138, as this is a very convenient way to perform the first stage of instruction decoding. We will eliminate the octal inverter and the diode matrix and its accompanying pull-up resistors.

We will minimise the rest of the decode logic finding a better overall solution for U12 and U13.

At the same time we will start to turn this manual adder/subtractor into a working CPU, allowing it to run stored programs from parallel memory.

This will require a Program Counter and additional circuitry, some of which will be absorbed into the enhanced ALU design.

A Better ALU Solution.

At this stage I would like to introduce the 74HC283 - it is a 4-bit parallel adder, with fast carry logic. Here you might say, but we only need a 1-bit full adder - a valid point. But the 74HC283 can provide an independent 1-bit full adder, and an independent 1-bit half-adder, which is exactly what we require for the Program Counter.

![image](https://github.com/user-attachments/assets/152b3b65-bce4-4a73-b108-997ff6d804e6)

Here is the internal logic equivalent of the 74HC283. It contains a total of 50 inverters, buffers and multi-input gates.

For our purposes we are going to split it into 2 sections, a 1-bit half-adder and a 1-bit full adder.  

A half-adder adds 2 binary digits producing the outputs 00, 01, 10 or 11.  It cannot produce an output 100, which would propagate a carry into the next stage. 

We rely on this certainty, to allow us to split a 74HC283 into 2 entirely separate halves.

![image](https://github.com/user-attachments/assets/329d07f3-a932-4be9-b9fe-c0a31888c48d)

In the sketch above, the half-adder for the Program Counter is constrained to the left hand side of the IC, whilst the full-adder for the ALU is constrained to the right-hand side of the IC.  The half-adder can never propagate a carry beyond S2, so is effectively isolated from S3 and S4.

The remaining XOR gate is the means to invert the bitstream from the B-register when we are performing a A-B subtraction operation.

Output Multiplexer.

In our original prototype ALU, most of a 2-input quad NAND gate was being used as the output multiplexer, to select between the SUM and CARRY terms of the full-adder.

A simmilar 2-input multiplexer will be needed for the Program Counter, to select between incrementing the PC by 1, or jumping to a completely new address.

There are almost no independent dual 2-input multiplexers available, in a single package - so we will make our own from  quad tristate buffers, the 74HC126 and its sister chip the 74HC125. Here we rely on using the tristate enable signal to switch several signal sources onto the one data output. The 74HC126 has a logic 1 tristate enable and the 74HC125 has a logic 0 tristate enable. With a 125 and a 126 working together, we can have 4 independent 2 input multiplexers in just 2 14-pin packages.

But first we look at just the '126.

![image](https://github.com/user-attachments/assets/7ae05fcd-408e-4dde-8564-f4c87e9429eb)


Here is a very specific use case, the ALU output multiplexer. Only the right hand of the IC is being used, the left hand side is available as another multiplexer.

We choose between using the ALU_SUM term or the ALU_CARRY term to provide our output function Fout.

Instruction control signals I0 and I1 are used to "steer" this logic selection.

If I0 is high we select the ALU_SUM, if I1 is high we select the ALU_CARRY.  If they are both low, both tristate buffers are off and a logic low is provided by the pull-down resistor.  If both I0 and I1 are high, thhe two diodes OR the outputs together providing a logical OR of the ALU operands.

If we need more than two independent multiplexers, we can combine a 125 and a 126 to make a 4 independent, 2 input multiplexers in a compact 2 chip solution.


Instruction Decoding, Subtraction and Carry Suppress Logic.

![image](https://github.com/user-attachments/assets/4c33b7bd-1532-4ea1-baa2-131c8989108e)

I have given this a revamp to remove the diode array and the octal inverter. It still uses the same 74HC138 for the initial instruction decode, but this is followed by three basic gate packages, a 74HC00 quad 2-input NAND, a 74HC86 quad 2-input XOR and a 74HC11 triple 3-input AND. The two inverters shown are actually XOR gates with one of their inputs tided high.








