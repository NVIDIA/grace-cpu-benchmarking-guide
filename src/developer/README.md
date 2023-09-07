# Developing for NVIDIA Grace

## Architectural Features

NVIDIA Grace implements both the SVE2 and the NEON single-instruction-multiple-data (SIMD) instruction sets.
See [Arm SIMD Instructons](vectorization.md) for details.

All server-class Arm64 processors support low-cost atomic operations which can improve system throughput for thread communication, locks, and mutexes. 
See [Locks, Synchronization, and Atomics](atomics.md) for details.

All Arm CPUs (including NVIDIA Grace) provide several ways to determine the available CPU resources and topology at runtime.
See [Runtime CPU Detection](cpudetect.md) for more details and example code.

## Debugging and Profiling

Generally speaking, all the same debuggers and profilers you rely on when working on x86 are available on NVIDIA Grace.
The notable exceptions are vendor-specific products, e.g. Intel VTune.  The capabilities provided by these tools are
provided by other tools on NVIDIA Grace.  See [Debugging](debugging.md) and [Profiling](profiling.md) for details.

## Language-specific Guidance

Check the [Languages](languages/index.html) page for any language-specific guidance related to LSE, locking, synchronization, and atomics. 
If no guide is provided then there are no Arm-related specific issues for that language.  Just proceed as you would on any other platform.
