+++
date = '2026-08-06T10:07:19+05:45'
draft = false
title = 'Day 2'
+++

I was very excited to learn how the heck, using just logic gates, we can build memory to store bits. It turns out that, using just two NOR gates, we can store 1 bit. The two NOR gates are configured such that one input of the gates is the set and reset input respectively, whereas the other inputs are connected to the output of the opposite gate. Now, connecting the input of one gate to the output of the other gate creates a kind of feedback loop by which the previous state of the circuit determines or influences the current state. It's the nature of a feedback system.

Now there are two outputs from this circuit configuration, and the two outputs are always opposite to each other. They are denoted by Q and Q'. Similarly, the set and reset inputs of the circuit should also always be opposite, such that when set is 1, reset should be 0, so that the output Q is 1 whereas Q' is 0. For the case where set is 0, reset should be 1, so that the output Q is 0 and Q' is 1.

Now, by combining these individual circuits, also called latches, we can make ourselves an 8-bit (1-byte) register, a 32-bit register, and so on, as per our needs. For me, implementing this system was quite easy because we could mimic the feedback loop of the circuit in our machine. So I used variables to store the values, such as:
```ts
export class register32 {
    private q: Bit32;

    constructor() {
        this.q = Array.from({length: 32}, () => 0 as Bit) as Bit32;
    }

    get() {
        return this.q;
    }

    set(dataIn: Bit32, clk: Bit) {
        if (clk === 0) return;
        this.q = dataIn;
    }

    clear() {
        this.q = this.q.map(() => 0 as Bit) as Bit32;
    }
}
```
You can see above that I have implemented a class for the register, stored the bits in an array, and exposed methods for getting, setting, and clearing the register. The clk parameter of the set method is the clock signal. I did this for synchronization purposes, but for a simulation I am still confused whether I should apply it. And if yes, should it be enforced on every circuit? If not, I will remove it.

But either way, all the code I have implemented till now, and everything I implement in the future, is subject to change. In such cases, I will try to explain the updated parts of the code in this series.

I think we should keep it short for today. Honestly, I wanted to show and explain the RAM that I have implemented, but I am still not sure about its final implementation. Tomorrow I will discuss that with a clearer implementation.

Till then, cheers.

Have a good one.

Om Namah Shivaya.