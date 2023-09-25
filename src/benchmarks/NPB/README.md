# NAS Parallel Benchmarks (NPB 1)

The [NAS Parallel Benchmarks (NPB)](https://www.nas.nasa.gov/software/npb.html) are a small set of programs designed to help evaluate the performance of parallel supercomputers. The "NPB 1" benchmarks are derived from computational fluid dynamics (CFD) applications and consist of five kernels and three pseudo-applications. Problem sizes in NPB are predefined and indicated as different classes. Reference implementations of NPB are available in commonly-used programming models like MPI and OpenMP.

## Reference Results

| Superchip    | Capacity (GB) | Threads | numactl flags | bt | cg | ep | ft | is | lu | mg | sp | ua |
| ------------ | ------------- | ------- | ------------- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
| Grace-Hopper | 120           | 72      | -m0           | 00 | 00 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |
| Grace-Hopper | 480           | 72      | -m0           | 00 | 00 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |
| Grace CPU    | 240           | 72      | -m0           | 00 | 00 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |
| Grace CPU    | 240           | 144     | -m0,1         | 00 | 00 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |

## Build

Download and unpack the NPB source code from nas.nasa.gov:
```
wget https://www.nas.nasa.gov/assets/npb/NPB3.4.2.tar.gz
tar xvzf NPB3.4.2.tar.gz
cd NPB3.4.2/NPB3.4-OMP
```

Create the `make.def` file to configure the build for NVIDIA HPC compilers:
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

Create the `suite.def` file to build all benchmarks with the `D` problem size:
```bash
cat > config/suite.def <<'EOF'
bt	D
cg	D
ep	D
ft	D
is	D
lu	D
mg	D
sp	D
ua	D
EOF
```

Compile all benchmarks:
```bash
make -j suite
```

A successful compilation will generate these binaries in the `bin/` directory:
```bash
$ ls bin/
bt.D.x  cg.D.x  ep.D.x  ft.D.x  lu.D.x  mg.D.x  sp.D.x  ua.D.x
```

## Run

Run each benchmark as shown below.  Replace `{BENCHMARK}` with the benchmark name, e.g. `cg`, and replace `{THREADS}` and `{FLAGS}` with the appropriate values from the table of reference results shown above.
```bash
OMP_NUM_THREADS={THREADS} OMP_PROC_BIND=close numactl {FLAGS} ./bin/{BENCHMARK}.D.x
```
