# AISOC notes
1. How much memory mapped address space is allowed for your IP when building SOC?
> For AISOC platform, 4 KB is maxima. Any memory space allocation more than that cannot be used due to the limited address space of the AXI interface. For aiip1 and aiip2 the address space is 0xF9000_0000~0xF900_FFFF
2. Can we use SRAM within our IP blocks?
> Yes you can. However, the SRAM size you use should be smaller than 4KB otherwise AXI cannot help address every location in the SRAM you just defined. Note only 1024 * 32 registers or space can be used for direct access or seen by the CPU. Anything or data you want to move more than that, please put it on the surrounding SRAM on the bridge or DRAM.
3. What is the datapath width industry usually define for their own IP?
> Usually the maximum datapath width is defined according to the word of your CPU. For example 32-bit machines, uses a maximum of 32 bits datapath.
4. Who governs the whole system in this AISOC platform and in industry?\
> In usual case or in smaller companies, NIC-400 is used for the communications between different IP blocks within the SOC.

# Summary
1. AMBA bus Fabric and the AXI AHB protocals is a must if you want to cooperate with others within the idustry.
2. The protocals actually allows different bit transfer and different protocals communications, which is extremely powerful
3. NIC-400 is the bus TSRI used.
4. They support read write response for each IPs, also the in/out registers following the AXI or AHB protocals can get controlled by C code from CPU.
5. Interrupt can also be implemented.
6. Verilog for this bus can be automatically generated using the AMBA bus generator.

# DMA access
0. You provide registers location you want to transfer, instruct DMA how much data you want to transfer
1. Verilog code need to be used s.t. we can instruct the DMA where to transfer
2. To move how much data, determined by C code.
3. transfer_go is automatically set by C, enabling the transfer.(No need to tell CPU to do the job. DMA does it all)

## Registration
1. Lab materials can be found if you want to do further research on this platform.

## Memory space and MMIO
1. In AISOC, MMIO is used to tell CPU where the IP blocks are located at.

## aiips
1. 2 aiips are allowed for your design and if you want more peripherals, more I/O components can be attached.
2. Each IPs has Control(Input) Registers (Slave) and Status Registers(Output) (Masters).
3. One interrupt is reserved for all IP block.

## AHB protocals
1. AHB bus is slow and are usually used for initial value setting of registers for most of the IPs blocks

## AXI
1. AXI supports high burst mode which is the esssential reason for high-speed data transfer.
2. Burst mode INCR4,WRAP4
3. In WRAP memory address is not continuos.

## DOC
1. To quickly understand the behaviour of the IPs, study how the in/out registers this IP provided and how it interacts with the world.

## ICCM
1. Hex files can be generated and load into the Cache memory for booting the whole system.
2. ICCM is divided into 2 banks. So there is going to be hex0 and hex1.

## Simulation
1. When running simulation on SOC, suggest only run for a while.
2. Imaging you are processing an image, suggest running 1~30 lines of this image first. If you try to run 1~1080 lines of the image, it is going to take forever for you to find bug...

## Interrupts
1. In AISOC system, verilog needs to be written to implement the interrupt flag, however, CPU pulls it down after the request has been processed.

## Debug and Bus interface
0. Your HW interrupting, CPU clears the interrupt flag for you.
1. After initiating interrupt, exception handler are used to process the interrupt request.
2. Usually some output registers are always reserved for debugging particular signals, otherwise you cannot change the design after tape out.
3. Note for AXI and AHB a 1 cycle delay is usually needed for data transfer.
4. If the register width you want to use it less than 32 bits, remember to zeropad the other space to enable the use of in/out register of IP blocks.
5. GPIOs can be used in RTL sim to indicate info about the SOC. Tell you whether the system starts, perform some actions or complete processing.
6. You dont need to build the GPIO yourself, simply find the memory map of the system, find how to access the registers of GPIO, then write the embbeded C code, you can use this GPIO to do all sorts of things you want.
7. SRAM value can also be set or move from a place to another using C code.
8. Note in embbeded system, the following code is ALWAYS needed after you executing your code! Otherwise the system crash!
```C
    while(1){}

```
