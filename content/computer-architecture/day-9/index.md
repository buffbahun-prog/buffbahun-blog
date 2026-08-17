+++
date = '2026-08-17T05:56:20+05:45'
draft = false
title = 'Day 9 — Logical Rotator'
+++

Previously, I have talked extensively about the circuitry for logical left/right shifters and, with position inversion, implemented both left and right shifting using just the left shifter circuit, paired with position inverters in case of a right shifter. We have already built it.

Today, my ambitious plan is to again modify the shifter circuitry so that this same shifter circuit, with additions/modifications to some components, can perform both shifting operations as well as rotation operations—yes, in both directions.

Let’s first understand what exactly a rotation operation does. As in a shifting operation, the empty spaces that result from shifting the bits in one direction are filled with Bit 0 (we could have used Bit 1 too; it’s arbitrary). Similarly, instead of filling them with Bit 0, we fill them with the overflowed/removed bits from the other direction. Let’s look at an example:
```text
        shifting (left)             rotating (left)
            1 0 1 0                     1 0 1 0
               |                           |
               v                           v
remove<-[1] 0 1 0 _ <- 0       .<---[1] 0 1 0 _ <--.
               |               v                    |
               v               |____________________|
            0 1 0 0                     0 1 0 1

```

Simple, right? It forms a circular list, where the removed bits in one direction are appended to the empty spaces (due to the shifting) in the other direction. So, how do we translate this to the actual circuit? Remember, we used a 32 × 32-to-1 multiplexer for the 32 bits. Then, we configured the circuit such that each of the 32 wires of the 32 data bits was connected to each of the 32 multiplexers sequentially. The first mux was connected from bit 0 to bit 31 serially, the second mux starting from bit 1 to bit 32, but as the last input of the mux was out of connection as a result, we connected it with ground/Bit 0, as it represented the empty bits filled with zero.

So, for rotation, it is natural that instead of Bit 0, we connect the data bits from bit 0 up to the remaining data bits that were not present.

The function which I have modified will surely make this clear and give you an intuitive picture of the circuit:
```ts
export function shiftRotate32(data: Bit32, shiftBy: Bit32, shiftDir: Bit, rotate: Bit): Bit32 {
    // if shift value larger then 2^5 - 1, return 0
    const SHIFT_BITS = 5;
    const validShiftStartIndex = 32 - SHIFT_BITS;
    const isValidShift = inverter(
        orGateNInp(shiftBy.slice(0, validShiftStartIndex))
    );

    // equivalent to shiftBy % 32
    const selectionBits = shiftBy.slice(validShiftStartIndex) as Bit5;

    const transformedData = data.map((dt, indx) => mux2To1(
        dt,
        data[data.length - (indx + 1)],
        shiftDir // shiftDir = 0 is left shift, 1 is right shift
    )) as Bit32;
    
    const shiftMuxOutputData = transformedData.map((_, indx) => {
        const rotatedArray = transformedData.slice(0, indx);
        const shiftFillArray = rotatedArray.map(bit => andGate(bit, rotate));
        const dataForMux = [...transformedData.slice(indx), ...shiftFillArray] as Bit32;
        const shiftedBit = mux32To1(dataForMux, selectionBits);

        return andGate(shiftedBit, orGate(
            rotate,
            isValidShift
        ));
    }) as Bit32;

    return shiftMuxOutputData.map((dt, indx) => mux2To1(
        dt,
        shiftMuxOutputData[shiftMuxOutputData.length - (indx + 1)],
        shiftDir // shiftDir = 0 is left shift, 1 is right shift, on right shift reverse the reversed data
    )) as Bit32;
}
```
Here, most of the implementation is similar to our shifter circuit, but with some subtle modifications. This part:
```ts
// equivalent to shiftBy % 32
    const selectionBits = shiftBy.slice(validShiftStartIndex) as Bit5;
```
As after a complete rotation, the output returns to its original structure, it is implemented by the modulus operation. Don’t worry, I have an example for this explanation.
```text
left rotation
1 1 0 0
   |
   v
1 0 0 1
   |
   v
0 0 1 1
   |
   v
0 1 1 0
   |
   v
1 1 0 0
```
Compare the 1st and 5th rotations. They are the same, right? And the 6th rotation will be the same as the 2nd rotation, so it always forms a repeating structure every 5th rotation, which the modulo operation signifies. Taking the last 5 bits in the above function is the same as taking the 2⁵, i.e. 32, modulus.

But for shifting, all the bits are zero beyond the intended valid number of shifts:
```ts
return andGate(shiftedBit, orGate(
            rotate,
            isValidShift
        ));
```
This combination converts all the bits to 0 whenever the operation is shifting and beyond the valid shift number.

Okay, with this, all the other circuitry is the same, and this single circuitry/function will shift/rotate in both directions. I think with this, we should end today’s talk. I will surely see you tomorrow.

Till then, have a good one.

Cheers.

Om Namah Shivaya.