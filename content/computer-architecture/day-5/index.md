+++
date = '2026-08-10T09:58:17+05:45'
draft = false
title = 'Day 5 — Multiplexer'
+++

The previous day, we just began to automate our machine with a counter. It's definitely a stepping stone. And we also introduced what an instruction might look like for our machine. Now suppose we have just added two values in registers r1 and r2 and stored the value in register r3 with an ADD instruction. Now suppose we want to store the added value in r3 to memory for later use. For this, we will have a STORE instruction that could take the value from the register and store it at a memory address.

So, visualizing this, we actually have two paths/sources that can directly access the address lines in the memory. The program counter has its output line connected to the address line input of the memory, and just now we found out that we also want the address line of memory to be connected with the register's output line.

This is where I realized the true use case of a multiplexer. A multiplexer is a logical device (obviously built from gates) that, having inputs of 2 power n (2ⁿ), with a selector of n inputs, always outputs 1 value from any one of its input data, according to the selector input we provide. I will give you a simple example.

The simplest of these multiplexers is a 2-to-1 multiplexer, meaning it has two inputs and one output. Now, the selector is always log base 2 of the total number of data inputs. So, in this case, we have 1 select input. It's logically simple, as a bit can encode 2 distinct pieces of information/data, so with 1 select input we can have 2 data inputs. Similarly, with 2 select inputs, we can have 4 data inputs, which is a power of 2. And the inverse of power is log. So, I think this clears the doubt.

With this multiplexer, as I have already stated, I can select either of the data inputs by properly setting the select input. So, in our previous example, I can set the select input to 0 if I want the program counter as the input, whereas I can set the select input to 1 if I want the register as the input.

I have implemented the 2-to-1 multiplexer as below:
```ts
export function mux2To1(inp0: Bit, inp1: Bit, select: Bit) {
    return (
        orGate(
            andGate(
                inp0,
                inverter(select),
            ),
            andGate(
                inp1,
                select,
            )
        )
    );
}
```
With the above function, I think you get the idea of how the circuit is formed. The core is, an AND gate having input i1 and i2 always outputs i2 if i1 is 1, else always 0 when i1 is 0. So, inversing the select input in one of the AND gates always ensures our output is always zero, and the other is i1/i2. And the final OR operation outputs i0/i1 as the other input is 0. So, this way, the select input dictates which input, either inp0 or inp1 in the above function, will be the output.

```ts
export function mux32Bit2To1(inp0: Bit32, inp1: Bit32, select: Bit) {
    return inp0.map((_, indx) => mux2To1(
        inp0[indx],
        inp1[indx],
        select,
    )) as Bit32;
}
```
The above function is just the parallel combination of the above multiplexer so that our whole 32-bit data/address lines are covered. It's just using the multiplexer on each of the 32 lines.

You might feel this series is moving at a turtle's pace. But trust me, slow and steady is the way, at least when trying to understand small topics and concepts as intuitively as possible. Tomorrow, I am planning to study the history and all major timelines of the history of computing. Right now, I feel an itch to know how all of this marvelous engineering came to be.

So, I will be back tomorrow, same time, same place.

Till then, have a good one.

Cheers.

Om Namaha Shivaya.