+++
date = '2026-08-11T09:22:56+05:45'
draft = false
title = 'Day 6 — Demultiplexer'
+++

Yesterday, we got into detail about multiplexers. Now, you might be thinking that with n select inputs, we could output any of the 2ⁿ data inputs through a single output line. Now, isn't it possible that we could pass the data from a single input to any of the multiple output lines? Think of it this way: while with a multiplexer we narrowed multiple input lines down to a single output line with a selector, similarly, we could have a device that does the exact opposite. It takes a single input data line and, with the proper selector bits, passes it to one output line among the multiple ones. And this is called a demultiplexer.

The selector and the possible output lines are the same as those of the multiplexer. The primary distinction is only that while the multiplexer had 2ⁿ inputs for n selectors and one output line, the demultiplexer has 2ⁿ outputs for n selectors and only one input line. The MUX selects one output from many inputs, while the DEMUX (short for demultiplexer) selects one from the many possible outputs.

This is quite a useful circuit, as it can be used for selecting a particular item from many possible items. Let's, for example, say we have 8 total registers in our system, starting from r0 to r7. Now we have an instruction:

```text
LOAD r3, 0x127f
```

Meaning, load the constant value 0x127f into register r3. Now, the data path from memory to all registers is a common one. But the data in the data path is only written to the registers whose write-enable input is set to Bit 1. Now, how do we write to that particular register among all these registers, which all have this common data path? Here is where the demux is particularly useful.

Now, since there are 8 registers ranging from r0 to r7, we can map each register in binary form from 000 to 111, because 2 power 3 gives 8 possible values. So, we can map r0 to 000, r1 to 001, and so on until r7, which is 111. In our example above, r3 will be 011. Now, if we take this as the select input for the demux with n equal to 3, we will have 8 outputs. We can then connect each of the 8 outputs to the write-enable input of the registers serially, starting from the r0 register to r7 in sequential order.

Now, when we have the above instruction LOAD r3, 0x127f, r3 will be mapped to 011, which, when given as input to the demux, will output 1 only to the r3's write-enable input, so that only the r3 register writes the data to its register.

Let's have a look at the demux I have implemented:
```ts
export function demux1to2(dataInp: Bit, select: Bit): [Bit, Bit] {
    return [
        andGate(
            dataInp,
            inverter(select),
        ),
        andGate(
            dataInp,
            select,
        )
    ]
}
```
I have, for now, implemented the simplest demux, which is a 1-to-2 demux: 1 select input to 2 outputs. As there are two outputs, I have returned an array of two outputs. The use of the inverter on the select input in the first AND gate makes the first and second AND gates have opposite select inputs from each other, thus only one output is 1 while the other is guaranteed to be 0. Now, the AND gate which got its second input (select/inverted select) as 1 will output whatever the first input value is, i.e., the data input value. While the other AND gate, which surely got its second input as 0, will always return an output of 0, whatever the first input might be.

As you can see, I have implemented a lot of components and circuits till now. But now that I have many fundamental circuits/components needed to build a computer, I still have no idea how I would map the instructions to the data and control signals of all these components. From my perspective, this is the heart of the computer. I have planned to have an investigative look at the simple classic microprocessors from the past to get some ideas.

So, till then, have a good one.

Cheers.

Om Namaha Shivaya.