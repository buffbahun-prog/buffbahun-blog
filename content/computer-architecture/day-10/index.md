+++

date = '2026-08-23T08:00:44+05:45'

draft = false

title = 'Day 10 — Unified Logical/Arithmetic Rotator/Shifter in Progress'

+++

It has been more than two days since I started working on a unified logical/arithmetic shifter and rotator circuit. It is getting increasingly complex, especially after adding the arithmetic right shifter and, more importantly, the rotate-through-carry logic. But slowly, I am getting a better understanding of the design, and I am now getting close to completing and thoroughly testing it.

Let's first talk about the arithmetic right shifter.

We have already had a thorough explanation of the left and right logical shifters. Their usefulness comes from operations such as multiplication and division by powers of two, as we have already discussed.

But what about signed numbers?

In our logical shifters, we fill the gaps created by shifting with `0`. In the case of a right shift, however, the most significant bit represents the sign of a signed number. If we simply fill the newly created bits with `0`, we can lose the sign information and therefore change the meaning of the number.

This is where the arithmetic right shifter comes in. While shifting the bits to the right, it preserves the sign by filling the newly created positions with the original sign bit (the most significant bit).

Now, the rotate-through-carry operation is also conceptually simple. In a normal rotate operation, the bits are rotated within the word in a circular pattern. A rotate-through-carry operation introduces one additional bit—the carry bit—into that circular sequence. In other words, the carry bit becomes an additional position in the rotation.

Therefore, our unified circuit needs to support seven operations:

- Logical left shift
- Logical right shift
- Logical left rotate
- Logical right rotate
- Arithmetic right shift
- Logical left rotate through carry
- Logical right rotate through carry

To accomplish this, we also need control circuitry so that the appropriate shift or rotate operation can be selected. The goal is to build a general-purpose circuit that can perform all of these operations using the data, shift amount, carry bit, and control bits as inputs, and produce the transformed data and the new carry bit as outputs.

This time, I am also determined to implement a wide range of test cases. The circuitry is becoming increasingly complex, with many different variables and edge cases to consider, so I think this is a good point in the project to take testing much more seriously.

And, as always, once the circuitry is complete, we will take a thorough look at its implementation details and understand exactly how everything works at the gate level.

Till then, have a good one.

Cheers.

Om Namaha Shivaya.