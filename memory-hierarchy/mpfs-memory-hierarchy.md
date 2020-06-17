# PolarFire SoC Memory Hierarchy

This document provides a brief overview of the PolarFire SoC hardware features related to memory hierarchy and suggested uses of these features.

Please refer to the [PolarFire SoC Microprocessor Subsystem (MSS) User Guide](https://www.microsemi.com/document-portal/doc_download/1244570-ug0880-polarfire-soc-fpga-microprocessor-subsystem-mss-user-guide) for the detailed description of PolarFire SoC.

## Overview
PolarFire SoC uses a classic three level memory hierarchy including:
- 32 KiBytes instruction and data  level 1 caches associated with each U54 application core
- 16 KiBytes instruction cache and 8KiBytes Data Tightly Integrated Memory associated with the E51 monitor core
- 2 MiBytes level 2 cache
- external DDR memory

![Hierarchy](https://bitbucket.microchip.com/projects/FPGA_PFSOC_ES/repos/polarfire-soc-documentation/raw/images/memory-hierarchy/Cache-LIM-Scratchpad-hierarchy.png?at=refs%2Fheads%2Ftemporary-images)

The PolarFire SoC instruction caches can optionally be configured to be used as Instruction Tightly Integrated Memory (ITIM). ITIM is memory mapped to the address space, providing deterministic access to fast memory.
Likewise, L2 cache memory can be configured as either Loosely Integrated Memory (LIM) providing deterministic access to L2 memory or as fast access scratchpad.

![Memory Map](https://bitbucket.microchip.com/projects/FPGA_PFSOC_ES/repos/polarfire-soc-documentation/raw/images/memory-hierarchy/Cache-LIM-Scratchpad-full-memory-map.png?at=refs%2Fheads%2Ftemporary-images)

Cached and non-cached access to external DDR memory can be done through 32-bit or 64-bit addresses through separate memory ranges.
Non-cached DDR accesses can be done through a Write Combine Buffer (WCB) to improve performance by combining multiple accesses into a single burst. 

## L1 Memory Use Cases
The L1 instruction cache memory attached to the U54 processor cores can be used as fast deterministic access Instruction Tighly Integrated Memory (ITIM) instead of cache memory. L1 memory is converted from cache to ITIM by writing to the relevant ITIM memory range associated with each U54 processor core. The ITIM memory can be returned to being used as L1 cache by writing a zero to the first byte above the top of the ITIM region.

ITIM memory is intended to be used for time critical code such as interrupt service routines where fast deterministic operations are important.


## L2 Memory Use Cases
The PolarFire SoC L2 memory can be configured for different use cases where the actual L2 memory blocks are used differently based on processor accessible configuration registers.

### L2 Cache
L2 cache is on-chip memory providing fast access to copies of data and code stored in external DDR memory. The L2 cache increases system performance by decreasing the time required to access the content of recently used DDR external memory.

### Loosely Integrated Memory (LIM)
Loosely Integrated Memory (LIM) is L2 memory accessed with a deterministic access time. LIM memory is not cacheable, meaning that the content of LIM memory will never be cached in processors' L1 cache.
LIM memory is used where deterministic operations are more important than performance. It can also be used for simple bare metal software debug since the bulk of L2 memory (1920KiBytes) is available through the memory map coming out of reset.

### Scratchpad Memory
Scratchpad memory is made up of L2 memory blocks which are made accessible through the Zero Device address range. Scratchpad memory is cacheable. It provides the best execution performances as it is made up of low latency on-chip memory.

### Pinned Memory
Pinned memory is a similar use case to scratchpad memory. It locks the content of specific DDR memory addresses into the L2 cache, preventing eviction of the content of these memory locations back to DDR memory.

### Master Affinity
Master affinity is the ability to limit the eviction of data or code from the L2 cache to specific masters. This can be used to prevent some L2 masters activities from affecting the performance of other masters. It can also be used to allocate a larger L2 cache to one or more masters than others. This feature is used for system performance tuning.


## L2 Memory Configuration

### L2 Cache Memory Structure
The L2 cache is a 16-way set associative cache. It is made up of 4 banks, each bank contains 512 sets, each set contains 16 ways that each contain a 64 bytes block giving us 2 MiBytes of L2 memory.

![Cache Structure](https://bitbucket.microchip.com/projects/FPGA_PFSOC_ES/repos/polarfire-soc-documentation/raw/images/memory-hierarchy/Cache-LIM-Scratchpad-cache-structure.png?at=refs%2Fheads%2Ftemporary-images)

Please refer to the [PolarFire SoC Microprocessor Subsystem (MSS) User Guide](https://www.microsemi.com/document-portal/doc_download/1244570-ug0880-polarfire-soc-fpga-microprocessor-subsystem-mss-user-guide) for the detailed description of the L2 cache hardware.

The address of the memory accessed through the cache is used to determine which bank and set combination will be used to store the 64 byte block accessed from the backing memory. The backing memory is typically external DDR memory. The index of the way that will be used within the set is determined based on the current content and previous use of the various ways within the set. This means that a specific backing memory location can be stored in one of 16 possible ways within a set.

The L2 cache controller control registers can be used to add additional constraints on which cache ways can be used. It can limit the number of ways available within a set or prevent some ways to be used by specific L2 masters. These constraints are applied based on the index of cache way within a set. These constraints are applied across all sets and banks. As such, constraints are applied to all ways of the same index, i.e. 128KiBytes. The rest of this document refers to a specific way index across all sets and banks and not a single 64 byte way within a set when discussing ways configuration.


### Configuration Registers of Interest
The L2 memory is configured through a small number of configuration registers that can be written by any processor core. Although any processor core can write these registers, by convention, only the E51 core will modify these registers during system startup to setup a consistent memory hierarchy configuration for all cores in the system.

|  Address   |  Name       |  Description                                                                    |
| ---------- | ----------- | ------------------------------------------------------------------------------- |
| 0x02010000 | Config      | Read-only Information about the cache structure (banks, ways, sets, block size) |
| 0x02010008 | WayEnable   | Index of the largest way which has been enabled. May only be increased.         |
| 0x02010200 | Flush64     | Flush cache block, 64-bit address                                               |
| 0x02010240 | Flush32     | Flush cache block, 32-bit address                                               |
| 0x02010800 | WayMask0    | Platform DMA master way mask register                                           |
| 0x02010808 | WayMask1    | FPGA fabric AXI4 port 0 master way mask register                                |
| 0x02010810 | WayMask2    | FPGA fabric AXI4 port 1 master way mask register                                |
| 0x02010818 | WayMask3    | FPGA fabric AXI4 port 2 master way mask register                                |
| 0x02010820 | WayMask4    | FPGA fabric AXI4 port 3 master way mask register                                |
| 0x02010828 | WayMask5    | E51 DCache MMIO master way mask register                                        |
| 0x02010830 | WayMask6    | E51 ICache  master way mask register                                            |
| 0x02010838 | WayMask7    | U54_1 DCache master way mask register                                           |
| 0x02010840 | WayMask8    | U54_1 ICache  master way mask register                                          |
| 0x02010848 | WayMask9    | U54_2 DCache master way mask register                                           |
| 0x02010850 | WayMask10   | U54_2 ICache  master way mask register                                          |
| 0x02010858 | WayMask11   | U54_3 DCache master way mask register                                           |
| 0x02010860 | WayMask12   | U54_3 ICache  master way mask register                                          |
| 0x02010868 | WayMask13   | U54_4 DCache master way mask register                                           |
| 0x02010870 | WayMask14   | U54_4 ICache  master way mask register                                          |

Please refer to the [PolarFire SoC Microprocessor Subsystem (MSS) User Guide](https://www.microsemi.com/document-portal/doc_download/1244570-ug0880-polarfire-soc-fpga-microprocessor-subsystem-mss-user-guide) for the complete register descriptions.

#### Way Enable Register
This register controls how many cache ways are enabled as cache memory. It contains the index of the last enabled cache way. Cache ways not enabled are used as LIM memory.

This is typically the first register configured to partition the 2MiBytes of L2 memory between cache memory and LIM memory.

#### WayMask Registers
WayMask registers are used to specify which masters can evict from specific cache ways. Setting these WayMask registers allow to configure the following use cases:

- scratchpad
- master affinity
- pinning

The lower 16 bits in a WayMask register represents one of the 16 cache ways. Setting a 1 allows the master for that WayMask register to cause the content of that cache way to be evicted from L2 memory. For example, setting bit 3 of WayMask10 to 1 will allow U54_2 to evict from the fourth cache way and replace the content of that way with instructions it fetches.

Writing a zero in WayMask bits does not imply that a master cannot access memory from the underlying L2 cache way memory. It instead means that the WayMask master cannot affect the content of that cache way.

#### Flush Registers
Flush64 and Flush32 registers are available to flush the content of a specific physical address out of the cache to the DDR.

Flush64 takes a 64-bit address for the address of the memory location to flush out of the cache.

Flush32 takes a 32-bit address for the address of the memory location to flush out of the cache. The physical address flushed is that 32-bit value shifted left by 4 bits.

### Configuration Mechanisms
The L2 memory is organized into 16 cache ways. Each cache way is 128 KiBytes.

The L2 memory is configured through the WayEnable register and 15 WayMask registers.

The WayEnable register controls the number of L2 memory ways that are enabled as cacheable memory for the L2 masters. It contains the index of last available way starting from zero. In other words, it is the number of enabled cache ways minus one. This number can only be increased as decreasing it would  remove available memory from the cache hierarchy and cause cache coherence to fail.

The WayMask registers controls which L2 master can evict from specific ways' content. The content of cache ways is evicted as a result of cache content management or explicit flushes through memory barrier instructions (e.g. fence.i) or writes to the Flush64/Flush32 of the L2 Cache Controller control registers

#### L2 Memory After Reset
Coming out of reset there is only one cache way enabled. The remaining L2 memory is configured as Loosely Integrated Memory (LIM).

The WayEnable register is set to zero coming out of reset indicating that only one cache way is enabled. This also means that the remaining 15 cache ways are allocated to LIM. Given that each cache way is 128 KiBytes, PolarFire SoC comes out of reset with a 128KiBytes cache and a 1920 KiBytes LIM.

![L2 Reset Configuration](https://bitbucket.microchip.com/projects/FPGA_PFSOC_ES/repos/polarfire-soc-documentation/raw/images/memory-hierarchy/Cache-LIM-Scratchpad-reset.png?at=refs%2Fheads%2Ftemporary-images)

Note: The _WayMasks Registers_ in the diagram above is the logical OR of all 15 WayMask registers. This diagram assumes that all WayMask registers contain the same value.

#### L2 Memory Split Between Cache and LIM
The most basic level of L2 cache configuration is partitioning the size of the L2 memory between cache and LIM usage. This is achieved by setting the value of the WayEnable register, leaving all bits in the WayMask registers set to one. This provides the ability to partition the L2 cache memory between cache and LIM with a granularity of 128KiBytes since each way is 128KiBytes long,

Set the value of the WayEnable to the number of ways you want enabled as cache minus one. Leaving all bits in the WayMask registers set to one means that all L2 masters will be able to use all the enabled cache ways. WayMask register bits for ways not enabled through the WayEnable register have no effect.

![Cache LIM Partition](https://bitbucket.microchip.com/projects/FPGA_PFSOC_ES/repos/polarfire-soc-documentation/raw/images/memory-hierarchy/Cache-LIM-Scratchpad-simple-split.png?at=refs%2Fheads%2Ftemporary-images)

Note: The _WayMasks Registers_ in the diagram above is the logical OR of all 15 WayMask registers. This diagram assumes that all WayMask registers contain the same value.

#### Scratchpad Memory
Cache ways enabled as cache using the WayEnable register can be further configured to be accessible through the the Zero Device memory address range as a general purpose scratchpad memory by careful manipulation of the WayMask registers.

Once configured, you will see that WayMask registers are used to prevent eviction from a number of cache ways. All WayMask registers will have the same set of cache ways non-masked (bits set to zero). This means no master will be able to evict the content of these ways through cache management. The initial content of these L2 memory ways will have been set by the configuration algorithm.

The scratchpad configuration algorithm’s general method is to use one master, say Master S, to prevent eviction from the ways we want to use as scratchpad. To do this Master “S” forces the ways content (located in the cache way) to be mapped to the Zero Device address range by writing to the Zero Device. Master "S" then prevents itself from affecting the content of the cache way that is being mapped to the zero device by unmasking its own access to the cache way being used as a scratchpad.

The algorithm for setting up scratchpad memory is detailed in the L2 Cache Controller section of the [PolarFire SoC Microprocessor Subsystem (MSS) User Guide](https://www.microsemi.com/document-portal/doc_download/1244570-ug0880-polarfire-soc-fpga-microprocessor-subsystem-mss-user-guide). You can also refer to the bare metal library's MPFS-HAL implementation for a working example.


![Scratchpad](https://bitbucket.microchip.com/projects/FPGA_PFSOC_ES/repos/polarfire-soc-documentation/raw/images/memory-hierarchy/Cache-LIM-Scratchpad-scratchpad.png?at=refs%2Fheads%2Ftemporary-images)

Note: The _WayMasks Registers_ in the diagram above is the logical OR of all 15 WayMask registers. This diagram assumes that all WayMask registers contain the same value. Using different values for WayMask registers allows fine-grained control over which L2 masters can evist from specific ways. This can be used for tuning system performance.

#### Master Affinity
The WayMask control registers can be used to control an L2 master's affinity to specific cache ways. The examples above all assumed that all WayMask registers had the same value. We can partition the enabled cache ways between masters, allowing more cache for a set of masters than others based on system performance requirements or preventing performance interference from a system's task associated with a specific master.

For example, we could reserve two ways to handle data shared between the FPGA fabric and one hart processing that data. We could also allow the DMA controller to evict from these same ways to populate buffers modified by the FPGA fabric. We could reserve 3 ways for one of the processor core's data and another 3 ways for its executable while leaving the remaining ways to be shared by the remaining processor cores.

![Affinity](https://bitbucket.microchip.com/projects/FPGA_PFSOC_ES/repos/polarfire-soc-documentation/raw/images/memory-hierarchy/Cache-LIM-Scratchpad-affinity.png?at=refs%2Fheads%2Ftemporary-images)

#### Mixing Use Cases
The L2 configuration use case presented above are not all mutually exclusive. The two mutually exclusive use cases are Cache and LIM. A cache way is either cacheable or used as LIM. Scratchpad and affinity can be combined to reserve fast access memory for use by a limited set of L2 masters. One example of this is using L2 scratchpad for time constrained operations involving the FPGA fabric and one or more processor cores.

#### Effect of L2 Cache Controller Configuration on the Memory Map
Non-masked L2 memory ways can be accessed through the memory map.

L2 memory ways allocated to LIM via the WayEnable register are accessible througn the 0x08000000-0x081FFFFF LIM address range. The LIM address range starts with 1920KiBytes of addressable memory coming out of reset. This size decreases in 128KiBytes increment as L2 memory ways are assigned to cacheable L2 memory through increasing the value of the WayEnable register.

L2 memory ways allocated to the scratchpad via manipulation of the WayMask registers are accessible through the 0x0A000000-0x0BFFFFFF Zero Device address range. The offset and layout of the L2 ways within the Zero Device address range is software controlled. The MPFS-HAL start-up code provided as part of the PolarFire SoC bare metal library. allocates scratchpad ways contiguously from the bottom of the Zero Device address range.

L2 memory ways assigned to cache are not accessible through the memory map. These cache ways are managed by the cache coherence engine. Their content is opaque to the software executing on the system.


![L2 Allocation to Memory Map](https://bitbucket.microchip.com/projects/FPGA_PFSOC_ES/repos/polarfire-soc-documentation/raw/images/memory-hierarchy/Cache-LIM-Scratchpad-Memory-Map.png?at=refs%2Fheads%2Ftemporary-images)


