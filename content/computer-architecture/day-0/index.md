+++
date = '2026-08-04T14:52:46+05:45'
draft = false
title = 'Day 0'
+++

The other day I was studying some algorithms used in the file system and buffers that the Unix system used. I was fascinated by the cleverness and engineering intuition behind the implementation. The engineer inside of me screamed in excitement, "Holy shit! This is the real deal."

Now I have a crazy idea. Why not build my own operating system from scratch? Sounds insane, right? Yes, this scares me. I have no idea whatsoever, but with the right knowledge and, most of all, the right mindset, I think this can be accomplished, though it may take years and years. This is why you are reading this, as this is me documenting my journey. So let's dive into this voyage.

I have created a <a href="https://github.com/buffbahun-prog/she-is-the-booms" target="_blank" rel="noopener noreferrer">repo</a>. You may be shocked to see TypeScript here. Double shocked on seeing an HTML file. Inside the src directory you will see a directory named virtual-machine. Yes, a virtual machine. Don't get confused; I will explain why.

My goal is to create an operating system, and I have to eventually run it on an actual machine/microprocessor. But until then, I have a lot to implement and test. So I thought, what if I implement my OS in the browser first so that I can massively test algorithms and implementations? But the browser is just a user application, so I am implementing a simple virtual computer inside the browser.

See, my intuition is straightforward. To properly implement my OS, I need to have a good foundation in the architecture and hardware side of the computer, especially the processor, memory, and storage. Plus, I will add a lot of debugging principles from the hardware level so it will be way easier to debug my OS. Again, I am a novice in both architecture and hardware. We will learn and implement along the way.

Let us start with the building blocks of computers: the humble logic gates. I have my implementations below:

```ts
export function andGate(bit0: Bit, bit1: Bit): Bit {
    if (bit0 === 1 && bit1 === 1) return 1;
    else return 0;
}

export function orGate(bit0: Bit, bit1: Bit): Bit {
    if (bit0 === 1 || bit1 === 1) return 1;
    else return 0;
}

export function inverter(bit0: Bit): Bit {
    if (bit0 === 0) return 1;
    else return 0;
}

export function nandGate(bit0: Bit, bit1: Bit): Bit {
    return inverter( andGate(bit0, bit1) );
}

export function norGate(bit0: Bit, bit1: Bit): Bit {
    return inverter( orGate(bit0, bit1) );
}

export function xorGate(bit0: Bit, bit1: Bit): Bit {
    return andGate(
        orGate(bit0, bit1),
        nandGate(bit0, bit1),
    );
}
```
The Bit type I have implemented is simply:

```ts
export type Bit = 0 | 1;
```

Just as arithmetic operators such as addition (+) and subtraction (-), logical gates are devices that perform binary operations on inputs and return a binary output. The three major logic gates are the AND, OR, and NOT gates, and the binary operations they perform are AND, OR, and inversion respectively. The operations are also called logical operations because, as we see, we have just two values, 0 and 1, constituting the binary numbers. They can be thought of as logical values such as true and false, or on and off. I think you get the idea. It's the mathematical representation of philosophical logic.

The AND logic gate performs the AND operation on the inputs. It is the same as how we use "and" in our daily conversations. For example, when we go to a shop and say, "Please give me a shirt which has the colors red and blue." The store person gives you a shirt having both red and blue colors. The AND operator returns the Bit value 1 if and only if all the inputs are the Bit value 1; otherwise, it gives the output 0.

Similarly, the OR gate is also similar to our daily conversations, where we mean having either one of the options. The OR operator gives the output Bit 1 if any of the inputs is Bit 1, and Bit 0 only in the case when all the inputs are Bit 0.

Another fundamental logic gate is the NOT gate. It's more of an inverter than an operator, as it outputs the flipped value of the input. If the input is Bit 0, then the output is Bit 1, and vice versa.

Now, the other operators that I have implemented are combinations of these three fundamental gates (AND, OR, and NOT). The XOR operator is an interesting one, as it outputs Bit 1 when the two inputs are different; otherwise, when both inputs are the same, it outputs Bit 0.

I think this much will do for today. Tomorrow we will try to implement an adder from these logic gates. Till then, cheers. Have a good one.

Om Namah Shivaya.