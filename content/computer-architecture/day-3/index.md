+++
date = '2026-08-07T09:11:35+05:45'
draft = false
title = 'Day 3'
+++

Yes, I have implemented RAM. First, I want you to have a look at it:
```ts
export class RAM {
    private data: SharedArrayBuffer;
    private dataView: Uint8Array;
    private totalBytes: number;

    constructor() {
        // max address lane is 32 bits, so max ram size is 2 pow 32
        this.totalBytes = Math.min(2 ** 32, 1 * 1024 * 1024);
        this.data = new SharedArrayBuffer(this.totalBytes);
        this.dataView = new Uint8Array(this.data);
    }

    getBuffer() {
        return this.data;
    }

    read8(address: Bit32) {
        const dataNum = this.dataView.at(this.addressToIndex(address));
        if (dataNum !== undefined) {
            return decimalToBinary(dataNum, 8) as Bit8;
        }
    }

    write8(address: Bit32, data: Bit8) {
        const dataNum = binaryToDecimal(data);
        this.dataView[this.addressToIndex(address)] = dataNum;
    }

    private addressToIndex(address: Bit32): number {
        const index = binaryToDecimal(address);
        if (index >= this.totalBytes) {
            throw new HardwareExpection(
                HardwareExceptionType.MemoryFault,
                `Invalid address to RAM: 0x${index.toString(16)}`,
                index
            )
        }
        return index;
    }
}
```
Whereas in the register I used an array variable to store the values, the RAM uses SharedArrayBuffer. It is similar to ArrayBuffer, which is a raw binary buffer. The array buffer is quite similar to real memory because it is also laid out contiguously in memory, one byte after another, just like actual RAM. Thus, it provides good performance when accessing and manipulating the buffer.

Now, why SharedArrayBuffer instead of the plain ArrayBuffer? The reason is that while ArrayBuffer can mimic memory in a single thread, SharedArrayBuffer, as its name suggests, can be shared across multiple threads and workers simultaneously.

As of now, I will be implementing a single-core CPU, but I will be implementing the CPU in a dedicated worker while the shared buffer is initialized in the main thread. This approach helps me in two ways. First of all, all the computations and logic of the CPU will run on a different thread than the main thread, so the UI will remain responsive. My other goal is to implement multiple CPU cores in the future, and this approach will make it easier because all the cores can have simultaneous access to the SharedArrayBuffer, i.e., the memory.

In the constructor, I have initialized the SharedArrayBuffer with 1 MB of space. The maximum RAM size I have defined here is 2 ** 32, that is 4 GB, because with a 32-bit address space, the system can address only 4 GB worth of memory.

The read and write methods are quite straightforward. I have decided that the memory implementation should be simple and return 1 byte of data per address.

Now, the addressToIndex private method is quite interesting because it maps the binary type that I have implemented to an index in the array buffer. But the important part of this method is the exception it throws. Since we have a limited amount of memory, if we try to access memory beyond its capacity, an exception is raised to the CPU for that particular error condition.

The way I have implemented this is:
```ts
export enum HardwareExceptionType {
    MemoryFault,
    AlignmentFault,
}

export class HardwareExpection extends Error {
    public readonly type: HardwareExceptionType;
    public readonly address?: number;

    constructor(
        type: HardwareExceptionType,
        message: string,
        address?: number,
    ) {
        super(message);
        this.name = "HardwareExpection";
        this.type = type;
        this.address = address;
    }
}
```
I have simply created an enum with all the hardware-level exceptions. Right now I can think of only two. The first is MemoryFault, which is raised when an invalid memory address is accessed, and the second is AlignmentFault, which is also related to addressing 3- or 4-byte values in memory, but we will implement that later.

I then extended the generic Error class so that I can throw my own dedicated hardware exceptions. It's simple for now, but later we can add more information to it so that it becomes a great debugging tool, or at least a useful assistant for the debugger.

The other helper functions you see, decimalToBinary and binaryToDecimal, do exactly what their names suggest. I have implemented them as follows:
```ts
export function decimalToBinary(num: number, pad: number): Bit[] {
        const bin: Bit[] = [];
        if (num > 2 ** pad - 1) {
            console.error("binary conversion error as padding is less then number");
        }
        while (bin.length < pad) {
            const remainder = num % 2;
            num = Math.floor(num / 2);
            bin.unshift(remainder as Bit);
        }

        return bin;
    }

export function binaryToDecimal(bin: Bit[]): number {
    return [...bin].reverse().reduce((acc: number, cur: number, indx) => acc + (cur * (2 ** indx)), 0);
}
```
I think these functions don't need any explanation.

I think I have described enough of what I did for today. To be honest, I don't know what I will be implementing next. One thing I will do is read a bunch of books related to computer architecture and come back to you tomorrow. Sounds like a plan, right?

Till then, cheers.

Have a good one.

Om Namah Shivaya.