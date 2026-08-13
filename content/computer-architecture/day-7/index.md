+++
date = '2026-08-13T10:03:16+05:45'
draft = false
title = 'Day 7 | Logical Left Shifter'
+++

As usual, I was doing my study on the microprocessor and computer architecture. I was skimming through the ALU part, and as far as my understanding went, I had already implemented the adder. For logical operations, it seemed pretty obvious: the whole CPU/computer is built upon the foundation of these logical gates, so the Logic Unit must contain these gates to operate on 'N' number of inputs. But I was wrong, or rather, I had no idea about other binary operations that might be useful enough to be implemented at the hardware level with dedicated circuitry. The first one I encountered was the Logical Shifter.

I want to explain this with a simple example. Suppose we have an 8-bit binary number:
```text
11111111
```
Now I want to move/shift every bit to the left side, so what happens? As the leftmost bit shifts from its current position to the position on the left, the bit overflows because there are only 8 bits, and it is gone for good. Now the next bit gets to the leftmost position vacated by the previous bit while shifting. This ripples all the way to the rightmost bit. Now notice that the rightmost position is vacant, as its bit moved to the left due to shifting, so we insert a 0 bit into the vacant position.
```text
               1 1 1 1 1 1 1 1 <-
overflowed   1 1 1 1 1 1 1 1 _ empty due to shifting
               1 1 1 1 1 1 1 0

```
The above example is a simple one because we shifted the bits just once, but we can shift it by 'N' number of times. As we shifted the number just once in the above example, when shifting by 'N', the leftmost N bits get overflowed and discarded, while the rightmost N bits will be 0 due to the shifting.
```text
               Left Shift by 2
               1 1 1 1 1 1 1 1 <-
overflowed 1 1 1 1 1 1 1 1 _ _ empty due to shifting
               1 1 1 1 1 1 0 0

```
And as there are 8 bit positions in the above number, after the 8th shift and beyond, the value becomes 00000000 because its maximum possible shift is 8, as there are only 8 bits. Beyond this, no matter what the shifting value is, it is 00000000.

Why is it useful, you ask? As it turns out, left shifting produces a value double its input value, which is equivalent to multiplying the previous value by 2. And in the case of shifting by multiple positions, it gives us powers of 2. Okay, I will prove it with an example.
```text
                    00010110  decimal value 22

    leftshift by 1  00101100  decimal value 44 (22 * 2 = 44)              
    leftshift by 2  01011000  decimal value 88 (22 * (2 power 2) = 44)              




                     00000001  decimal value 1 (2 power 0 = 1)

    leftshift by 1   00000010  decimal value 2  (2 power 1 = 2)
    leftshift by 2   00000100  decimal value 4  (2 power 2 = 4)
    leftshift by 3   00001000  decimal value 8  (2 power 3 = 8)
    leftshift by 4   00010000  decimal value 16 (2 power 4 = 16)
```
Then, after I was quite surprised by these results, my mind shifted (seriously, not a pun) towards implementing it. But how, exactly? The answer lies in selecting/picking a bit and changing its position. And what do we have in our tools for selecting a particular bit? Yes, the multiplexer.

Think of the shift-by number as the select input of the multiplexer. Now, for example, say we have 8-bit input data, so we know that there are 3 select inputs, from 000 (0) to 111 (7). As we already know, a multiplexer outputs just one bit among the 8 data input bits, based on the select input. From this logic, we know that select input 000 outputs the leftmost bit, 001 the one just after the leftmost bit, and so on, until 111 gives the bit at the last/rightmost position of the data input.

Okay, now building on this logic. Suppose we bring in 8 of these 8-to-1 multiplexers. Think of it this way: when we say left shift by zero, we shift it 0 times, i.e. we don't shift it, and the output is the same as the input. Mapping shift-by-zero to the select input of the multiplexers gives us 000. Then, with these 8 multiplexers sharing the same select input, which is 000 in this condition, how should they be connected to output the same value as the input? Let's connect a multiplexer to the data input in the same order, from bit position 0 to 7, and intend this multiplexer to output the leftmost shifted output bit. When select input 000 (shifting by 0) is given, the data input connected to the first input of this multiplexer has its leftmost bit ([1]1111111) connected to it. So obviously, it will output the leftmost bit when shifting by 0 (that is, select input 000).

Similarly, with the same mindset, let's think about how the second multiplexer should be configured. Now, as we have already said, on shifting by zero (select input 000), the multiplexer should output the bit just after the leftmost bit (1[1]111111). For this, we should connect the first input of the multiplexer to the second-index bit, with the other bits connected serially, mapping with the data value. Now you will notice that the last bit, i.e. the 7th position, has nothing to connect to because we started from the second bit. And as we have already stated, the empty bits after shifting are replaced by 0, so a 0-bit connection is given to it.

Similarly, with the third multiplexer, using the same logic, we will have the first input for the multiplexer connected to the 3rd bit input line (11[1]11111), and its remaining 2 last inputs replaced by 0-bit connections. I think you see the pattern through all 8 multiplexers. Now, combining the 8 outputs from the 8 multiplexers, the first multiplexer that we configured with its 1st input line connected to the 1st data input will produce the leftmost bit, and the last one will produce the rightmost bit.

Honestly, it took me some days to get it properly, as I was trying to discover it from first principles. You can try this method; it's quite helpful.

We should now look at the implementation that I programmed:
```ts
export function logicalLeftShifter(data: Bit32, shiftBy: Bit32): Bit32 {
    // if shift value larger then 2^5 - 1, return 0
    const validShiftStartIndx = 32 - 5;
    const isValidShift = inverter(
        orGateNInp(shiftBy.slice(0, validShiftStartIndx))
    );
    const selectionBits = shiftBy.slice(validShiftStartIndx) as Bit5;
    return data.map((_, indx) => {
        const bit0Array = Array.from({length: indx}).fill(0) as Bit[];
        const dataForMux = [...data.slice(indx), ...bit0Array] as Bit32;
        const shiftedBit = mux32To1(dataForMux, selectionBits);

        return andGate(shiftedBit, isValidShift);
    }) as Bit32;
}
```
As I have already discussed above, the output will be zero for a shift-by value above the total length of bits present in the data input. In our implementation, as there is a 32-bit input line, 0 to 31 are the valid shift-by values. So we just take the rightmost (least significant) slice from the 32-bit shift-by input, and if the high-significant bits have at least one bit with value 1, that means the value is greater than 31. With the AND gate at the end, we output 0 bits in the case of invalid shift-by values.

Apart from this, the implementation is simple enough. For the 32-bit data input, I have mapped each multiplexer with the correct configuration, starting with the leftmost bit at the proper position of the data input and filling the rightmost positions with 0 bits according to the position of the data input and output.

One extra thing I did was implement the 32-to-1 multiplexer. It's a large function, so don't get overwhelmed; it's simple and carries the same logic as the simplest 2-to-1 multiplexer. One other extra gate I added for this multiplexer is the N-input gate. We have our useful, simplest, and most-used 2-input gates, but we also require gates with multiple inputs. So have a look at it:
```ts
export function mux32To1(data: Bit32, selectBits: Bit5): Bit {
    return orGateNInp([
        // for select value 00000, the first bit in the data should be selected
        // so all the select inputs inverted
        andGateNInp([
            data[0],
            ...selectBits.map(bit => inverter(bit)),
        ]),
        // for select value 00001, only the first 4 bits needed to be inverted
        andGateNInp([
            data[1],
            ...selectBits.map((bit, indx) => indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 00010, except 3rd index bit other needed to be inverted
        andGateNInp([
            data[2],
            ...selectBits.map((bit, indx) => indx !== 3 ? inverter(bit) : bit),
        ]),
        // for select value 00011, except 3rd and 4th index bit other needed to be inverted
        andGateNInp([
            data[3],
            ...selectBits.map((bit, indx) => indx !== 3 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 00100, except 2nd index bit other needed to be inverted
        andGateNInp([
            data[4],
            ...selectBits.map((bit, indx) => indx !== 2 ? inverter(bit) : bit),
        ]),
        // for select value 00101, except 2nd and 4th index bit other needed to be inverted
        andGateNInp([
            data[5],
            ...selectBits.map((bit, indx) => indx !== 2 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 00110, except 2nd and 3rd index bit other needed to be inverted
        andGateNInp([
            data[6],
            ...selectBits.map((bit, indx) => indx !== 2 && indx !== 3 ? inverter(bit) : bit),
        ]),
        // for select value 00111, except 2nd, 3rd and 4th index bit other needed to be inverted
        andGateNInp([
            data[7],
            ...selectBits.map((bit, indx) => indx !== 2 && indx !== 3 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 01000, except 1st index bit other needed to be inverted
        andGateNInp([
            data[8],
            ...selectBits.map((bit, indx) => indx !== 1 ? inverter(bit) : bit),
        ]),
        // for select value 01001, except 1st and 4th index bit other needed to be inverted
        andGateNInp([
            data[9],
            ...selectBits.map((bit, indx) => indx !== 1 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 01010, except 1st and 3rd index bit other needed to be inverted
        andGateNInp([
            data[10],
            ...selectBits.map((bit, indx) => indx !== 1 && indx !== 3 ? inverter(bit) : bit),
        ]),
        // for select value 01011, except 1st, 3rd and 4th index bit other needed to be inverted
        andGateNInp([
            data[11],
            ...selectBits.map((bit, indx) => indx !== 1 && indx !== 3 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 01100, except 1st and 2nd index bit other needed to be inverted
        andGateNInp([
            data[12],
            ...selectBits.map((bit, indx) => indx !== 1 && indx !== 2 ? inverter(bit) : bit),
        ]),
        // for select value 01101, except 1st, 2nd and 4th index bit other needed to be inverted
        andGateNInp([
            data[13],
            ...selectBits.map((bit, indx) => indx !== 1 && indx !== 2 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 01110, except 1st, 2nd and 3rd index bit other needed to be inverted
        andGateNInp([
            data[14],
            ...selectBits.map((bit, indx) => indx !== 1 && indx !== 2 && indx !== 3 ? inverter(bit) : bit),
        ]),
        // for select value 01111, except 1st, 2nd, 3rd and 4th index bit other needed to be inverted
        andGateNInp([
            data[15],
            ...selectBits.map((bit, indx) => indx !== 1 && indx !== 2 && indx !== 3 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 10000, except 0th index bit other needed to be inverted
        andGateNInp([
            data[16],
            ...selectBits.map((bit, indx) => indx !== 0 ? inverter(bit) : bit),
        ]),
        // for select value 10001, except 0th and 4th index bit other needed to be inverted
        andGateNInp([
            data[17],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 10010, except 0th and 3rd index bit other needed to be inverted
        andGateNInp([
            data[18],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 3 ? inverter(bit) : bit),
        ]),
        // for select value 10011, except 0th, 3rd and 4th index bit other needed to be inverted
        andGateNInp([
            data[19],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 3 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 10100, except 0th and 2nd index bit other needed to be inverted
        andGateNInp([
            data[20],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 2 ? inverter(bit) : bit),
        ]),
        // for select value 10101, except 0th, 2nd and 4th index bit other needed to be inverted
        andGateNInp([
            data[21],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 2 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 10110, except 0th, 2nd and 3rd index bit other needed to be inverted
        andGateNInp([
            data[22],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 2 && indx !== 3 ? inverter(bit) : bit),
        ]),
        // for select value 10111, except 0th, 2nd, 3rd and 4th index bit other needed to be inverted
        andGateNInp([
            data[23],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 2 && indx !== 3 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 11000, except 0th and 1st index bit other needed to be inverted
        andGateNInp([
            data[24],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 1 ? inverter(bit) : bit),
        ]),
        // for select value 11001, except 0th, 1st and 4th index bit other needed to be inverted
        andGateNInp([
            data[25],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 1 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 11010, except 0th, 1st and 3rd index bit other needed to be inverted
        andGateNInp([
            data[26],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 1 && indx !== 3 ? inverter(bit) : bit),
        ]),
        // for select value 11011, except 0th, 1st, 3rd and 4th index bit other needed to be inverted
        andGateNInp([
            data[27],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 1 && indx !== 3 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 11100, except 0th, 1st and 2nd index bit other needed to be inverted
        andGateNInp([
            data[28],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 1 && indx !== 2 ? inverter(bit) : bit),
        ]),
        // for select value 11101, except 0th, 1st, 2nd and 4th index bit other needed to be inverted
        andGateNInp([
            data[29],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 1 && indx !== 2 && indx !== 4 ? inverter(bit) : bit),
        ]),
        // for select value 11110, except 0th, 1st, 2nd and 3th index bit other needed to be inverted
        andGateNInp([
            data[30],
            ...selectBits.map((bit, indx) => indx !== 0 && indx !== 1 && indx !== 2 && indx !== 3 ? inverter(bit) : bit),
        ]),
        // for select value 11111, nothing need to be inverted
        andGateNInp([
            data[31],
            ...selectBits,
        ]),
    ]);
}
```

See, it's the same as a 2-to-1 multiplexer, just larger because of the 32 AND gates that needed to be added for the 32 input lines. That's it.

Okay, I think it's time to end today's talk. After today's left shifter, you may be asking, if there is left shifting going on, then what about right shifting, thus the Logical Right Shifter? Wouldn't that exist? Yes, you are right to have this question. And I promise you, tomorrow we will get the Right Shifter done right!

Till then, have a good one.
Cheers.

Om Namaha Shivaya.