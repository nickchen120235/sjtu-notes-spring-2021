# Kernel Synchronization
## Kernel Control Paths
- each part of kernel are executed interleaved
- kernel control path: a series of command of current process when executing in kernel mode
  - interrupt or exception
  - less cost than process context switch (little context switch)
  
### Kernel Preemption
- preemptible kernel: a process running in kernel mode can be forcibly switched by scheduler
- target: decrease user mode process dispatch latency (delay between the process being ready and the process is actually being executed)
- the kernel can be preempted only if it is executing the bottom half of interrupt handler or workqueue
- whether preemption is allowed or not is not explicitly set

## Synchronization
- race condition: system's behavior is dependent on the sequence or timing of other uncontrollable events
- critical regions: only one process can be inside, others cannot enter until the one inside has left
  - disable interrupts
  - disable kernel preemption
  - multiple core
- simplification
  - no same type of interrupt is allowed before the processing of current interrupt is done
  - interrupts processing cannot be blocked or preempted
  - software interrupts cannot be interleaved on one CPU
- primitives
  - per-cpu variables
    - copy of same variable across CPUs
    - each CPU can only access and modify its own copy on the start of process
    - prevent race condition of multiple CPUs
    - race conditions may still be caused by asynchronous process or kernel preemption
  - atomic operation: the operation is executed in one single instruction, so it won't be interrupted
  - memory barriers: instruction reordering prevention, so all operations before the barrier will be completed before crossing
  - locks
    - spin locks: for multiple CPU, busy waiting, random entrance (no first-come-first-serve guarantee)
    - read/write spin locks: multiple reads, single write
    - seqlocks: write locks are superior to read locks and can be covered, so multiple tries of read may be needed
  - read-copy update
    - allows multiple process read/write at the same time
    - lock-free
    - only dynamic allocated data can be protected
    - sleeping is not allowed
  - semaphores
    - kernel semaphores: for kernel control paths
    - system v ipc semaphores: for user mode processes
  - local interrupt disabling
- how to choose
  - as less locks as possible (as less critical regions as possible)
  - use less spin lock, especially for single-cpu environment
  - atomic_t is preferred
  
## Race Condition Prevention
- atomic_t counter
- global kernel lock (old versions)
- semaphores: memory management, slab cache list, inode
- deadlock: multiple semephores may cause deadlocks
  - by order of semaphore address