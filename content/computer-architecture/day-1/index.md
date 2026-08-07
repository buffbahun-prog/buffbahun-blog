+++
date = '2026-08-05T09:17:32+05:45'
draft = false
title = 'Day 1'
+++

As we are building a computer, though it's virtual, from the ground up, we should all ask ourselves a question. What actually is a computer? Hmm, time to think deep.

In my opinion, a computer is a glorified calculator. More specifically, it's a high-speed adder, capable of doing millions and billions of additions per second. But you may ask, "Shut up! Computers do crazy calculations." Yes, you are right, but every operation and calculation you think of can be, and is, derived from the addition operation.

Think of multiplication. It's just the addition of a number a number of times. A times B means adding A for B number of times, or vice versa. The power operation, similarly, is multiplication of a number some number of times. Now let's have a look at subtraction. Think of it as the addition of two numbers where the first number is positive whereas the second number is negative. Now how do we represent the negative number? Think of digits and numbers not as a linear line but as a circular clock. Now let's make an assumption that the first half of the numbers in the circle are positive numbers and the rest are negative numbers.

Let's analyze it with an example. Suppose we have numbers 0 to 9. Now imagine the numbers are arranged in a circle, just like a clock. Let's suppose we want the difference of 5 and 3 (5 - 3), which we can rewrite as (5 + (-3)). Now to get negative 3, we go backward (counter-clockwise) three steps, which is 7. So negative 3 (-3) is represented by 7. Summing 5 and 7 gives 12, where the 1 is the carry-over, so discarding it we get 2. And ta-da! 2 is the correct answer.

Now we have our maths right and we are convinced that the adder can be used for other operations such as subtraction. But how do we actually perform addition?

On the previous day we looked at the logical operations of binary numbers. Today we will be looking at their arithmetic operations. We humans have decimal numbers to do our daily chores. Have you noticed what's so special about them? Looking from the perspective of mathematics, there is nothing special. It's just that we have 10 fingers, and our ancestors used them for counting. This is how we developed our decimal number system.

In the decimal number system we start from 0, counting all the way to 9. After 9 we abandon any new symbol and instead combine 1 and 0 to form 10, and the increment repeats. Similarly, in binary first comes 0 and then 1, so what will be the next number? Yes, that's right: 10. Don't assume it's ten; it's simply 1 and 0.

Similar to arithmetic in the decimal system, adding zero to a number gives the same number. This applies not just to binary, but to any number system we choose to work with.

Let's add two binary numbers and see their results:
```text
             carry   sum
0 + 0 = 0    0       0
0 + 1 = 1    0       1
1 + 0 = 1    0       1
1 + 1 = 10   1       0
```

Are you noticing something? Yes, the sum part is the result of the XOR operation, whereas the carry part is the result of the AND operation. This means if we have AND and XOR gates, and we connect the inputs to these two gates, the output of the XOR gate is the sum, whereas the output of the AND gate is the carry-over.

I have implemented this in my code as:
```ts
export function halfAdder(bit0: Bit, bit1: Bit): [sum: Bit, carryOut: Bit] {
    return [
        xorGate(bit0, bit1),
        andGate(bit0, bit1),
    ]
}
```
This circuit is called a half adder because we have not accounted for the carry-in that should also be added to the two input numbers. So let's do just that:
```ts
export function fullAdder(carryIn: Bit, bit0: Bit, bit1: Bit): [sum: Bit, carryOut: Bit] {
    const [sumWithoutCarry, carry1] = halfAdder(bit0, bit1);
    const [sum, carry2] = halfAdder(carryIn, sumWithoutCarry);
    return [
        sum,
        orGate(carry1, carry2),
    ];
}
```
This circuit consists of two half adders. First, the sum of the two inputs is calculated, and then that sum is added with the carryIn input by another half adder, producing the final sum.

The carry-out is the OR output of the two carry values produced by the two half adders. This is because:
```text
0 + 0 = 0
0 + 1 = 1
1 + 0 = 1
1 + 1 = not possible/invalid
```
As you can see, since the last output is not possible, the remaining three inputs produce exactly the same result as the OR operator. Therefore, the carry-out can be obtained using the OR gate.

The above adders were all one-bit adders, so I have implemented an 8-bit adder as:
```ts
export function bitAdder8(carryIn: Bit, inp0: Bit8, inp1: Bit8): [sum: Bit8, carryOut: Bit , lastBitCarryIn: Bit] {
    const [sum0, carry0] = fullAdder(carryIn, inp0[7], inp1[7]);
    const [sum1, carry1] = fullAdder(carry0, inp0[6], inp1[6]);
    const [sum2, carry2] = fullAdder(carry1, inp0[5], inp1[5]);
    const [sum3, carry3] = fullAdder(carry2, inp0[4], inp1[4]);
    const [sum4, carry4] = fullAdder(carry3, inp0[3], inp1[3]);
    const [sum5, carry5] = fullAdder(carry4, inp0[2], inp1[2]);
    const [sum6, carry6] = fullAdder(carry5, inp0[1], inp1[1]);
    const [sum7, carry7] = fullAdder(carry6, inp0[0], inp1[0]);


    return [
        [sum7, sum6, sum5, sum4, sum3, sum2, sum1, sum0],
        carry7,
        carry6,
    ]
}
```
The Bit8 type is simply an array of eight Bits:
```ts
export type Bit8 = [Bit, Bit, Bit, Bit, Bit, Bit, Bit, Bit];
```
In bitAdder8, the carryOut of each fullAdder becomes the carryIn input of the next fullAdder.

The last function I have implemented is an adder-subtractor:
```ts
export function bitAdderSubstractor32(subMode: Bit, inp0: Bit32, inp1: Bit32): [result: Bit32, carryOut: Bit, overflow: Bit] {

    const [byte0Sum, carryOut0] = bitAdder8(subMode,
                                           inp0.slice(24, 32) as Bit8,
                                           inp1.slice(24, 32).map(bit => xorGate(subMode, bit)) as Bit8
                                        );
    const [byte1Sum, carryOut1] = bitAdder8(carryOut0,
                                            inp0.slice(16, 24) as Bit8,
                                            inp1.slice(16, 24).map(bit => xorGate(subMode, bit)) as Bit8,
                                        );
    const [byte2Sum, carryOut2] = bitAdder8(carryOut1,
                                            inp0.slice(8, 16) as Bit8,
                                            inp1.slice(8, 16).map(bit => xorGate(subMode, bit)) as Bit8,
                                        );
    const [byte3Sum, carryOut3, lastBitCarryIn] = bitAdder8(carryOut2,
                                            inp0.slice(0, 8) as Bit8,
                                            inp1.slice(0, 8).map(bit => xorGate(subMode, bit)) as Bit8,
                                        );

    return [
        [...byte3Sum, ...byte2Sum, ...byte1Sum, ...byte0Sum],
        // for add operation if 1 overflow,
        // and for sub 1 means A >= B and the result is positive,
        // so no borrow needed
        carryOut3,
        xorGate(lastBitCarryIn, carryOut3),
    ]
}
```
As I already explained above, subtraction is possible using addition if we first obtain the complement of the negative number. After that, the operation is simply addition between a positive and a negative number.

One nice thing about binary numbers is that if we invert all the bits and add 1, we get the complement. Let's look at an example.

Suppose we need to subtract 5 and 3 (5 - 3). The binary representation of 5 is 101, and for 3 it is 011, so the operation becomes:
```text
101 - 011
```
which we can rewrite as:
```text
101 + (-011)
```
Now, to get the complement of -011, we first invert 011, which gives 100, and then add 1 to it (100 + 1 = 101).

So 101 is the complement of -011. Since these are binary numbers, we call it the 2's complement.

Now let's add the two numbers:
```text
5 - 3
5 + (-3)
101 + (-011)
101 + 101
010 = 2
```
Here the carry bit is 1, which means the result is positive for the subtraction operation. If the carry bit is 0, it means the result is negative, which is denoted by the leftmost bit.

I'll give you an example with 3 - 5. If we follow exactly the same steps as above, we get:
```text
carryOut = 0
sum      = 110
```
Since the carry bit is 0, the resulting number is negative, which is denoted by the leftmost 1 in the result:
```text
110
(1)10
```
So the actual result is:
```text
-10
-2
```
Above is exactly what the bitAdderSubstractor32 function does. The first parameter is subMode, where 1 means subtraction and 0 means addition. We invert the bits only when subMode is 1 by using the XOR operator, because XOR outputs the inverted input bits when one of its inputs is 1; otherwise, it leaves them unchanged.

Phew, that was a lot. Now we are pretty familiar with how our computer actually adds and subtracts, and how it performs other complex operations using combinations of these simple but fundamental operations.

In the next chapter, I will explore how, using these gates, computers implement fast memory. Honestly, it's still over my head right now, but we'll figure it out together.

Till then, have a good one. Cheers.

Om Namah Shivaya.