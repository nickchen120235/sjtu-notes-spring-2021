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