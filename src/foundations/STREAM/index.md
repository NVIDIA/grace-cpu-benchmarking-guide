# STREAM

STREAM is the standard for measuring memory bandwidth. The STREAM benchmark is a simple, synthetic benchmark program that measures the sustainable main memory bandwidth in MB/s and the corresponding computation rate for simple vector kernels. The benchmark includes the following kernels that operate on 1D arrays `a`, `b`, and `c`, with scalar `x`:

- **COPY**: Measures transfer rates in the absence of arithmetic: `c = a`
- **SCALE**: Adds a simple arithmetic operation: `b = x*a`
- **ADD**: Adds a third operand to test multiple load/store ports: `c = a + b`
- **TRIAD**: Allows chained/overlapped/fused multiply/add operations: `a = b + x*c`

The kernels are executed in sequence in a loop, and the following parameters configure STREAM:

- `STREAM_ARRAY_SIZE`: The number of double-precision elements in each array. When you measure the bandwidth to/from main memory, you must select a sufficiently large array size.
- `NTIMES`: The number of iterations of the test loop.

Use the STREAM benchmark to check LPDDR5X memory bandwidth. The following
commands download and compile STREAM with a total memory footprint of approximately
2.7GB, which is sufficient to exceed the L3 cache without excessive runtime


## Install

The following commands download and compile STREAM with memory footprint of approximately 2.7GB per Grace CPU, which is sufficient to exceed the total system L3 cache without excessive runtime. The general rule for running STREAM is that each array must be at least four times the size of the sum of all the last-level caches that were used in the run, or 1 million elements, whichever is larger.

```bash
STREAM_ARRAY_SIZE="($(nproc)/72*120000000)"
wget https://www.cs.virginia.edu/stream/FTP/Code/stream.c
gcc -Ofast -march=native -fopenmp -mcmodel=large -fno-PIC \
  	-DSTREAM_ARRAY_SIZE=${STREAM_ARRAY_SIZE} -DNTIMES=200 \
  	-o stream_openmp.exe stream.c
```

## Execute

To run STREAM, set the number of OpenMP threads (OMP_NUM_THREADS) according to the following example. Replace `${THREADS}` with the appropriate value from the table of reference results shown above. To distribute the threads evenly over all available cores and maximize bandwidth, use `OMP_PROC_BIND=spread`.

```bash
OMP_NUM_THREADS=${THREADS} OMP_PROC_BIND=spread ./stream_openmp.exe
```

Grace superchip memory bandwidth is proportionate to the total memory capacity. Find your system's memory capacity in the table above and use the same number of threads to generate the expected score
for STREAM TRIAD. For example, when running on a Grace-Hopper superchip with a memory capacity of 120GB, this command will report between 410GB/s and 486GB/s in STREAM TRIAD:

```bash
OMP_NUM_THREADS=72 OMP_PROC_BIND=spread ./stream_openmp.exe
```

Similarly, the following command will report between 820GB/s and 972GB/s in STREAM TRIAD on a Grace CPU Superchip with a memory capacity of 480GB:

```bash
OMP_NUM_THREADS=144 OMP_PROC_BIND=spread ./stream_openmp.exe
```

## Reference Results

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

Memory bandwidth depends on many factors, for instance, operating system kernel version and the default memory page size.  
Without any code changes, STREAM TRIAD should score between 80% and 95% of the system's theoretical peak memory bandwidth.

| Superchip    | Memory Capacity (GB) | Threads | TRIAD Min MB/s |
| ------------ | -------------------- | ------- | -------------- |
| Grace Hopper | 120                  | 72      | 400,000        |
| Grace Hopper | 480                  | 72      | 307,000        |
| Grace CPU    | 240                  | 144     | 800,000        |
| Grace CPU    | 480                  | 144     | 800,000        |

Here is an example of the STREAM execution output:

```
OMP_NUM_THREADS=144 OMP_PROC_BIND=spread ./stream_openmp.exe
-------------------------------------------------------------
STREAM version $Revision: 5.10 $
-------------------------------------------------------------
This system uses 8 bytes per array element.
-------------------------------------------------------------
Array size = 240000000 (elements), Offset = 0 (elements)
Memory per array = 1831.1 MiB (= 1.8 GiB).
Total memory required = 5493.2 MiB (= 5.4 GiB).
Each kernel will be executed 200 times.
 The *best* time for each kernel (excluding the first iteration)
 will be used to compute the reported bandwidth.
-------------------------------------------------------------
Number of Threads requested = 144
Number of Threads counted = 144
-------------------------------------------------------------
Your clock granularity/precision appears to be 1 microseconds.
Each test below will take on the order of 5729 microseconds.
   (= 5729 clock ticks)
Increase the size of the arrays if this shows that
you are not getting at least 20 clock ticks per test.
-------------------------------------------------------------
WARNING -- The above is only a rough guideline.
For best results, please be sure you know the
precision of your system timer.
-------------------------------------------------------------
Function    Best Rate MB/s  Avg time     Min time     Max time
Copy:          662394.7     0.005964     0.005797     0.008116
Scale:         685483.8     0.005744     0.005602     0.007843
Add:           787098.2     0.007689     0.007318     0.008325
Triad:         806812.4     0.007713     0.007139     0.011388
-------------------------------------------------------------
Solution Validates: avg error less than 1.000000e-13 on all three arrays
-------------------------------------------------------------
```
