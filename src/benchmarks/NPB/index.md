# NAS Parallel Benchmarks

The [NAS Parallel Benchmarks (NPB)](https://www.nas.nasa.gov/software/npb.html) are a small set of programs designed to help evaluate the performance of parallel supercomputers. The NPB 1 benchmarks are derived from computational fluid dynamics (CFD) applications and consist of five kernels and three pseudo-applications. Problem sizes in NPB are predefined and indicated as different classes. Reference implementations of NPB are available in commonly-used programming models like MPI and OpenMP.


## Building the Benchmarks

1. Download and unpack the NPB source code from nas.nasa.gov:

    ```bash
    wget https://www.nas.nasa.gov/assets/npb/NPB3.4.2.tar.gz
    tar xvzf NPB3.4.2.tar.gz
    cd NPB3.4.2/NPB3.4-OMP
    ```

2. Create the `make.def` file to configure the build for NVIDIA HPC compilers:

    ```bash
    cat > config/make.def <<'EOF'
    FC = nvfortran
    FLINK = $(FC)
    F_LIB =
    F_INC =
    FFLAGS = -O3 -mp
    FLINKFLAGS = $(FFLAGS)
    CC = nvc
    CLINK = $(CC)
    C_LIB = -lm
    C_INC =
    CFLAGS = -O3 -mp
    CLINKFLAGS = $(CFLAGS)
    UCC = gcc
    BINDIR = ../bin
    RAND = randi8
    WTIME  = wtime.c
    EOF
    ```

3. Create the `suite.def` file to build all benchmarks with the `D` problem size:

    ```bash
    cat > config/suite.def <<'EOF'
    bt	D
    cg	D
    ep	D
    lu	D
    mg	D
    sp	D
    ua	D
    EOF
    ```

4. Compile all benchmarks:

    ```bash
    make -j suite
    ```

A successful compilation will generate these binaries in the `bin/` directory:

```bash
$ ls bin/
bt.D.x  cg.D.x  ep.D.x  ft.D.x  lu.D.x  mg.D.x  sp.D.x  ua.D.x
```

## Running the Benchmarks

Run each benchmark individually using the command shown below.  In the command, replace `${BENCHMARK}` with the benchmark name, for example `cg.D.x`, and replace `${THREADS}` and `${FLAGS}` with the appropriate values from the reference results shown above.

```bash
OMP_NUM_THREADS=${THREADS} OMP_PROC_BIND=close numactl ${FLAGS} ./bin/${BENCHMARK}
```

## Reference Results

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

### Grace CPU Superchip, 480GB Memory Capacity

Use this script to run all the benchmarks on 72 cores of the Grace CPU:

```bash
#!/bin/bash
for BENCHMARK in bt cg ep lu mg sp ua ; do
    OMP_NUM_THREADS=72 OMP_PROC_BIND=close numactl -m0 ./bin/${BENCHMARK}.D.x
done
```

Performance is reported on the line marked "Mops / total".  The expected performance is shown below.

| Benchmark | Mops / total |
| --------- | ----------- |
| bt.D.x    | 386758.21   |
| cg.D.x    | 26632.65    |
| ep.D.x    | 10485.73    |
| lu.D.x    | 293407.59   |
| mg.D.x    | 125382.93   |
| sp.D.x    | 136893.59   |
| ua.D.x    | 973.52      |
