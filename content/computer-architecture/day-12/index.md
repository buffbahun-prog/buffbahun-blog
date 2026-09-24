+++
date = '2026-09-06T08:28:25+05:45'
draft = false
title = 'Day 12 — Arithematic Logic Unit'
+++

We have come across the arithmetic part, which was binary addition and subtraction. We have also come across the logical parts, such as AND, OR, NOT, and XOR gates. Now, the natural progression is to combine these circuitries to form a unit. So, that's the ALU.

Let's have my implementation presented as follows. I first had a 1-bit ALU, which is a simpler circuitry:
```ts
// control signals
// [ input 1 invert, input 2 invert/negetive, ...operations(2 bit)]
// operations
// [0,0] -> AND
// [0,1] -> OR
// [1,0] -> XOR
// [1,1] -> ADD
export function aluBit1(carryIn: Bit, inp1: Bit, inp2: Bit, controlBits: Bit4): [result: Bit, carryOut: Bit] {
    const inp1Invert = controlBits[0];
    const inp2Invert = controlBits[1];
    const operationBits = controlBits.slice(2) as Bit2;

    const inp1Transformed = xorGate(inp1, inp1Invert);
    const inp2Transformed = xorGate(inp2, inp2Invert);

    const andResult = andGate(inp1Transformed, inp2Transformed);
    const orResult = orGate(inp1Transformed, inp2Transformed);
    const xorResult = xorGate(inp1Transformed, inp2Transformed);

    const [addResult, carryOut] = fullAdder(carryIn, inp1Transformed, inp2Transformed);

    const result = mux4To1(
        [
            andResult,
            orResult,
            xorResult,
            addResult,
        ]
        ,
        operationBits
    );

    return [result, carryOut];
}
```
There are four control bits in total. The first two are for inverting the two inputs, and the remaining two are inputs to the mux to select the desired primitive operations. With 2 bits for operation control, we get a total of four unique operations. I have chosen AND, OR, XOR, and ADD. You might think: where is the NOT operation? What about subtraction? How can we get other gates, such as NOR and NAND? Those are some legitimate queries.

Here is the interactive model for the 1 Bit ALU:
{{< circuit src="https://aksa-os.pages.dev/playground/alu?circuit=alu-1-bit" title="Interactive 8-bit ALU" >}}

We have a deep understanding of binary arithmetic at this point. We know that subtraction is also an addition operation, but between positive and negative numbers. The 2's complement method and the representation of binary codes make the subtraction problem an addition problem. So, inverting the second input and adding 1 bit (from the input carryIn) makes the subtraction operation possible.

Likewise, NAND and NOR operations can be easily carried out using De Morgan's laws by setting the inversion of inputs A and B. The XOR operation is also useful for carrying out the NOT operation, where we can set input 1 as our intended binary value, whereas setting input 2 to all 1s is equivalent to the NOT operation.

So, intentionally, I am keeping the 1-bit ALU simple enough to have more operations performed with less circuitry and fewer control signals. Now, how many operations are to be performed by the accumulation of these 1-bit ALUs is in the power of the main ALU circuitry, which I will simply call the ALU. Here is its implementation:
```ts
// 000 -> AND     A & B
// 001 -> OR      A | B
// 010 -> XOR     A ^ B
// 011 -> PASS_B  B
// 100 -> ADD     A + B
// 101 -> SUB     A - B
// 110 -> SLT     A < B (signed)
// 111 -> SLTU    A < B (unsigned)
export function ALU(inp1: Bit32, inp2: Bit32, controlBits: Bit3): [result: Bit32, carryOut: Bit, overflow: Bit, zero: Bit] {

    // map
    // AND     000 --> [0,0,0,0]
    // OR      001 --> [0,0,0,1]
    // XOR     010 --> [0,0,1,0]
    // PASS_B  011 --> [0,0,1,1] final mux pass B
    // ADD     100 --> [0,0,1,1]
    // SUB     101 --> [0,1,1,1] CarryIn = 1
    // SLT     110 --> [0,1,1,1] CarryIn = 1, lsb = carryOut/borrow and rest 0
    // SLTU    111 --> [0,1,1,1] CarryIn = 1, lsb = overflow xor msb and rest 0

   const mappedAluCode = [
    0,
    andGate(controlBits[0], orGate(controlBits[1], controlBits[2])),
    orGate(controlBits[0], controlBits[1]),
    orGate(controlBits[0], controlBits[2]),
   ] as Bit4;

   const negateB = mappedAluCode[1];
   const carryIn = negateB;

   let msbCarryOut = carryIn;
   let msbCarryIn = carryIn;

   const aluResult = Array.from({length: 32}) as Bit32;

   for (let i = 31; i >= 0; i--) {
    const a = inp1[i];
    const b = inp2[i];

    const [result, carryOut] = aluBit1(msbCarryOut, a, b, mappedAluCode);
    msbCarryIn = msbCarryOut;
    msbCarryOut = carryOut;

    aluResult[i] = result;
   }

   const carryOut = msbCarryOut;

   const overflow = xorGate(msbCarryIn, msbCarryOut);

   const signedLess = xorGate(
        aluResult[0],
        overflow,
    );

   const unsignedLess = inverter(carryOut);

   const isLess = mux2To1(
        signedLess,
        unsignedLess,
        controlBits[2],
   );

   const issltOp = andGate(negateB, controlBits[1]);

   const aluResultExceptLsb = aluResult.slice(0, 31);
   const aluResultLsb = aluResult[31];

   const afterSltResult = [
    ...aluResultExceptLsb.map(bit => andGate(issltOp, bit)),
    mux2To1(aluResultLsb, isLess, issltOp),
   ] as Bit32;

   const isPassBOp = andGateNInp([
    inverter(controlBits[0]),
    controlBits[1],
    controlBits[2],
   ]);

   const finalResult = afterSltResult.map((bit, indx) => mux2To1(
    bit,
    inp2[indx],
    isPassBOp,
   )) as Bit32;

   return [finalResult, carryOut, overflow, norGateNInp(finalResult)];
}
```
Just have a look at the comments where I have pointed out the control signals and the mapping to the 1-bit ALU below. The other operations are fairly simple at this point, involving linearly mapping them to the 1-bit ALU and using its output without further transformation. The SUB operation that we have already discussed above does what we had anticipated. The second input is inverted, and the carryIn for the LSB 1-bit ALU is set to 1.

The operations that we have not yet encountered are PASS_B, SLT, and SLTU. The PASS_B operation, as the name suggests, outputs input 2 as it is. Why is this so useful that it is an operation of the ALU? Keep this in mind: later, when we map out the datapaths of all our circuitry as a whole, we will have the register datapath always be an input to the ALU (the register files will have two data outputs). Now, suppose we want to store a value in a register. We first pass it through the ALU. While it is very useful and must pass through the ALU, it also passes through the 1-bit ALU, right? So, in this case, we could have any operations, such as ADD, in my design arbitrarily, and at the end, I have used a mux to output input 2 in this case.

Now, the SLT and SLTU operations are set less than, meaning they check whether input 1 is less than input 2. If true, they set the output data to 000...1 (the LSB is 1); otherwise, all bits are 0. SLTU is for unsigned values, whereas SLT is for signed binary values. These operations are also very useful in the case of jump instructions with certain conditions.

Okay, let's briefly talk about their implementation. Let's have a look at the following mathematical operation:
```text
    a < b
    substract b on both sides
    a - b < b - b
    a - b < 0
```
With the last expression, we can conclude that if the sign bit (in the case of signed numbers) is negative, i.e., Bit 1 is the MSB, or if the borrow (the inverse of carryOut) is 1, then surely the output number is negative. So, we can conclude that the first input is less than the second input. The rest of the logic is where I have implemented this logic.

```ts
const overflow = xorGate(msbCarryIn, msbCarryOut);

const signedLess = xorGate(
     aluResult[0],
     overflow,
 );

const unsignedLess = inverter(carryOut);

const isLess = mux2To1(
     signedLess,
     unsignedLess,
     controlBits[2],
);
```
We XOR the MSB with the overflow flag in the case of signed values. This is because overflow occurs when the output sign is not the correct one. In this case, the inverse of the output MSB is the correct one, so we simply XOR it, which ultimately inverts the MSB when overflow is 1.

With all of this, we finally output the result data along with carryOut, overflow, and the zero flag (which is 1 when all the result output bits are 0; otherwise, it is 0). The zero flag is also useful in the case of branch and jump instructions with certain conditions.

With this, I have also created an interactive 8-bit ALU as a playground below. You can play with it to your heart's content and can email me for further improvements or to report any bugs encountered.

{{< circuit src="https://aksa-os.pages.dev/playground/alu?circuit=alu-8-bit" title="Interactive 8-bit ALU" >}}

Okay, with this and the shifter logic, we have all our arithmetic and logical operations completed. Next, what I will do, honestly, I don't know yet. Till then, have a good one.

Cheers.

Om Namaha Shivaya.