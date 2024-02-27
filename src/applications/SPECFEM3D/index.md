# SPECFEM3D

SPECFEM3D Cartesian simulates acoustic (fluid), elastic (solid), coupled acoustic/elastic, poroelastic or seismic wave propagation in any type of conforming mesh of hexahedra (structured or not).

## Build

The following script will build SPEMFEM3D. The script tested on a freshly booted Ubuntu 22.04. 

```bash
#!/bin/bash

set -e

sudo apt update && sudo apt install -y git build-essential gcc gfortran libopenmpi-dev openmpi-bin
git clone https://github.com/SPECFEM/specfem3d.git
pushd ./specfem3d
./configure FC=gfortran CC=gcc
make all
cp -r EXAMPLES/applications/meshfem3D_examples/simple_model/DATA/* DATA/
sed -i "s/NPROC .*/NPROC    = $(nproc)/g" DATA/Par_file
sed -i "s/NSTEP .*/NSTEP    = 10000/g" DATA/Par_file
sed -i "s/DT .*/DT       = 0.01/g" DATA/Par_file
sed -i "s/NEX_XI .*/NEX_XI       = 448/g" DATA/meshfem3D_files/Mesh_Par_file
sed -i "s/NEX_ETA .*/NEX_ETA      = 576/g" DATA/meshfem3D_files/Mesh_Par_file
sed -i "s/NPROC_XI .*/NPROC_XI      = 8/g" DATA/meshfem3D_files/Mesh_Par_file
sed -i "s/NPROC_ETA .*/NPROC_ETA     = 18/g" DATA/meshfem3D_files/Mesh_Par_file
sed -i '/^#NEX_XI_BEGIN/{n;s/1.*/1 448 1 576 1 4 1/;n;s/1.*/1 448 1 576 5 5 2/;n;s/1.*/1 448 1 576 6 15 3/}' DATA/meshfem3D_files/Mesh_Par_file
popd
```

## Running Benchmarks on Grace

```bash
pushd ./specfem3d
mkdir -p DATABASES_MPI
rm -rf DATABASES_MPI/*
rm -rf OUTPUT_FILES/*
mpirun --allow-run-as-root -n $(nproc) --bind-to none --map-by core ./bin/xmeshfem3D
mpirun --allow-run-as-root -n $(nproc) --bind-to none --map-by core ./bin/xgenerate_databases
mpirun --allow-run-as-root -n $(nproc) --bind-to none --map-by core ./bin/xspecfem3D
cat OUTPUT_FILES/output_solver.txt
popd
```
## Reference Results

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

The following result was collected on a Grace Superchip using 144 CPU cores.

```bash
 Time loop finished. Timing info:
 Total elapsed time in seconds =    991.00492298699999
 Total elapsed time in hh:mm:ss =      0 h 16 m 31 s
```
