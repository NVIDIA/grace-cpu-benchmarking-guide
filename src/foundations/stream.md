# STREAM

STREAM is a defacto standard for measuring memory bandwidth. The benchmark includes four kernels that operate on 1D arrays `a`, `b`, and `c`, with scalar `x`:

 * **COPY**: `c = a`
 * **SCALE**: `b = x*a`
 * **ADD**: `c = a + b`
 * **TRIAD**: `a = b + x*c`

The kernels are executed in sequence in a loop.  Two parameters configure STREAM:
 * `STREAM_ARRAY_SIZE`: The number of double-precision elements in each array.
   It is critical to select a sufficiently large array size when measuring 
   bandwidth to/from main memory.
 * `NTIMES`: The number of iterations of the test loop.

There are many versions of STREAM, but they all use the same four fundamental kernels.  

## Reference Results

 TBD

## Install

The following commands download and compile STREAM with a total memory footprint of approximately 1.8GB, which is sufficient to exceed the L3 cache without excessive runtime.

```bash
wget https://www.cs.virginia.edu/stream/FTP/Code/stream.c
gcc -Ofast -mcpu=native -fopenmp \
  	-DSTREAM_ARRAY_SIZE=80000000 -DNTIMES=50 \
  	-o stream_openmp.exe stream.c
```

## Execute

Stream is parallelized with OpenMP, so execution can be controlled via OpenMP environment variables.  Spreading threads across multiple NUMA domains (`OMP_PROC_BIND=spread`) maximizes the number of active memory controllers and delivers the best performance.

```bash
OMP_NUM_THREADS=72 OMP_PROC_BIND=spread ./stream_openmp.exe
```
