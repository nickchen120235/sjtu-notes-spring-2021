# Process Communication
## Critical Section
> *def* a code segment where shared variables can be accessed
- Locks
- Semaphores
### Locks
- two states
  - unlocked: no process is in the critical section, so any process can enter
  - locked: other process is in the critical section, so the process must wait until unlocked
- procedure
    ```
    suppose X is the lock

      |----------(no)
      V            |
    LOCK(X) --> X == 0? -(yes)-> X = 1 --> |CRITICAL SECTION| --> UNLOCK(X) --> X = 0
    ```
- locking/unlocking protection
  - test-and-set instruction: single **atomic** instruction to write 1 to a memory location
  - interrupt disabling: disable interrupts so the control flow won't change, only works with uni-processor system
  - mixing hardware locks with software locks: protect test-and-set by hardware locks and once it's set, the rest of the critical section is protected by software locks

### Semaphores
- definition
    ```
    Semaphore := {
      Integer value
      Queue queue
    }
    ```
  - value
    - `> 0`: number of processes that can continue (usually no more than 1)
    - `< 0`: there are `|value|` processes being blocked
  - queue: the blocked processes will be here until `signal`ed by others
- actions
  - `wait`: subtract `semaphore.value` by 1, if it's `< 0` then wait until `signal`ed by processes
  - `signal`: notify the waiting queue by adding `semaphore.value` by 1
- use cases (*suppose `S` is the semaphore*)
  - critical section protection
    ```
    WAIT(S) -> |CRITICAL SECTION| -> SIGNAL(S)
    ```
  - sleep and wake up
    ```
    process 1: ... -> WAIT(S) -> get result from process 2 -> do something with it
    process 2: do some calculation -> put the result into buffer -> SIGNAL(S) -> ...
    ```
  - producer-consumer model
    > three semaphores are used here:
    >   - `Sput`: put to buffer
    >   - `Sget`: get from buffer
    >   - `Smutex`: race condition prevention
    - producer
        ```
        while(true) {
          produce next product
          WAIT(Sget) // wait until consumer get all products in the buffer
          WAIT(Smutex) // producer is using the buffer
          put products into the buffer
          SIGNAL(Smutex) // producer no longer uses the buffer
          SIGNAL(Sput) // tell consumer that products are ready
        }
        ```
    - consumer
        ```
        while (true) {
          WAIT(Sput) // wait until producer put finishes putting products into the buffer
          WAIT(Smutex) // consumer is using the buffer
          get products from the buffer
          SIGNAM(Smutex) // consumer no longer uses the buffer
          SIGNAL(Sget) // tell producer that all products are taken
          consume product
        }
        ```

## Inter Process Communication
### Message Queue
a buffer managed by system with specific message structure, for example in UNIX SYS V
```c
typedef struct msg_t {
  long mtype;
  char* buf;
} msg_t;
```
related calls are `msgsnd()`, `msgrcv()`, `msgctl()`

### Shared Memory
a shared memory segment mapped to virtual memory address of different processes, uses procuder-consumer model

related calls are `shmget()`, `shmctl()`, `shmdt()`

### Pipe
file-based, FIFO

related calls are `mkfifo()`, `mknod()`

### Software Interrupt & Signaling
simulates hardware interrupt, can be passed to other processes

related calls are `signal()`, `kill()`

## Deadlock
### Conditions
- mutual exclusion: only one process can access the resource at any time
- hold and wait: a process is holding resources while waiting for additional resources held by others
- no preemption: a resource can be released only voluntarily by the process holding it
- circular wait: each process must be waiting for a resource which is being held by another process, which in turn is waiting for the first process to release the resource

### Prevention
- mutual exclusion: if exclusive access is needed, then deadlock will occur
- hold and wait: requiring processes to request all needed resources before start up, or even requiring processes to request resources only if they have none. a process waiting for popular resources may die because of starvation
- no preemption: preemption of a "locked out" resource generally implies a rollback, and is to be avoided since it is very costly in overhead.if a process holding some resources and requests for some another resource(s) that cannot be immediately allocated to it, the condition may be removed by releasing all the currently being held resources of that process
- circular wait: disabling interrupts during critical sections and using a hierarchy to determine a partial ordering of resources

### Correction
> the target is to break the symmetry
- process termination: abort/rollback one or more competing processes until the deadlock is resolved
- resource preemption: resources allocated to various processes may be successively preempted and allocated to other processes until the deadlock is broken

### Banker's Algorithm
- important items
  - MAX: how much of each resource each process could possibly request
  - ALLOCATED: how much of each resource each process is currently holding
  - AVAILABLE: how much of each resource the system currently has available
- safe/unsafe state
  - safe: all requested resources can be satisified in finite time
  - unsafe: any requested resource cannot be satisified in finite time
- algorithm: resources are allocated only if the system is safe
- example
```
 total     AVAILABLE
A B C D     A B C D
6 5 7 6     3 1 1 2

ALLOCATED       MAX
   A B C D     A B C D
P1 1 2 2 1  P1 3 3 2 2
P2 1 0 3 3  P2 1 2 3 4
P3 1 2 1 0  P3 1 3 5 0

1. P1 needs additional 2 A, 1 B, 1 D
  - AVAILABLE -> 1 0 1 1
2. P1 terminates, returning all resources held by it
  - AVAILABLE -> 4 3 3 3
3. P2 acquires 2 B, 1 D
  - AVAILABLE -> 4 1 3 2
4. P2 terminates
  - AVAILABLE -> 5 3 6 6
5. P3 acquires 1 B, 4 C
  - AVAILABLE -> 5 2 2 6
6. P3 terminates
  - AVAILABLE -> 6 5 7 6

Because all processes were able to terminate, this state is safe.
```