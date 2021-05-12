---
layout: post
title: "Registers"
tags: cpu devlog
date: 2021-04-01
---


# Registers

Welcome. In this post, we'll look at the basic building block of our registers: the D flip-flop. We'll also talk about our register design before building a few of them. 

## D flip flop

A D flip-flop has two inputs: a clock signal, and a data signal. When the clock is high, Q and !Q (inverted Q) will follow whatever D is. When the clock signal is low, changing D results in no change to the outputs, but the last state of D is preserved. This gives us a way to store 1 bit of information.

Let's walk through how this works. The following diagram shows how you can construct a D flip-flop out of NAND gates:

![](/assets/registers/circuit.png)

The triangle is the symbol for an inverter, while the half-pills with a circle are NAND gates. 

Consider the case in which CLK (the clock signal) is high. If D is also high, the output of the top left NAND gate is 0. The bottom left NAND gate, however, gets the inverse of D as its second input, so the output of this gate is 1. 

Recall the truth table for a NAND gate: 

| A | B | Out |
|---|---|-----|
| 0 | 0 | 1   |
| 1 | 0 | 1   |
| 0 | 1 | 1   |
| 1 | 1 | 0   |

An important observation is that whenever one of the inputs to the gate is 0, we know that the output is 1, irrespective of the other input value. So, in the case where both CLK and D are 1, the output of the top left NAND gate is 0, so the output of the top right NAND gate (Q output) must be 1. The inputs to the bottom left NAND gate in this case are 1 (CLK) and 0 ($\overline{D}$). Thus, the output of this NAND will be 1, so the two inputs to the bottom right NAND will both be 1, resulting in a 0 output. 

When CLK is high, the value of Q will follow whatever the D input is. When CLK is low, however, the previous Q and !Q outputs are preserved. This gives us a way to store 1 bit of information. 

This circuit as is would be useful, but we can get a bit more control over this flip-flop by AND'ing the clock signal with another control signal, Load Enable (LE). If we make this change, then both CLK and LE need to be high in order for the input value D to be latched. This is substantially more useful, as now we can determine when (in terms of clock cycles) we want the input data to be latched. 

## Chip Choice

If we take 8 of these, tie all of their CLK and LE signals together, and add a similar output enable (OE) control signal, then we have an 8 bit register.

The chip we'll be using for this is the [74LS573](https://www.futurlec.com/74LS/74LS573.shtml). It functions in (almost) exactly as our registers need to, complete with control signals. 

There are, however, a couple of issues we need to sort out. First, we want to be able to observe the contents of the register at any given moment and to wire the outputs of the A register (Accumulator) directly to the ALU, without a pass-through connection to the main data bus. Second, we don't want the register to track the inputs for the entire duration of the clock pulse, we want to store the input value (from the bus) at the *instant* the clock goes high. 

To address the first issue, we can have the output enable pin always be active (tied directly to ground), and use a second chip to actually interface with the bus. This approach will allow us to wire the output of the register to some LED's and the ALU, without having these outputs affect the bus. 

There is already a chip for this, called the [74LS245](https://www.ti.com/lit/ds/symlink/sn54ls245-sp.pdf?ts=1616930648534&ref_url=https%253A%252F%252Fwww.google.com%252F). This chip is a bidirectional bus transceiver, it has 20 pins, 8 to connect to the A bus, and 8 to connect to the B bus. It also has a direction pin, which determines whether the OE pin will send data from A to B or from B to A. 

In our case, the B bus connection will be wired to the output pins of the 74LS573. The A bus pins will connect to our main bus, as well as the input pins of the 74LS573. 

We'll just be using one direction, as we don't need the same control over whether the register should be reading from the bus as we do for output. The output control signal for any given register we build will be connected to the 74LS245's output enable, rather than the output enable of the 74LS573. 


## Edge Detection

The second issue I pointed out has to do with timing. When we're reading the value on the bus into a register, it'll be difficult to guarantee that we're actually storing the intended value: we might have certainty around what value is there at the start of the clock cycle, but we don't have any guarantees that the state of the bus will remain constant for the entire cycle. 

This might sound like a weird thing to be worried about

Let's assume our bus is normally tied to ground through some resistors. 








