## Debugging 

Here are some useful techniques and tools for finding and resolving
bugs while migrating your applications to NVIDIA Grace.

### Sanitizers

The compiler may generate code and layout data slightly differently on Arm64
compared to an x86 system and this could expose latent memory bugs that were previously
hidden. On GCC, the easiest way to look for these bugs is to compile with the
memory sanitizers by adding the below to standard compiler flags:
```makefile
    CFLAGS += -fsanitize=address -fsanitize=undefined
    LDFLAGS += -fsanitize=address  -fsanitize=undefined
```
Run the resulting binary, and any bugs detected by the sanitizers will cause
the program to exit immediately and print helpful stack traces and other
information.

### Memory Ordering 

Arm is weakly ordered, similar to POWER and other modern architectures, while
x86 is a variant of total-store-ordering (TSO).
Code that relies on TSO may lack barriers to properly order memory references.
Arm64 systems are [weakly ordered multi-copy-atomic](https://www.cl.cam.ac.uk/~pes20/armv8-mca/armv8-mca-draft.pdf).

While TSO allows reads to occur out-of-order with writes and a processor to
observe its own write before it is visible to others, the Armv8 memory model has
further relaxations for performance and power efficiency.
**Code relying on pthread mutexes or locking abstractions
found in C++, Java or other languages shouldn't notice any difference.** Code that
has a bespoke implementation of lockless data structures or implements its own
synchronization primitives will have to use the proper intrinsics and
barriers to correctly order memory transactions.  
See [Locks, Synchronization, and Atomics](atomics.md) for more information.
