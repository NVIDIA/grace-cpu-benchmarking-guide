# STREAM

STREAM is a defacto standard for measuring memory bandwidth. The STREAM benchmark is a simple, synthetic benchmark program that measures sustainable main memory bandwidth in MB/s and the corresponding computation rate for simple vector kernels. The benchmark includes four kernels that operate on 1D arrays `a`, `b`, and `c`, with scalar `x`:

- **COPY**: Measures transfer rates in the absence of arithmetic, `c = a`
- **SCALE**: Adds a simple arithmetic operation. `b = x*a`
- **ADD**: Adds a third operand to allow multiple load/store ports on vector machines to be tested, `c = a + b`
- **TRIAD**: Allows chained/overlapped/fused multiply/add operations, `a = b + x*c`

The kernels are executed in sequence in a loop. Two parameters configure STREAM:

- `STREAM_ARRAY_SIZE`: The number of double-precision elements in each array.
  It is critical to select a sufficiently large array size when measuring
  bandwidth to/from main memory.
- `NTIMES`: The number of iterations of the test loop.

There are many versions of STREAM, but they all use the same four fundamental kernels.

Use the STREAM benchmark to check LPDDR5X memory bandwidth. The following
commands download and compile STREAM with a total memory footprint of approximately
2.7GB, which is sufficient to exceed the L3 cache without excessive runtime

## Reference Results

| Superchip    | Capacity (GB) | Threads | numactl flags | TRIAD GB/s |
| ------------ | ------------- | ------- | ------------- | ---------- |
| Grace-Hopper | 120           | 36      | -m0           | 450+       |
| Grace-Hopper | 120           | 72      | -m0           | 450+       |
| Grace-Hopper | 480           | 36      | -m0           | 350+       |
| Grace-Hopper | 480           | 72      | -m0           | 350+       |
| Grace CPU    | 240           | 36      | -m0           | 450+       |
| Grace CPU    | 240           | 72      | -m0,1         | 900+       |
| Grace CPU    | 240           | 144     | -m0,1         | 900+       |

Example of the STREAM Execution Output:

```
jlinford@fc01-gg01:~$ OMP_NUM_THREADS=72 OMP_PROC_BIND=spread numactl -m0,1
./stream_openmp.exe
-------------------------------------------------------------
STREAM version $Revision: 5.10 $
-------------------------------------------------------------
This system uses 8 bytes per array element.
-------------------------------------------------------------
Array size = 120000000 (elements), Offset = 0 (elements)
Memory per array = 915.5 MiB (= 0.9 GiB).
Total memory required = 2746.6 MiB (= 2.7 GiB).
Each kernel will be executed 200 times.
 The *best* time for each kernel (excluding the first iteration)
 will be used to compute the reported bandwidth.
-------------------------------------------------------------
Number of Threads requested = 72
Number of Threads counted = 72
-------------------------------------------------------------
Your clock granularity/precision appears to be 1 microseconds.
Each test below will take on the order of 2927 microseconds.
 (= 2927 clock ticks)
Increase the size of the arrays if this shows that
you are not getting at least 20 clock ticks per test.
-------------------------------------------------------------
WARNING -- The above is only a rough guideline.
For best results, please be sure you know the
precision of your system timer.
-------------------------------------------------------------
Function Best Rate MB/s Avg time Min time Max time
Copy: 919194.6 0.002149 0.002089 0.002228
Scale: 913460.0 0.002137 0.002102 0.002192
Add: 916926.9 0.003183 0.003141 0.003343
Triad: 903687.9 0.003223 0.003187 0.003308
-------------------------------------------------------------
Solution Validates: avg error less than 1.000000e-13 on all three arrays
--------------------------------------------------------
```

## Install

The following commands download and compile STREAM with a total memory footprint of approximately 2.7GB, which is sufficient to exceed the L3 cache without excessive runtime. The general rule for running STREAM is that each array must be at least 4x the size of the sum of all the last-level caches used in the run, or 1 Million elements, whichever is larger.

```bash
wget https://www.cs.virginia.edu/stream/FTP/Code/stream.c &&
gcc -Ofast -march=native -fopenmp \
  	-DSTREAM_ARRAY_SIZE=120000000 -DNTIMES=200 \
  	-o stream_openmp.exe stream.c
```

## Execute for a single socket

To run STREAM, set the number of OpenMP threads (OMP_NUM_THREADS) and the numactl flags according to the example below. Replace `{THREADS}` and `{FLAGS}` with the appropriate values from the table of reference results shown above. Use `OMP_PROC_BIND=spread` to distribute the threads evenly over all available cores and maximize bandwidth.

```bash
OMP_NUM_THREADS={THREADS} OMP_PROC_BIND=spread numactl {FLAGS} ./stream_openmp.exe
```

System bandwidth is proportional to the memory capacity. Find your system’s memory capacity in the table above and use the given parameters to generate the expected score
for STREAM TRIAD. For example, when running on a Grace-Hopper superchip with a memory capacity of 120GB, this command will score at least 450GB/s in STREAM TRIAD:

```bash
OMP_NUM_THREADS=72 OMP_PROC_BIND=spread numactl -m0 ./stream_openmp.exe
```

Similarly, this command will score at least 900GB/s in STREAM TRIAD on a Grace CPU Superchip with a memory capacity of 240GB:

```bash
OMP_NUM_THREADS=144 OMP_PROC_BIND=spread numactl -m0,1 ./stream_openmp.exe
```
