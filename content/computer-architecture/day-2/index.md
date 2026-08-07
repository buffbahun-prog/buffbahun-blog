+++
date = '2026-08-06T10:07:19+05:45'
draft = true
title = 'Day 2'
+++

I was very excited how the heck using just logical gates we can build a memory to store bits. It turns out, just using two NOR gates we can store 1 Bit. The two NOR gates are configured such that, one input of the gates are set and reset respectively whereas the next inputs are connected to the output of the other gate. Now connecting the input of one gate to the output of another gate creates a kind of feedback loop by which the previous state of the circuit deremines/influences the current state. It the nature of the feedback system.

Now there are two oytputs from this circuit configurations and the two outputs are always opposite to each other and are denoted by Q and Q'. Similarly the two set and reset inuts of the circuit also should always be opposite, such that when set is 1, reset should be 0, so that the output Q is 1 whereas Q' is 0. Foe the case of set is 0 reset sould be 1, so that the output Q is 0 and Q' is 1.

Now combining this individual circuit also called latch, we have make ourself 8 bit/ 1 byte register, 32 bits register and so on as per our need. For me to implement this system was quite easy as we could mimick the feedback loop of the circuit in our machine so that I have used variables to store the values. Such as:
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

You can see above, I have implemented a class for the resiter and stored bits in an array and exposed methods for geting, seting and clearning the register. The clk on the set methods parameter is the clock output. I did this for synchronization purpose, but for a simulation I am confused whether should I apply it and if yes it should be enforsed on every circuit and if no I will remove it. But either way all the codes I have implemented till now and on the future are subjected to change, and in such cases I will try to explain the updated part of codes in the series.

I think we should keep it short for today. Honestly I wanted to show and explain the RAM I had implemented but I am still not sure the final implemnetation of it, and tommorow I will discuss that with a clear implementation.

Till then Cheers.
Have a good one.

Om Namaha Shivaya.