+++
date = '2026-08-04T14:52:46+05:45'
draft = false
title = 'Day 0'
+++

The other day I was studying some algorithms used in file system and buffers that the unix system used. I was fasinated by the cleverness and engineering intiutiveness for the implementation. The engineering inside of me scremed in excitement, "Holy shit! This is the real deal". Now I have a crazy idea. Why not build my own operating system form scratch. Sounds insane right? Yes this scares me, I have no idea whatso ever, but with the right knowledge and most of all with right mindset I think this can be accomplished though it may take years and years. This is why you are reading this as this is me documenting my journey. So lets dive into this voyage.

I have created a repo ()[], you may be shoked you see typescript here. Double shocked on seeing html file. Inside the src you will see directory named virtual-machine. Yes virtual machine. Dont get any confused I will explain why. My goal being creating a operating system, I have to eventually run it on a actual machine/microprocessor. But till then I have lot to implement and test. So I though whah if I implement my OS on the browser first so that I can masively test algorithms and implementations. But the browser is just a user application so I am implementing a simple virtual computer inside browser.

See my intiution is straight forward. To properly implement my OS, I need to have a good foundation on the architecture and  hardware side of the computer especially the processor, memory and storage. Plus I will add lot of debugging principles from the hardware level so it will be way easier to debug my OS. Again, I am a novice on both architecture and hardware. We will lern and implement along the way.

Let us start from the building block of computers. The humble logical gates. I have my implementations below:

```ts
export function andGate(bit0: Bit, bit1: Bit): Bit {
    if (bit0 === 1 && bit1 === 1) return 1;
    else return 0;
}

export function orGate(bit0: Bit, bit1: Bit): Bit {
    if (bit0 === 1 || bit1 === 1) return 1;
    else return 0;
}

export function inverter(bit0: Bit): Bit {
    if (bit0 === 0) return 1;
    else return 0;
}

export function nandGate(bit0: Bit, bit1: Bit): Bit {
    return inverter( andGate(bit0, bit1) );
}

export function norGate(bit0: Bit, bit1: Bit): Bit {
    return inverter( orGate(bit0, bit1) );
}

export function xorGate(bit0: Bit, bit1: Bit): Bit {
    return andGate(
        orGate(bit0, bit1),
        nandGate(bit0, bit1),
    );
}
```
The Bit type I have implemented is simply:

```ts
export type Bit = 0 | 1;
```

Just as arethematic operators such as addition'+', minus'-', the logical gates are devices that does binary operations of inputs and return a binary output. The three major logical gates are and, or and not gate and the binary operations they perform are and, or and inverse operations respectfully. The operations are also called logical operations because as we see we have just two valuse '0' and '1' consistuting the binary numbers they can we though of logical values such as true and false, or on and off. I think you get the idea. Its the mathematical representation of the philoshophical logic.

The AND logical gate performs AND operation on the inputs. Same as we such an in our statements for example when we go to a shop and we say "Please give me a shirt which has colors red 'and' blue". The store person gives you a shirt haveing both red and blue color, the AND operator return Bit value '1' if and only if all the inputs are Bit value '1', else give output '0'. Similarly the OR gate is also similar to our use of daily conversations and statements of having either any of it, the OR operator gives output Bit '1' if any of the input is Bit '1' and Bit '0' only in the case when all the inputs are Bit '0'.

Another fundamental logical gate is the NOT gate, its more of a inverter then a operator as it outputs the flipped value of the input. If input is Bit '0' then output is Bit '1' and vice versa. Now other operator that I have implemented are combinations of these three fundamental gates(AND, OR, NOT). The XOR operator is an interesting one as it outputs Bit '1' when the two inputs are different else on both inputs same it outputs Bit '0'.

I think for today this much will do the job. Tommorow we will try to implement Adder from this logical gates. Till then cheers. Have a good one.

Om Namaha Shivaya.