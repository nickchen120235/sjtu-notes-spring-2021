# Memory Management
## Addressing
- Fixed: addresses are used as-is
- Static relocation: addresses are translated by linker before execution
- Dynamic relocation: addresses are translated during run-time

## Memory Allocation
### Algorithms
- First fit: the process is allocated at the first block whose size is sufficiently large. the scan starts from the beginning of the memory
- Next fit: same as first fit but the scan starts from where the last allocation is
  - *Faster than first fit*
- Best fit: the process is allocated at the smallest block whose size is sufficiently large
  - *Minimizes the wastage space but slow and sometimes stupid*
- Worst fit: the process is allocated at the largest block available
  - *Very fast*

### Extension
- Overlay: separate one big process into pieces by its logical structure, code that won't execute together can be in the same overlay segment
- Swapping: When one process is stoppping (out-of-time or waiting), swap its context with the next process to be executed. (context switching)
- Virtual memory: some part of the code will only be executed once (e.g. initialization) or rarely used (e.g. exception handling), put those part outside main memory (e.g. pagefile in Windows or /swap in Linux) whenever needed

### Paging
- Divide virtual memory address space into pages, a page frame is the smallest fixed-length contiguous block of physical memory into which memory pages are mapped by the operating system
- Page tables are used to translate virtual addresses seen by application to physical addresses seen by hardware
- For any entry in the page table
  - If a page is in the real memory, the address listed will be the corresponding physical address
  - If not, a **page fault exception** will be raised, and the operating system must handle it by
    1. Determine the location of the data on disk
    2. Obtain an empty page frame as the container
    3. Load the data into the previous obtained container
    4. Update page table
    5. Return control to the application
- Replacement
  - Theoretically optimal algorithm: discard those which won't appear in the future
  - FIFO: discard the oldest arrival
  - Least recently used: put the accessed page into the back, always discard the one in the front
  - Not recently used: two additional bits in each entry (referenced, modified) and are set accordingly. Discard randomly from those that are neither referenced nor modified. The two bits are reset at a time interval

### Segmentation
- Divide physical address space into segments
- Applications can be divided into segments (e.g. code, data, etc.)
- Segmentation + Paging