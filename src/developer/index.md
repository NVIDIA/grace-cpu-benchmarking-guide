# Developing for NVIDIA Grace

## Architectural Features

NVIDIA Grace implements the SVE2 and the NEON single-instruction-multiple-data (SIMD) instruction sets (refer to [Arm SIMD Instructions](vectorization.md) for more information).

All server-class Arm64 processors support low-cost atomic operations that can improve system throughput for thread communication, locks, and mutexes
(refer to [Locks, Synchronization, and Atomics](atomics.md) for more information).

All Arm CPUs (including NVIDIA Grace) provide several ways to determine the available CPU resources and topology at runtime
(refer to [Runtime CPU Detection](cpudetect.md) for more information and the example code).

## Debugging and Profiling

Typically, all the same debuggers and profilers you rely on when working on x86 are available on NVIDIA Grace.
The notable exceptions are vendor-specific products, for example, Intel® VTune.  The capabilities provided by these tools are
provided also by other tools on NVIDIA Grace (refer to [Debugging](debugging.md) for more information).

## Language-Specific Guidance

Check the [Languages](languages/index.md) page for any language-specific guidance related to LSE, locking, synchronization, and atomics. 
If no guide is provided then there are no Arm-related specific issues for that language.  You can proceed as you would on any other platform.
