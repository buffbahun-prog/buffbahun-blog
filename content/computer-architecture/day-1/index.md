+++
date = '2026-08-05T09:17:32+05:45'
draft = true
title = 'Day 1'
+++

As we are building a computer though its virtual, but from ground up, we all should ask ourself  question. What actually is a computer? Hmm, time to think deep. In my openion a computer is a glorified calculator. More specifically, its a high speed adder, capable of doing millions and billions of addition per second. But you may ask "shut up!, computers do crazy calculations". Yes you are right but every operations and calculations you think of cn be and is stem by the addition operation.

Think of multiplication, its just addition of a number a number of times. A time B means add A, for B number of time or vice-versa. The power opertion similarly is multiplication of a number some number of times. Now lets hve a look at substraction. Think of it as addition of two numbers where the first number is  positive number whereas the second number is a negetive number. Now how do we represent the negetive number? Now think of digits and numbers as not a linear line but a circular clock, now lets make an assumption that first half of the numbers in the circle are positive numbers and the rest as negetive numbers.

Lets analyze it with an example, suppose we have numbers 0 to 9, now imagive the numbers are in a circle same as in a clock. lets suppose we want difference of 5 and 3( 5 - 3 ), which we can arrange it like ( 5 + (-3) ). Now to get negetive 3, we go backward/counterclock-wise three steps which is 7. So negetive 3 (-3) is represented by 7, so summing 5 and 7 gives 12, where the 1 is crry over so discarding it we get 2. And thada 2 is the correct answer.

Ok now we have our maths right and we are convinced that adder operator can be used for other operations such as substraction. Now how do we do the additions? On previous day we looked at the logical operations of binary numbers, today we will be looking at its arithematic operations. We human have decimal numbers to do our daily chores. have you noticed thats so special about it? Its nothing special looking from the prespective of mathematics, its just we have 10 fingers and our ancestors used to do counting with their fingers and this is how we developed our decimal number system.

In decimal number system we start from 0 counting all the way to 9, now after 9 we abondone any new symbol but combine 1 and 0 to form 10, and the increment is repeted. Similarly in binary first comes 0 and then 1, so what will be the next number? Yes right 10, dont assume its ten, its 1 and 0. Similar to the arethematics in the decimal system something added with zero gives that number, applies to not just binary but we get the idea that to any number system we decide to work with.

Lets add two binary numbers and lets see their results:
             carry   sum
0 + 0 = 0    0       0
0 + 1 = 1    0       1
1 + 0 = 1    0       1
1 + 1 = 10   1       0

Are you noticing something? Yes, the sum part is the result of xor operation, whereas the carry part is the result of the and operation. This means if we have a and and xor gates and we connect the inputs to this two gates and the output of the xor gate is the sum part, whereas the output of the and gate is the carry over part. I have implemented this in my code as:
```ts
export function halfAdder(bit0: Bit, bit1: Bit): [sum: Bit, carryOut: Bit] {
    return [
        xorGate(bit0, bit1),
        andGate(bit0, bit1),
    ]
}
```
This circuit is called a half adder, because we have not accounted for the carry in that shouldalso be added to the two input numbers. Then lets do just that:
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
This circuit consists of two half adder, first the sum of the two inputs are calculated, and then the sum output is added with the carryIn input by another hald adder, so the ouptupted sum is the full sum. Now the carry out is the OR output of the two calculated carry from the two hald adder. This is because:
0 + 0 = 0
0 + 1 = 1
1 + 0 = 1
1 + 1 = not possible/invalid
You can see hereas the last output is not possible the above three inputs have the same result of that of the OR operator so the carry out is ouptuped from the OR gate.

The above adders where all one bit adder, so I have implemented a 8 bit added as:
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
and the Bit8 type is just a array of 8 Bits:
```ts
export type Bit8 = [Bit, Bit, Bit, Bit, Bit, Bit, Bit, Bit];
```
on the bitAdder8 the carryout of each of the fullAdder is the carryIn input of the next fullAdder. The last function I have implemented is adder-substractor:
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
As I have already cleared it out on the above paragraphs, the substraction is possible between two numbers with a addition operation if we get the complement of the negetive number, and after that the operation is just addition between a possitive and negetive number. And one nice thing about binary numbers is if we invert all the bits and add 1 we get the complement. Lets look at an example. Supose we need to substract 5 and 3 such as 5 - 3, now the binary representation of 5 is 101 and for three is 011, so the operation becomes:
101 - 011
which we can write is as:
101 + (-011)
Ok now to get the compliment of -011, we first invert 011, which gives 100, and now we add 1 to it (100 + 1 = 101). So 101 is the compliment of -011, as its binary numbers we call is 2's compliment. Lets now add the two numbers:
5 - 3
5 + (-3)
101 + (-011)
101 + 101
010 = 2
Here the carry bit is 1 which means the result is positive for the substraction operation, if the carry bit is 0 it means the result is negetive, which is denoted by the leftmost digit. I will give you and example for 3 - 5, if we did all step exactly as above we get result:
0 as carryOut and 110 as sum, now as the carry bit is 0 meaning the number resulting is negetive which is denoted by the leftmost 1 in the result so the result actually is:
110
(1)10
if carryOut bit for substraction operation is 0 ( leftmost bit if 0 then (+) else if leftmost bit is 1 then (-) )
-10
-2
Above this is what the bitAdderSubstractor32 function dos, the first parament is subMod, i.e. if 1 substraction operation ,if 0 addition. And below we re doing invertion if subMod is 1 with a xor operator as xor operator outputs the inverted input bits when subMod is 1 else gives the output as it is when subMod is 0.

Phew, that was a lot. Now we are pretty familiar on how our computer actually adds and substracts and does other complex operation with the combination of this simple but fundamental operations. Ok on the next chpter, I will explore about how the hect using this gates does computer imlements fast memory. Honesty its over my head right now. Ok till then have a good one. Cheers.
Om Namaha Shivaya.