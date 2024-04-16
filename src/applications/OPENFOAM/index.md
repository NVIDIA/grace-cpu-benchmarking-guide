# OpenFOAM

OpenFOAM is a C++ toolbox for the development of customized numerical solvers, and pre-/post-processing utilities for the solution of continuum mechanics problems, most prominently including computational fluid dynamics.

## Build

The following script will build OpenFOAM and all the dependencies. The script tested on a freshly booted Ubuntu 22.04. 

```bash
#!/bin/bash

set -e

sudo apt update && sudo apt install -y time libfftw3-dev curl wget \ 
    build-essential libscotch-dev libcgal-dev git flex libfl-dev bison cmake \ 
    zlib1g-dev libboost-system-dev libboost-thread-dev \ 
    libopenmpi-dev openmpi-bin gnuplot \ 
    libreadline-dev libncurses-dev libxt-dev numactl

wget https://dl.openfoam.com/source/v2312/OpenFOAM-v2312.tgz 
wget https://dl.openfoam.com/source/v2312/ThirdParty-v2312.tgz
tar -zxvf OpenFOAM-v2312.tgz && tar -zxvf ThirdParty-v2312.tgz

source ./OpenFOAM-v2312/etc/bashrc
pushd $WM_PROJECT_DIR 
./Allwmake -j -s -l -q
popd
```

## Running Benchmarks on Grace

```bash
export OPENFOAM_ROOT=${PWD}
export OPENFOAM_MPIRUN_ARGS="--map-by core --bind-to none --report-bindings"

source ./OpenFOAM-v2312/etc/bashrc
git clone https://develop.openfoam.com/committees/hpc.git 
pushd hpc/incompressible/simpleFoam/HPC_motorbike/Large/v1912 
sed -i "s/numberOfSubdomains.*/numberOfSubdomains $(nproc);/g" system/decomposeParDict 
sed -i "s/vector/normal/g" system/mirrorMeshDict 
sed -i "s/^endTime.*/endTime         100;/" system/controlDict
sed -i "s/^writeInterval.*/writeInterval   1000;/" system/controlDict
curl -o system/fvSolution "https://develop.openfoam.com/Development/openfoam/-/raw/master/tutorials/incompressible/simpleFoam/motorBike/system/fvSolution?ref_type=heads"
chmod +x All*

./AllmeshL
./Allrun
cat log.*
popd
```
## Reference Results

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

The following result was collected on a Grace CPU Superchip using 144 CPU cores.

```
ExecutionTime = 189.49 s  ClockTime = 197 s
```
