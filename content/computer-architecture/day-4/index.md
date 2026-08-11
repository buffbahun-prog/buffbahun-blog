+++
date = '2026-08-09T11:18:38+05:45'
draft = false
title = 'Day 4 — Program Counter'
+++

As I was studying computer architecture, we implemented logical gates, combined logical gates to build an adder circuit, implemented flip-flops with a combination of logical gates to store a bit of information, and by combining multiple flip-flops, we implemented registers and memory.

We have the tools necessary for computation, but as you can see, without a human punching in every number and setting/resetting circuits, no complex computation is possible yet. Taking multiplication as an example, we need to store one number in memory/register, enter another number as the input to the adder, sum it, then store the value in the accumulator, and repeat the process until the multiplication is complete. It's a hassle. So automation is required.

As we have already implemented a clock, which will work as a synchronizer across the whole circuit, this is indeed a step toward automation. Now, instead of manually inputting every number that we want to compute, we can use RAM to have a set of numbers and also a set of instructions the machine should follow. We can arrange both the code/instructions and the data in an order so that the machine can sequentially read the instructions with their arguments and execute them correctly.

We have also already implemented RAM, and we can visualize its structure as a long array from index 0 to n - 1, where each element contains a byte of data to store. So, we can surely put our program in sequential order, starting from memory index 0 and continuing until we halt, by incrementing the index by 1 or some arbitrary number on each clock cycle.

So, we get the idea for a circuit that, on each clock cycle, gives a value starting from 0, then one number more (which could be one or some constant number) than the previous value. This is called a program counter. The name itself gives the meaning. It is an incremental counter that sequentially gives the next index/address for the next instruction in the program.

Let's have a look at my implementation:
```ts
export class ProgramCounter extends register32 {

    constructor() {
        super();
    }

    increment() {
        const currCounterVal = this.get();
        const incBy = decimalToBinary(4, 32) as Bit32; // Instruction size is 4 bytes
        const nextCount = bitAdderSubstractor32(0, 0, currCounterVal, incBy)[0];
        this.set(nextCount);
    }
}
```
I have extended the 32-bit register because all it is, is a register that stores a 32-bit number, which we intend to use as an address on the RAM. Now, for it to be a counter, I have implemented an increment method that gets the current number/address stored in its register and then updates the register with a constant number incremented. In this case, it is by 4 bytes every clock cycle because I have intended to make the instruction size exactly 4 bytes. So, on every clock cycle, the increment method is called so that we can move through the memory one instruction per cycle.

It is what I did today. I also updated the adderSubstractor function and made it more modular, as the circuit itself is implemented on the machine. I did this so that the different logical parts that we have implemented have a clear boundary. You can view the changes [here](https://github.com/buffbahun-prog/she-is-the-booms/commit/2ac26bad06a4650508ddf895a83d84b5b8f93258#diff-91114871da01094c14ba9ba02f0b56fddd8234a14e291620be8c482ab1b1933e) in the adders.ts file.

I will see you tomorrow, as I now want to study how the circuits are connected to all the logical parts we have built so far, and how it is correlated/mapped to an instruction.

Till then, have a good one.

Cheers.

Om Namah Shivaya.