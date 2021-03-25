---
layout: post
title: "Processor Log 2: Architecture Boogaloo"
tags: cpu devlog
date: 2021-03-22
usemathjax: true
---

# Processor Log 2: Architecture Boogaloo

Welcome. In this post, we'll talk about some of the high-level architecture plans for this processor. We'll take a look at a fantastic educational processor, the *SAP-1*, and use this as a baseline.

## SAP-1

The first 9 chapters of the book [Digital Computer Electronics by Malvino and Brown](https://www.amazon.com/Digital-Computer-Electronics-Albert-Malvino/dp/0028005945) teach the reader how to build the fundamental building blocks of computers out of logic gates. These chapters serve as the foundation needed to introduce the first iteration of their Simple As Possible processor (SAP-1). Below is a block diagram of this design:

![SAP-1](/assets/log2/SAP1.PNG){:height="75%" width="75%"}

Each block represents a circuit with a particular function. The lines on the outside edge of each block represent control signals which tell that block what it should do on the next clock cycle. For example, a high E<sub>A</sub> enables the output of the A register, meaning that whatever value this register holds will be asserted onto the bus. 

The bus is like a shared group of wires that the different modules can put their value on or read whatever value is on the bus. A high $ E\_A $ and a low $ \bar{L}\_O $ tells the A register to put the data it stores on the bus, and tells the output register to read and store whatever value is on the bus. In this way, data can be passed around the computer. 

### Program Counter

The program counter is a 4 bit counter (0000, 0001, 0010, ...) used to keep track of where we are in program execution. Each instruction is stored in the 16x8 RAM starting at address 0000. The first step to executing an instruction, then, involves asserting the value of the program counter to the bus, and read the value of the bus into the MAR (memory address register). 

### RAM

The RAM is where we store program instructions and data. RAM is a type of volatile memory: if we remove power from the circuit, the contents of our RAM will clear. This means that any program you want to run needs to be entered manually while the processor is powered on. This can be done with DIP switches, like these:

![SAP-1](/assets/log2/dipswitch.jpg){:height="75%" width="75%"}

You'd have one to select which address of RAM you'd like to write to, and another (with 8 switches) to set the 8 bit word you'd like to write to that address.

### Accumulator, B Register, and Adder/subtractor (ALU)

The accumulator is, functionally, just a register. It is used to store results of calculations carried out by the ALU, and the contents of the accumulator are wired directly to the ALU. The B register can only read from the bus, and feeds it's contents to the ALU. The Accumulator and B register can then be thought of as the two operands for any operation we want the ALU to perform. These registers are hard wired to the ALU, so any change to the contents of either register results in an instantaneous change in the ALU output value. 

The ALU of the SAP-1 is pretty much as simple as they come. It supports two operations: addition and subtraction.

### Controller/sequencer

The controller/sequencer is responsible for taking an instruction from the Instruction Register and producing the appropriate control signals for all the other modules of the CPU. Each instruction is divided into several steps, with a separate control word produced at each step.

### Where to improve?

Already there are a few pain points worth highlighting.

First, the 16x8 RAM. The RAM stores both program and data, meaning that our program can be at most 16 instructions long. If it is, then we have no room for storing any data we might want our program to use. There are a few ways of getting around this. One would be to make use of the [harvard architecture](https://en.wikipedia.org/wiki/Harvard_architecture), in which program and data memory are separate. This approach would effectively double our memory, we'd have one 16x8 program memory module, and one 16x8 data memory module. This would complicate the design a bit, but could be useful if decide to rework how instructions are executed later. A more straightforward option is to keep program and data memory shared, but dramatically increase the size. Instead of using just 4 address lines for 16 possible memory addresses, we can use 8 address lines giving us 256 addresses. I think this is the approach I'd like to start with, as it will give us plenty of room to work with, and we can always increase it later. This change will also allow us to increase the program counter to 8 bits.

Second, the lack of general purpose registers will make writing programs for this thing a nightmare. Since the B register can't even write to the bus, the Accumulator is full featured register we have. I'd like to add at least 2, both with bidirectional communication with the bus. Along with general purpose registers, we'll also want a [status register](https://en.wikipedia.org/wiki/Status_register). This can be useful for conditional jump instructions, in which we only want to branch program execution based on the result of an ALU operation. 

The adder/subtractor could also use some fleshing out. The ability to perform logical operations on the operands would make it a proper ALU, so I think this is a good goal.

Output is another interesting one. In a previous post I mentioned that an output display wasn't important to figure out early on, as just connecting an output register to 8 LED's would provide us with enough information to get off the ground. I still think this is the case, but I stumbled across a beautiful line of dot matrix displays that are ["ideal for applications where displaying eight or more characters of dot matrix information in an aesthetically pleasing manner is required."](https://www.broadcom.com/products/leds-and-displays/smart-alphanumeric-displays/parallel-interface/hdsp-2534) I'm determined to make use of this as our primary output display, so that decision is sorted, at least. 

We will, however, be using LED's for any display related to the processor's current state. Things like register contents, memory addresses, the value of the program counter, and the current bus contents, are all things that we'll want to be able to observe. I prefer using the LED's as a binary display approach for this sort of display, as it will directly tell us which pins are high and low. This will be convenient for troubleshooting, and to keep things tidy, we'll use LED bar graph units along with resistor networks like this:

![state display](/assets/log2/led-bar.jpg){:height="50%" width="50%"}


Finally, we'll want to implement a stack register/pointer. This will allow us to write subroutines, giving us some form of code reuse. I'll make another post about this later on in the project, for now we have enough components that can be built independently. 

Thanks for reading this far. All of these individual modules will get their own posts with more detail when it comes time to build them. In the next post, we'll get to it and build the heart of any CPU, the clock.

jc







