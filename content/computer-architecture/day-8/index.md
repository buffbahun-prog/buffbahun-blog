+++
date = '2026-08-14T10:58:48+05:45'
draft = false
title = 'Day 8 — Logical Right Shifter'
+++

As promised, I have come up with an elegant solution for implementing the right shifter.

Initially, I was thinking of building a separate circuit for it and then using a 2-to-1 multiplexer so that the appropriate circuit could be selected. But as I was implementing this circuit, I came up with a realization:

Isn't right shifting just the reverse—or perhaps the exact opposite—of left shifting?

Let's investigate it ourselves.

Say we have an 8-bit binary number:
```text
11110000

1     1    1    1    0    0    0    0
bit0 bit1 bit2 bit3 bit4 bit5 bit6 bit7
```
We have positioned the bits from index 0 to index 7, from the higher-precedence bit to the lower-precedence bit.

Now say we want to right shift it by 1.

Let's first reverse its position so that the lower-precedence bit goes to position 0, and eventually the higher-precedence bit goes to position 7.
```text
 11110000
    |
    v
reverse its position
    |
    v
 00001111
```
The next step is to left shift it by 1, since we have already reversed its position.
```text
 11110000
    |
    v
reverse its position
    |
    v
 00001111
    |
    v
left shift by 1
    |
    v
 00011110
```
Now, to get the value we actually want, we need to reverse it again.
```text
11110000
    |
    v
reverse its position
    |
    v
 00001111
    |
    v
left shift by 1
    |
    v
 00011110
    |
    v
again reverse its position
    |
    v
 01111000
```
Voilà! We correctly get the right-shifted value.

With mathematics, we have figured out the inverse relationship between the two shifters. This can be written as:
```text
RSHIFT(data, shiftBy)
=
REVERSE(
    LSHIFT(
        REVERSE(data),
        shiftBy
    )
)
```
This is not an uncommon idea in mathematics or engineering. Very often, when two operations have a strong structural relationship, we can transform one operation into another instead of independently implementing both.

We can see a similar idea when thinking about addition and subtraction.

Consider the number line, an infinite line stretching from negative infinity to positive infinity, with the origin at 0.
```text
----|---|---|---|---|---|---|---|---|---|---|------------------------
   -4  -3  -2  -1   0   1   2   3   4   5   6
```
We can think of addition as movement along this number line.

When we say +5, we move 5 positions to the right from 0.

When we say -5, we move 5 positions to the left.

Now consider:
```text
5 + 3
```
We first move 5 positions to the right:
```text
0 → 1 → 2 → 3 → 4 → 5
-2
```
Then, because we are adding 3, we move another 3 positions to the right:

```text
5 → 6 → 7 → 8
```
So we arrive at 8.

Now consider subtraction:
```text
5 - 3
```

Subtraction can be understood as adding the inverse of a number:

```text
5 - 3
=
5 + (-3)
=
2
```

On the number line, -3 simply means moving three positions in the opposite direction:
```text
5 → 4 → 3 → 2
```

So again, we arrive at the correct answer.

The important insight here isn't that we should literally implement subtraction by reversing a number and then adding it. Rather, it is that mathematical transformations can sometimes let us express one operation in terms of another operation we already have.

And that is exactly the insight we are exploiting with our shifter.

Applying the insight to our circuit

With the theory out of the way, let's look at the implementation I came up with by applying the same inversion principle.

```ts
export function logicalShifter(data: Bit32, shiftBy: Bit32, shiftDir: Bit): Bit32 {
    // if shift value larger then 2^5 - 1, return 0
    const validShiftStartIndx = 32 - 5;
    const isValidShift = inverter(
        orGateNInp(shiftBy.slice(0, validShiftStartIndx))
    );
    const selectionBits = shiftBy.slice(validShiftStartIndx) as Bit5;

    const selectedDirData = data.map((dt, indx) => mux2To1(
        dt,
        data[data.length - (indx + 1)],
        shiftDir // shiftDir = 0 is left shift, 1 is right shift
    )) as Bit32;
    
    const shiftMuxOutputData = selectedDirData.map((_, indx) => {
        const bit0Array = Array.from({length: indx}).fill(0) as Bit[];
        const dataForMux = [...selectedDirData.slice(indx), ...bit0Array] as Bit32;
        const shiftedBit = mux32To1(dataForMux, selectionBits);

        return andGate(shiftedBit, isValidShift);
    }) as Bit32;

    return shiftMuxOutputData.map((dt, indx) => mux2To1(
        dt,
        shiftMuxOutputData[shiftMuxOutputData.length - (indx + 1)],
        shiftDir // shiftDir = 0 is left shift, 1 is right shift, on right shift reverse the reversed data
    )) as Bit32;
}
```

You will notice that I have removed—or, rather, modified—the previous left shifter I had made by adding the position-inversion multiplexers to both the input and output ends of the main left-shifter multiplexer network.

We now have an additional argument, shiftDir.

When shiftDir is 0, it means left shift.

When shiftDir is 1, it means right shift.

This value is fed into the 2-to-1 multiplexers that control the position inversion.

The first input to each multiplexer is the data value wired in its original position, while the second input is the exact opposite position from the original.

For example:
```text
Original:

bit0 bit1 bit2 bit3
  A    B    C    D

Reversed:

bit0 bit1 bit2 bit3
  D    C    B    A
```

Therefore, when shiftDir is 0, we get the unaltered data input.

When shiftDir is 1, we get the position-inverted data input.

Now we reach the middle layer, where our original left-shifter circuitry resides.

It doesn't care about the direction of the shift.

It simply shifts the supplied data to the left according to the shiftBy value.

And finally, at the output, we have another set of the same position-inversion multiplexers.

This is where the inversion introduced at the input is reversed.

So, when performing a left shift:
```text
original data
     ↓
no inversion
     ↓
left shift
     ↓
no inversion
     ↓
result
```
The middle shifter simply performs the left shift we intended.

For a right shift:
```text
original data
     ↓
position inversion
     ↓
left shift
     ↓
position inversion
     ↓
result
```
The data is first position-inverted, the existing left-shifter shifts the inverted data, and finally the second inversion restores the original orientation.

Therefore, we have implemented both left and right shifting using the same underlying shifter circuitry.

The main takeaway

I think I will end it here for today.

Although the implementation itself didn't take much time and looks relatively simple, the mathematical insight is the main topic of today's work.

Without this insight, we could have built another right-shifter circuit, potentially as large as the left-shifter circuit we already had.

Instead, we recognized the relationship:
```text
RSHIFT(data, shiftBy)
=
REVERSE(
    LSHIFT(
        REVERSE(data),
        shiftBy
    )
)
```
and reused the existing circuitry.

This is one of the things I am beginning to appreciate about computer architecture.

The complexity isn't always in the number of gates we need to build. Sometimes the biggest reduction in complexity comes from finding the right relationship between operations.

And this is exactly why I want to approach the architecture from first principles rather than simply looking up the circuit and reproducing it.

I will see you tomorrow.

Till then, have a good one.

Cheers.

Om Namah Shivaya.