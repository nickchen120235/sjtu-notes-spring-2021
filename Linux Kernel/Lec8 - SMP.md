# Symmetric Multiprocessing (SMP)
## Introduction
### Definition
- multiple processors of same kind connected to a shared ram
- all processors have full control of all i/o devices
- controls by the same operating system

### Non-uniform Memory Access (NUMA)
- different processors connected to different memory node
- accessing local memory are faster than accessing foreign memory
- for localized load

### Characteristics
- different parts of Linux kernel can be run simultaneously on different processors
- scheduling across processes is possible
- challenges
  - scheduling: no repetitive execution and starvation
  - synchronization: simultaneous access of shared data
  - re-entrant
  - memory management
  - reliability
  
## Scheduling
- time-sharing: use only one structure for scheduling
- space-sharing: multiple threads are scheduled on partitioned processors
- [scheduling domain](./Scheduling Domain.md)

## Synchronization
- test and set lock: access simultaneously, and multiple processors claim that they have the lock, while only one lock is allowed
  - lock the bus so one processor at a time
- cache thrashing: due to cache coherency, processors executing test and set lock may invalid locks of others, resulting in cache thrashing
  - use multiple locks for each processor, and connect locks by order of access, every processor spins on its own lock