# STREAM

STREAM is a defacto standard for measuring memory bandwidth. The STREAM benchmark is a simple, synthetic benchmark program that measures sustainable main memory bandwidth in MB/s and the corresponding computation rate for simple vector kernels. The benchmark includes four kernels that operate on 1D arrays `a`, `b`, and `c`, with scalar `x`:

 * **COPY**: Measures transfer rates in the absence of arithmetic, `c = a`
 * **SCALE**: Adds a simple arithmetic operation. `b = x*a`
 * **ADD**: Adds a third operand to allow multiple load/store ports on vector machines to be tested, `c = a + b`
 * **TRIAD**: Allows chained/overlapped/fused multiply/add operations, `a = b + x*c`

The kernels are executed in sequence in a loop. Two parameters configure STREAM:
 * `STREAM_ARRAY_SIZE`: The number of double-precision elements in each array.
   It is critical to select a sufficiently large array size when measuring
   bandwidth to/from main memory.
 * `NTIMES`: The number of iterations of the test loop.

There are many versions of STREAM, but they all use the same four fundamental kernels.

## Reference Results

Sample on 3.2 GHz, C2 (A02) system

```bash
-------------------------------------------------------------
STREAM version $Revision: 5.10 $
-------------------------------------------------------------
This system uses 8 bytes per array element.
-------------------------------------------------------------
Array size = 80000000 (elements), Offset = 0 (elements)
Memory per array = 610.4 MiB (= 0.6 GiB).
Total memory required = 1831.1 MiB (= 1.8 GiB).
Each kernel will be executed 50 times.
 The *best* time for each kernel (excluding the first iteration)
 will be used to compute the reported bandwidth.
-------------------------------------------------------------
Number of Threads requested = 72
Number of Threads counted = 72
-------------------------------------------------------------
Your clock granularity/precision appears to be 1 microseconds.
Each test below will take on the order of 3526 microseconds.
   (= 3526 clock ticks)
Increase the size of the arrays if this shows that
you are not getting at least 20 clock ticks per test.
-------------------------------------------------------------
WARNING -- The above is only a rough guideline.
For best results, please be sure you know the
precision of your system timer.
-------------------------------------------------------------
Function    Best Rate MB/s  Avg time     Min time     Max time
Copy:          438512.5     0.003040     0.002919     0.003214
Scale:         442014.6     0.003046     0.002896     0.003313
Add:           421141.3     0.004804     0.004559     0.004951
Triad:         426584.6     0.004803     0.004501     0.004953
-------------------------------------------------------------
Solution Validates: avg error less than 1.000000e-13 on all three arrays
-------------------------------------------------------------
```

## Install

The following commands download and compile STREAM with a total memory footprint of approximately 1.8GB, which is sufficient to exceed the L3 cache without excessive runtime. The general rule for running STREAM is that each array must be at least 4x the size of the sum of all the last-level caches used in the run, or 1 Million elements, whichever is larger.

```bash
wget https://www.cs.virginia.edu/stream/FTP/Code/stream.c &&
gcc -Ofast -march=native -fopenmp \
  	-DSTREAM_ARRAY_SIZE=80000000 -DNTIMES=50 \
  	-o stream_openmp.exe stream.c
```

## Execute for a single socket

Update the governors to performance.

```bash
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor > /dev/null
```

Stream is parallelized with OpenMP, so execution can be controlled via OpenMP environment variables and numactl, in the case of a dual socket system. Spreading threads across multiple NUMA domains (`OMP_PROC_BIND=spread`) maximizes the number of active memory controllers and delivers the best performance.

```bash
OMP_NUM_THREADS=72 OMP_PROC_BIND=spread numactl -N 0 --localalloc ./stream_openmp.exe
```
