# GRAPH500 

The Graph500 is a rating of supercomputer systems, focused on data-intensive loads.

## Build

The following script will build Graph500 and all the dependencies. The script tested on a freshly booted Ubuntu 22.04. 

```bash
#!/bin/bash

set -e

sudo apt update && sudo apt install -y wget build-essential python3 numactl git
wget https://download.open-mpi.org/release/open-mpi/v5.0/openmpi-5.0.1.tar.gz
gunzip -c openmpi-5.0.1.tar.gz | tar xf -
mkdir -p ./ompi
export PATH="${PWD}/ompi/bin:${PATH}"
export LD_LIBRARY_PATH="${PWD}/ompi/lib:${LD_LIBRARY_PATH}"
pushd openmpi-5.0.1
./configure --prefix=${PWD}/../ompi
make all install
popd

git clone https://github.com/graph500/graph500.git
pushd ./graph500/src/
sed -i '/^CFLAGS/s/$/ -DPROCS_PER_NODE_NOT_POWER_OF_TWO -fcommon/' Makefile
make
popd
```

## Running Benchmarks on Grace

```bash
#!/bin/bash
export SKIP_VALIDATION=1
unset SKIP_BFS

mpirun --allow-run-as-root -n $(nproc) --map-by core ./graph500/src/graph500_reference_bfs 28 16 
```

## Reference Results

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

In the below output, bfs harmonic_mean_TEPS and bfs mean_time are our performance and runtime metrics respectively. TEPS here is traversed edges per second which is our absolute perf metric.

```bash
SCALE:                          28
edgefactor:                     16
NBFS:                           64
graph_generation:               14.2959
num_mpi_processes:              144
construction_time:              8.21669
bfs  min_time:                  1.4579
bfs  firstquartile_time:        1.47956
bfs  median_time:               1.54229
bfs  thirdquartile_time:        1.70613
bfs  max_time:                  1.81811
bfs  mean_time:                 1.58271
bfs  stddev_time:               0.112696
min_nedge:                      4294921166
firstquartile_nedge:            4294921166
median_nedge:                   4294921166
thirdquartile_nedge:            4294921166
max_nedge:                      4294921166
mean_nedge:                     4294921166
stddev_nedge:                   0
bfs  min_TEPS:                  2.3623e+09
bfs  firstquartile_TEPS:        2.51735e+09
bfs  median_TEPS:               2.78478e+09
bfs  thirdquartile_TEPS:        2.90284e+09
bfs  max_TEPS:                  2.94596e+09
bfs  harmonic_mean_TEPS:     !  2.71366e+09
bfs  harmonic_stddev_TEPS:      2.43441e+07
bfs  min_validate:              -1
bfs  firstquartile_validate:    -1
bfs  median_validate:           -1
bfs  thirdquartile_validate:    -1
bfs  max_validate:              -1
bfs  mean_validate:             -1
bfs  stddev_validate:           0
```
