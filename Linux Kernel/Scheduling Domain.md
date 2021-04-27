# Scheduling Domain
## Why
- one CPU
  - `schedule()` is used to pick a process from runqueue
  - the need of CPU usage per process, priority
- multiple CPU
  - each CPU picks a process from its own runqueue
  - each process can be in only one runqueue
  - kernel checks whether runqueues are balanced
  - balancing across CPUs, scheduling domain
  
## Structure
- a group of CPUs
- different cost between different methods of balancing
  - different threads in one core vs different threads across different cores
  - cores in the same NUMA node vs cores across different NUMA nodes
- hierarchical
  - each level contains processors with same attributes (e.g. same core, same NUMA node)
  - using method with less cost first
*example picture*

## Load Balancing
- each scheduling domain are divided into one or more scheduling group
  - each scheduling group contains scheduling domains in the next level of hierarchy
  - load balancing are accomplished between scheduling groups
- process
  1. register `SCHED_SOFTIRQ`, load balancing process starts if the interrupt is triggered by one CPU
  2. the CPU looks for the busiest runqueue in the busiest scheduling group from the lowest level
  3. if the average load of the CPU is lower than the busiest one, move some processes from the busiest to the CPU