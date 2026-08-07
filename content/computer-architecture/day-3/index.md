+++
date = '2026-08-07T09:11:35+05:45'
draft = true
title = 'Day 3'
+++

Yes, I have implemented RAM, first I want you to have a look at it:
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
While on the register where I used the array variable to store the values, the RAM uses SharedArrayBuffer. Its the same as ArrayBuffer which is raw binary buffer. The array buffer is quite similar to the real memory as its also situated on the real memory one after another adjecent to eachother like a real memory, thus is has good speed which accessing and manipulating the buffer. Now why SharedArrayBuffer instead of the plain ArrayBuffer is beacause while ArrayBuffer can mimic memory in a single core/thread, the shared buffer as its name can be shared accross multiple cores and workers simultaneously.

As per now I will be implementing a single core, but I will be implementing the CPU in a dedicated worker while the shared buffer in initialized in the main thread, this approach helps me in two ways. First of all all the computations and logics of the cpu will be on a different thread then that of the main thread which wont lag the main thread and the UI, and my other goal is to implement many cpu cores in the future, and this approach will make it earier to do it as all the cores can have simultaneous access to the sharedArrayBuffer i.e. the memory.

In the constructor I have initialized the shareArrayBuffer with 1 MB of space. The maximum RAM space I have defined here is 2 ** 32, that is 4GB, becuse as we have 32 bits as the address space/lane the system can only address 4 GB worth of data.

The read and write methods are quite straightforward. I have decided that the memory implementation sholud be simple and give 1 byte of data per address. Now the addressToIndex private method is quite interesting as this maps the binary type that I have implemented to index on the arrayBuffer. But the important part of this method is the error throwing. As we have memory of limited space, if we try to index/address the memory past its capacity then a exception is raised to the CPU about that particular error condition.

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
I just have created a enum with all the hardware level exceptions. Right now I can only think of two, Memory Fault that is the indexing of invalid address which I had used, the other is also related to the addressing of the 3-4 bytes of memory but we will be implementing it latter. Now I just extented the generic Error class to have my own dedicated error thrown on hardware expections. Its simple and later we could add more to it so that it can be a gret debugging tool or atleast assistance for the debugger.

The other helper funtions you see which are decimalToBinary and binaryToDecimal is as its names says as I have implemented is sch as:
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
I think these functions dont need any explainations. Ok I think I have described enough of my doing for today. I dont know what I will be implementing next to be honest. One thing I will do is, first I will read a bunch of books relating to computer architecture and come back to you tommorow. Sounds like a plan right.

Till then, Cheers.
Have a good one.

Om Namaha Shivaya.