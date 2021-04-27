# Interrupt Handling
## Restriction
- deal with interrupt asap
- interrupts can be interrupted by other interrupts
- uninterruptible critical zones (e.g. mutex locks)

## Interrupt Function
- unstoppable
- data exchange with user-space is prohibited
- re-entrant is not nedded
- sleeping (e.g. `msleep()`) is prohibited
- register in kernel: `request_irq()`

## Interrupt Handling
### Interrupt Descriptor Table
- mapping between interrupt (irq) and interrupt function
- structure
  - task gate
  - interrupt gate
  - trap gate
### Context switch
- save current state
- execute interrupt function
- recover state before interrupt

## Linux Interrupt Handling
### Interrupt Vector
- hardware interrupt numbers: assigned during design of SoC
- software interrupt numbers: mapping of Linux kernel
  - compatibility across different architecture
  - multiple same-type devices support
### Mapping between hardware and software interrupts
- BITMAP `allocated_irqs`
- steps
  1. find an unallocated number in the bitmap
  2. create a `struct irq_desc` instance
### Interrupt Handler Registration
- `request_irq()`
- `request_threaded_irq()`: scheduled with other process to decrease the delay of processes with high priority
- irq flags: trigger type, for specified cpu, oneshot, etc.
- `irqaction`: encapsulation of interrupt handler
  - for shared interrupts, multiple `irqaction` are chained into a linked list
  
## Two Halves of Interrupt Handling
### Top Half
- characteristics
  - critical operations
  - cannot be interrupted again
  - immediately handled by kernel
- why
  - hardware interrupts are triggered asynchronously, which breaks the execution of important code
  - avoiding blocking normal processes for too long
  - hardware interrupts are usually not interruptible

### Bottom Half
- characteristics
  - most operations of interrupt handling
  - can be interrupted
  - will be done later
- when
  - after top half finishes, before returning to normal operation
  - as a kernel thread, scheduled with other processes

### Where to put
- time-critial operation should be in the top half
- hardware-related operation should be in the top half
- operations which can't be interrupted by others should be in the top half
- the rest should be in the bottom half

## SoftIRQ
### Software Interrupts
- static define
- bottom half of interrupt handling
- `_softirq_pending`: pending software interrupts for each CPU
- re-entrant is needed for handler function
- `open_softirq()`, `raise_softirq()`, `do_softirq()`
- interruptible
  - cannot be interrupted by the top half of itself
- when: before the top half returns
- one kind of interrupt context, therefore high priority

### Tasklet
- based on software interrupt
- one type of tasklet only runs on one CPU, parallel execution is not allowed
- tasklets can be moved between CPU
- parallel execution of different types of tasklets on different CPU is allowed
- `tasklet_vec`, `tasklet_hi_vec`

### Workqueue
- process context, as a kernel thread
- can be scheduled or even put into sleep
- Concurrency-Managed WorkQueues (CMWQ): thread pools
  - why: too many kernel threads, low concurrency, deadlock
  - bound: specified CPU
  - unbounded: any CPU
  
### How to Choose
- if some sleep is needed, use workqueue
- needed to be executed in a short time, use software interrupt or tasklet
- the rest should use workqueue