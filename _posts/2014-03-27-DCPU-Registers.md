---
layout: post
title: DCPU - Registers
categroy: posts
---

For my DCPU project I'm going to try and talk a little about each major component.
The first of which is the general purpose registers.

In virtually all architectures, there is access to the
[Processor Registers](http://en.wikipedia.org/wiki/Processor_register), some
architectures may provide many that have direct access. For instance, the
[MIPS Architecture](http://en.wikipedia.org/wiki/MIPS_architecture) provides
several dozen general purpose registers that can be used for any kind of storage.
The advantage of these registers is that they usually have faster write/read times
than reading from memory. So for example, if a variable is being read many times
over the course of a routine, it would be advantageous to store it in a register
so that it may be used more efficiently.

Now, this is of course only the tip of the iceberg. There are many different kinds
of registers, and many different uses for them. Check out the link above if your
interested! The chip I've implemented is a *very* simple 8 bit register, with 4
of them stuffed into a single chip, toggled by a write mode.

## Composure

Like the project implies, I wanted to go down to the discrete logic level of
implementation for the CPU. Naturally, I needed to make some kind of general storage
for any values of transaction in the CPU. Internally, this is done by a type of
[flip-flop](http://en.wikipedia.org/wiki/Flip-flop_(electronics)) called a Gated
D-Latch:

![dgate](/images/dgate.png)

I chose the Gated D Flip-Flop because it allows one to toggle whether the flip-flop
can, well, flip. This is very useful later on for assigning and maintaining addresses
between opcodes.

## Layout

Currently I have a 4 x 8 Bit Register Bank that I have been using, honestly it would
be very tough to integrate anything more complex at the moment. In each chip, there 
is 4 columns of 8 (1 bit) D flip-flops, in other words, each column is one bytes. 
I consider each one of these to be a processor register, so the word size is 1 Byte
(8 Bits). While I opted to expand the logic in my actual register, here's a simplified
array conceptually similiar to what is going on:

![darray](/images/darray.png)

In my circuit, I delineate each array by a decoded input that 'selects' an array within
the bank. This is done by AND'ing the chosen circuit with the gate control pin of each
flip flop, all I have to do is enable the whole array and the it is 'chosen'. Something
similiar is done for the return value on the output:

![registers](/images/registers.png)

As you can see, there is a 'mode' pin that enables writing to the bank. This modularizes
the chip enough that I could put any arbitrary size or amount of registers within it.
This comes in handy later as I use it for both input and output of user values.

Funny, while writing this I actually found a bug where you would write a value and the bit
would persist across a row. Would have been really nast to figure out with the ALU. You
can see the changes on the right-most side. The all not-ed AND gate (I know I could have 
just used a NAND), is an addition I had to make.

## Conclusion

It's not the prettiest thing, but I really like how modularized it is, and how simple it
is to use. I also should be able to build the thing later on, though there is a fair amount
of components to it. By inputing the desired register with the "Selector" pins, you can get
or set the value of the register. This is of course up to a limit of 256, since the word
size of the register is 8 bits. 

I had to seperate out the input and output pins, which as different from the original design,
as [Logisim](http://ozark.hendrix.edu/~burch/logisim/) did not support a two-in-one.
Originally I had them as one controlled by buffers that would cut the circuit depending on
the mode (read/write).
