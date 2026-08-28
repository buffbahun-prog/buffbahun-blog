+++
date = '2026-08-27T09:16:43+05:45'
draft = false
title = 'Day 11 | Logical/Arithematic Shifter and Rotator'
+++

It's been long, I guess the shifter made me shift the day when I should have posted. But, not going to lie, I was skimming through both the logical and arithmetic shift operations and the rotation operations too. I have used mathematics and the inversion law to normalize the bit position and have just a single left shifter/rotator circuit for both directions. But there was just one more operation where I had to really think deep and analyze: rotate through carry.
Looks simple, right? It's just the rotation with an extra carry bit included. Yes, indeed that's true, but I had some problem with the shift amount. I will explain why.
Now suppose you are rotating an 8-bit data, so we can rotate it in 8 unique positions, i.e., 7 total rotations, after which it repeats again. For this, just the first 3 bits of the 8-bit shift amount are required, and the other 5 most significant bits we can discard. Why? It's because after 111, if we increment it by 1 we get 000 (1 is the carry). Now, after rotating the 8-bit data 7 times, the 8th rotation is the exact same as the 0th one. You can clearly see it cycle from 0 to 7 repetitively.
This works great with the rotation operation. But for rotate with carry operation, there is one more extra unique rotation that can be done due to the extra carry bit. So from our previous example, the data is now actually 9 bits (8 data bits + 1 carry bit). So there are 0000 to 1000 total rotations, i.e., 8 total rotations. Now, the modulus operation that we got above nicely — this can't be done here, as if we discard the 4th one we don't have all the possible shift amounts, so we have to keep it. But now how can we reduce 1001 to 0000, 1010 to 0001, and so on, so that it works out nicely.
Let's look closely at what shift amounts need to be converted:
```text
1001 same as 0000
1010 same as 0001
1011 same as 0010
1100 same as 0011
1101 same as 0100
1110 same as 0101
1111 same as 0110
```
Look closely at the above two values, then you see the first value is the sum of the second value + 1001. Now, from shift amount 1011, if we want to get the normalized/reduced shift value, we simply can subtract 1011 with 1001. This gives the normalized shift amount. For the subtraction there are just two conditions: the 4th bit must be 1 and not 1000 (it's the last rotation). With this, taking the first 4 bits of the 8-bit shift amount, then with the above conditions evaluated, we can get the normalized shift amount.
You have noticed that even after this normalization of shift amount, while in the case of rotation it plays out nicely and correctly because even if we discard the 5 last bits, it doesn't play that well in the case of rotation with carry. So my plan is making good documentation in the ISA which we will develop later, putting this constraint such that the 6 least significant bits of the shift amount are only considered (my machine is 32 bits, thus (5 + 1 (rotation with carry)) shift bits).
So let's have a look at the implementation then:
```ts
// controlBits -> [left/right, shift/rotate, without-carry/with-carry]
//               [0/1,        0/1,          0/1]
//
// 000 -> no operation
// 001 -> left shift
// 010 -> left rotate without-carry
// 011 -> left rotate with-carry
// 100 -> right shift arithmetic
// 101 -> right shift
// 110 -> right rotate without-carry
// 111 -> right rotate with-carry

export function shiftRotate32(
    data: Bit32,
    shiftBy: Bit32,
    carryBit: Bit,
    controlBits: Bit3,
): { result: Bit32; carry: Bit } {

    // ------------------------------------------------------------
    // Control
    // ------------------------------------------------------------

    const [shiftDirection, rotate, rotateWithCarry] = controlBits;

    const operationEnabled = orGateNInp(controlBits);

    const arithmeticShift = andGateNInp([
        shiftDirection,
        inverter(rotate),
        inverter(rotateWithCarry),
    ]);

    const signBit = data[0];

    const rotateWithCarryEnabled = andGate(rotate, rotateWithCarry);
    const rotateRightWithCarry = andGate(
        shiftDirection,
        rotateWithCarryEnabled,
    );

    const shiftAmount = normalizedShiftAmt(
        shiftBy,
        rotateWithCarryEnabled,
    );

    // ------------------------------------------------------------
    // Normalize input direction
    //
    // The barrel stages always perform their operation in the
    // same direction. For a right operation, reverse the data
    // before entering the barrel shifter.
    // ------------------------------------------------------------

    const directionNormalizedData = data
        .map((bit, index) =>
            mux2To1(
                bit,
                data[data.length - (index + 1)],
                shiftDirection,
            )
        )
        .map((bit, index, normalizedData) =>
            mux2To1(
                bit,
                index >= normalizedData.length - 1
                    ? carryBit
                    : normalizedData[index + 1],
                rotateRightWithCarry,
            )
        ) as Bit32;

    let stageCarry = mux2To1(
        carryBit,
        data[data.length - 1],
        rotateRightWithCarry,
    );

    // ------------------------------------------------------------
    // Barrel shifter stage
    // ------------------------------------------------------------

    const barrelStage = (
        shiftAmount: number,
        index: number,
        inputData: Bit32,
        stageEnabled: Bit,
    ): Bit => {

        const fillStartIndex = inputData.length - shiftAmount;
        const isFillPosition = index >= fillStartIndex;

        // --------------------------------------------------------
        // Shift
        // --------------------------------------------------------

        const shiftFillBit = andGate(
            arithmeticShift,
            signBit,
        );

        const shiftedBit = isFillPosition
            ? shiftFillBit
            : inputData[index + shiftAmount];

        // --------------------------------------------------------
        // Rotate
        // --------------------------------------------------------

        const rotatedBit = isFillPosition
            ? mux2To1(
                inputData[index - fillStartIndex],
                index === fillStartIndex
                    ? stageCarry
                    : inputData[index - fillStartIndex - 1],
                rotateWithCarryEnabled,
            )
            : inputData[index + shiftAmount];

        // --------------------------------------------------------
        // Select shift or rotate
        // --------------------------------------------------------

        const transformedBit = mux2To1(
            shiftedBit,
            rotatedBit,
            rotate,
        );

        // --------------------------------------------------------
        // Enable / bypass this barrel stage
        // --------------------------------------------------------

        return mux2To1(
            inputData[index],
            transformedBit,
            stageEnabled,
        );
    };

    // ------------------------------------------------------------
    // Barrel shifter
    //
    // Stages:
    //
    //   1 -> 16 -> 8 -> 4 -> 2 -> 1
    //
    // This ordering allows a shift of 32 to be represented as:
    //
    //   1 + 16 + 8 + 4 + 2 + 1 = 32
    //
    // which is required for rotate-through-carry.
    // ------------------------------------------------------------

    // 1-place shifter / rotator
    const shift1Data = directionNormalizedData.map(
        (_, index, inputData) =>
            barrelStage(
                1,
                index,
                inputData as Bit32,
                shiftAmount[0],
            )
    );

    stageCarry = mux2To1(
        stageCarry,
        directionNormalizedData[0],
        shiftAmount[0],
    );

    // 16-place shifter / rotator
    const shift16Data = shift1Data.map(
        (_, index, inputData) =>
            barrelStage(
                16,
                index,
                inputData as Bit32,
                shiftAmount[1],
            )
    );

    stageCarry = mux2To1(
        stageCarry,
        shift1Data[15],
        shiftAmount[1],
    );

    // 8-place shifter / rotator
    const shift8Data = shift16Data.map(
        (_, index, inputData) =>
            barrelStage(
                8,
                index,
                inputData as Bit32,
                shiftAmount[2],
            )
    );

    stageCarry = mux2To1(
        stageCarry,
        shift16Data[7],
        shiftAmount[2],
    );

    // 4-place shifter / rotator
    const shift4Data = shift8Data.map(
        (_, index, inputData) =>
            barrelStage(
                4,
                index,
                inputData as Bit32,
                shiftAmount[3],
            )
    );

    stageCarry = mux2To1(
        stageCarry,
        shift8Data[3],
        shiftAmount[3],
    );

    // 2-place shifter / rotator
    const shift2Data = shift4Data.map(
        (_, index, inputData) =>
            barrelStage(
                2,
                index,
                inputData as Bit32,
                shiftAmount[4],
            )
    );

    stageCarry = mux2To1(
        stageCarry,
        shift4Data[1],
        shiftAmount[4],
    );

    // 1-place shifter / rotator
    const finalShiftData = shift2Data.map(
        (_, index, inputData) =>
            barrelStage(
                1,
                index,
                inputData as Bit32,
                shiftAmount[5],
            )
    );

    stageCarry = mux2To1(
        stageCarry,
        shift2Data[0],
        shiftAmount[5],
    );

    // ------------------------------------------------------------
    // Restore original direction
    //
    // The barrel shifter operates in the normalized direction.
    // Reverse the data again when the requested operation is right.
    // ------------------------------------------------------------

    const directionRestoredData = finalShiftData
        .map((bit, index, transformedData) =>
            mux2To1(
                bit,
                transformedData[transformedData.length - (index + 1)],
                shiftDirection,
            )
        )
        .map((bit, index, restoredData) =>
            mux2To1(
                bit,
                index >= restoredData.length - 1
                    ? stageCarry
                    : restoredData[index + 1],
                rotateRightWithCarry,
            )
        ) as Bit32;

    stageCarry = mux2To1(
        stageCarry,
        finalShiftData[finalShiftData.length - 1],
        rotateRightWithCarry,
    );

    // -----------------------------------------------------------
    // If shift and shift amount greater then 011111 then
    // arthematic right shift all bits sign bits
    // normal shift all 0
    // -----------------------------------------------------------

    const isShift = inverter(rotate);
    const fillValue = andGate(arithmeticShift, signBit);
    const shiftOverflowData = directionRestoredData.map(
        bit =>
            mux2To1(
                bit,
                fillValue,
                andGate(
                    isShift,
                    orGateNInp(
                        shiftBy.slice(0, 27)
                    )
                )
            )
    );

    // ------------------------------------------------------------
    // Operation enable
    //
    // 000 means no operation, so preserve the original input.
    // ------------------------------------------------------------

    const result = shiftOverflowData.map(
        (bit, index) =>
            mux2To1(
                data[index],
                bit,
                operationEnabled,
            )
    ) as Bit32;

    // Carry only changes for rotate-through-carry.
    const finalCarry = mux2To1(
        carryBit,
        stageCarry,
        rotateWithCarryEnabled,
    );

    return {
        result,
        carry: finalCarry,
    };
}
```

I won't go into much detail, but the general flow is:

1. The three control bits which constitute 8 possible operations are implemented with the proper control description and the required sections in the comment above the function and the first few lines in the beginning of the function.

2. The function normalizedShiftAmt is used as we have already discussed; its implementation and constraints are as follows:
```ts
function normalizedShiftAmt(
    shiftBy: Bit32,
    rotateWithCarry: Bit,
): Bit6 {

    const shiftAmount = shiftBy.slice(32 - 6) as Bit6;

    shiftAmount[0] = andGate(
        rotateWithCarry,
        shiftAmount[0],
    );

    const isShiftBy32 = andGate(
        shiftAmount[0],
        norGateNInp(shiftAmount.slice(1)),
    );

    const shouldModulo33 = andGateNInp([
        rotateWithCarry,
        shiftAmount[0],
        inverter(isShiftBy32),
    ]);

    const negative33OrZero = [
        0,
        shouldModulo33,
        shouldModulo33,
        shouldModulo33,
        shouldModulo33,
        shouldModulo33,
    ] as Bit6;

    const moduloResult = bitAdder6(
        0,
        shiftAmount,
        negative33OrZero,
    )[0];

    // Normal shift/rotate:
    //     32 -> 0
    //
    // Rotate-through-carry:
    //     33 -> 0
    //
    // Instead of:
    //     32 -> 16 -> 8 -> 4 -> 2 -> 1
    //
    // We use:
    //     1 -> 16 -> 8 -> 4 -> 2 -> 1
    //
    // so that shift-by-32 becomes:
    //     1 + 16 + 8 + 4 + 2 + 1 = 32

    return moduloResult.map(
        bit => orGate(isShiftBy32, bit)
    ) as Bit6;
}
```
As discussed already, we do the subtraction operation to get the desired normalized value with the particular constant value. Now, another thing which is important to understand is the last return part along with the above comments. When the operation is rotate through carry and the shift amount is exactly 32, then all the 6 shift amount bits are converted to 1. Why I have done this, I will explain shortly.

3. In case of right rotate/shift, we inverse the bit position of the data so that it can be operated just with the left shifter/rotator we have implemented without needing additional right shift/rotate circuitry, and re-inverse the transformed data back to its original bit positions.

4. Mathematically, with 16, 8, 4, 2, 1 amount shifter/rotator circuits, with combinations of these circuits we can get all the possible shift amounts within this range, i.e., 0 to 31. So if the shift amount is 00000, none of the shifter/rotators are activated, thus no shifting/rotating. Now, if we want to shift by 21 (10101), which is 16 + 4 + 1, aligning the bit positions to the shifter such as:
```text
shifter/rotator               16  8  4  2  1
shift amount                  b0 b1 b2 b3 b4
example 21 = (16 + 4 + 1)  --> 1  0  1  0  1
        | (binary value)   |  16  0  4  0  1
        v                  |
      10101-----------------
```
The above circuitry is for 31 total shift/rotate operations which fits nicely for our simple shifter/rotator, but for rotate-with-carry there is an extra shift required, that is 31 + 1, which is achieved by one additional shift amount 1 shifter/rotator, and we put it at the beginning as 1 16 8 4 2 1. This is why all the bits were set to 1 in the normalize function when the operation is rotate-with-carry and the rotate amount is exactly 32 (1+16+8+4+2+1).

5. The barrel shift function simulates all the barrel left shifter/rotator circuitry with just the required circuitry enabled according to the shift amount.

6. Along with the data bit rotating, the carry bit is also shifted and calculated according to the shift amount and returned with the function along with the final data result.

7. As we have 6 bits for shift amount, but as all the 6 bits are only required by the rotate-with-carry, and on shifting beyond and equal to 31 transforms the data with the fill bit (0 or sign bit in case of arithmetic shift), the output data is accounted as such. And finally, the transformed data along with the possibly shifted carry bit is returned on circuit enabled; in case not, the data and carry are returned as is.

I think all that is to be done with the shifter/rotator operation and circuit is sufficiently simulated by this function. Now I will have a large test case coverage, and if any bug is found, I will update the implementation with the corrected one.

I think this much for today will do the job. With my current study and research I am doing, I am trying to make specific operations done with the arithmetic unit and logic unit and trying to figure out the appropriate inputs, outputs, along with the basic operations they should be doing so that I can modularize these two units and it will be easier for me while creating the ALU circuit. I will be posting the updates as soon as I have some progress worthy of sharing.
Till then have a good one.
Cheers.

Om Namaha Shivaya.