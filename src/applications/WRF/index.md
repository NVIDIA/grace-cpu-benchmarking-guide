# Weather Research and Forecasting Model 

The [Weather Research and Forecasting (WRF) Model](https://www.mmm.ucar.edu/weather-research-and-forecasting-model) is a next-generation mesoscale numerical weather prediction system designed for both atmospheric research and operational forecasting needs. It features two dynamical cores, a data assimilation system, and a software architecture facilitating parallel computation and system extensibility.

Arm64 is supported by the standard WRF distribution as of WRF 4.3.3. The following is an example of how to perform the standard procedure to build and execute on NVIDIA Grace. See [http://www2.mmm.ucar.edu/wrf/users/download/get_source.html](http://www2.mmm.ucar.edu/wrf/users/download/get_source.html) for more details.

## Install WRF

### Initial Configuration

Verify that the most recent NVIDIA HPC SDK is available in your environment. The simplest way to do this is to load the `nvhpc` module file.

```bash
module load nvhpc
```

```bash
nvc --version
```

```
nvc 23.7-0 linuxarm64 target on aarch64 Linux -tp neoverse-v2
NVIDIA Compilers and Tools
Copyright (c) 2022, NVIDIA CORPORATION & AFFILIATES.  All rights reserved.
```

NVIDIA HPC SDK includes optimized MPI compilers and libraries, so you'll also have the appropriate MPI compilers in your path:

```bash
$ which mpirun
/opt/nvidia/hpc_sdk/Linux_aarch64/23.7/comm_libs/mpi/bin/mpirun

$ mpicc -show
nvc -I/opt/nvidia/hpc_sdk/Linux_aarch64/23.7/comm_libs/openmpi/openmpi-3.1.5/include -Wl,-rpath -Wl,$ORIGIN:$ORIGIN/../../lib:$ORIGIN/../../../lib:$ORIGIN/../../../compilers/lib:$ORIGIN/../../../../compilers/lib:$ORIGIN/../../../../../compilers/lib -Wl,-rpath -Wl,/opt/nvidia/hpc_sdk/Linux_aarch64/23.7/comm_libs/openmpi/openmpi-3.1.5/lib -L/opt/nvidia/hpc_sdk/Linux_aarch64/23.7/comm_libs/openmpi/openmpi-3.1.5/lib -lmpi
```

NetCDF requires libcurl. On Ubuntu, you can install this easily with this command:

```bash
sudo apt install libcurl4-openssl-dev
```

Create a build directory to hold WRF and all its dependencies

```bash
mkdir WRF

# Configure build environment
export BUILD_DIR="$HOME/WRF"
export HDFDIR=$BUILD_DIR/opt
export HDF5=$BUILD_DIR/opt
export NETCDF=$BUILD_DIR/opt

export PATH=$HDFDIR/bin:$PATH
export LD_LIBRARY_PATH=$HDFDIR/lib:$LD_LIBRARY_PATH
```

### Dependencies

WRF depends on the NetCDF Fortran library, which in turn requires the NetCDF C library and HDF5. This guide assumes that all of WRF's dependencies have been installed at the same location such that they share the same `lib` and `include` directories.

#### HDF5

```bash
cd $BUILD_DIR

wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.14/hdf5-1.14.2/src/hdf5-1.14.2.tar.gz
tar xvzf hdf5-1.14.2.tar.gz
cd hdf5-1.14.2

CC=mpicc FC=mpifort \
    CFLAGS="-O3 -fPIC" FCFLAGS="-O3 -fPIC" \
    ./configure --prefix=$HDFDIR --enable-fortran --enable-parallel

make -j72
make install
```

#### NetCDF-C

```bash
cd $BUILD_DIR

wget https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.9.2.tar.gz
tar xvzf v4.9.2.tar.gz
cd netcdf-c-4.9.2

CC=mpicc FC=mpifort \
    CPPFLAGS="-I$HDFDIR/include" \
    CFLAGS="-O3 -fPIC -I$HDFDIR/include" \
    FFLAGS="-O3 -fPIC -I$HDFDIR/include" \
    FCFLAGS="-O3 -fPIC -I$HDFDIR/include" \
    LDFLAGS="-O3 -fPIC -L$HDFDIR/lib -lhdf5_hl -lhdf5 -lz" \
    ./configure --prefix=$NETCDF

make -j72
make install
```

#### NetCDF-Fortran

```bash
cd $BUILD_DIR

wget https://github.com/Unidata/netcdf-fortran/archive/refs/tags/v4.6.1.tar.gz
tar xvzf v4.6.1.tar.gz
cd netcdf-fortran-4.6.1/

CC=mpicc FC=mpifort \
    CPPFLAGS="-I$HDFDIR/include" \
    CFLAGS="-O3 -fPIC -I$HDFDIR/include" \
    FFLAGS="-O3 -fPIC -I$HDFDIR/include" \
    FCFLAGS="-O3 -fPIC -I$HDFDIR/include" \
    LDFLAGS="-O3 -fPIC -L$HDFDIR/lib -lhdf5_hl -lhdf5 -lz" \
    ./configure --prefix=$NETCDF

make -j72
make install
```

### Build WRF with NVIDIA Compilers

```bash
cd $BUILD_DIR

wget https://github.com/wrf-model/WRF/releases/download/v4.5.2/v4.5.2.tar.gz
tar xvzf v4.5.2.tar.gz
cd WRFV4.5.2
```

Run `./configure` and select the following options:

- Choose a `dm+sm` option on the `NVHPC` row. In this example, this is option **20**.
- Choose **1** for nesting.

```
./configure
```

```
------------------------------------------------------------------------
Please select from among the following Linux aarch64 options:

  1. (serial)   2. (smpar)   3. (dmpar)   4. (dm+sm)   GNU (gfortran/gcc)
  5. (serial)   6. (smpar)   7. (dmpar)   8. (dm+sm)   GNU (gfortran/gcc)
  9. (serial)  10. (smpar)  11. (dmpar)  12. (dm+sm)   armclang (armflang/armclang): Aarch64
 13. (serial)  14. (smpar)  15. (dmpar)  16. (dm+sm)   GCC (gfortran/gcc): Aarch64
 17. (serial)  18. (smpar)  19. (dmpar)  20. (dm+sm)   NVHPC (nvfortran/nvc)

Enter selection [1-16] : 16
------------------------------------------------------------------------
Compile for nesting? (0=no nesting, 1=basic, 2=preset moves, 3=vortex following) [default 0]: 1
```

```admonish important
Depending on the compilers available in your environment, other options may be presented in the menu. Check the numbers in the menu before making your selection.
```


Reset environment variables:

```bash
# Reset build environment to include `-lnetcdf` in LDFLAGS
export CC=$(which mpicc)
export CXX=$(which mpicxx)
export FC=$(which mpifort)
export CPPFLAGS="-O3 -fPIC -I$HDFDIR/include"
export CFLAGS="-O3 -fPIC -I$HDFDIR/include"
export FFLAGS="-O3 -fPIC -I$HDFDIR/include"
export LDFLAGS="-O3 -fPIC -L$HDFDIR/lib -lnetcdf -lhdf5_hl -lhdf5 -lz"
export PATH=$NETCDF/bin:$PATH
export LD_LIBRARY_PATH=$$NETCDF/lib:$LD_LIBRARY_PATH
```

Set stack size to "unlimited":
```bash
ulimit -s unlimited
```

Run `./compile` to build WRF and save the output to `build.log`:
```bash
./compile em_real 2>&1 | tee build.log
```

Look for a message similar to this at the end of the compilation log:

```
==========================================================================
build started:   Wed Oct  4 05:07:19 PM PDT 2023
build completed: Wed Oct 4 05:07:44 PM PDT 2023

--->                  Executables successfully built                  <---

-rwxrwxr-x 1 jlinford jlinford 44994360 Oct  4 17:07 main/ndown.exe
-rwxrwxr-x 1 jlinford jlinford 44921440 Oct  4 17:07 main/real.exe
-rwxrwxr-x 1 jlinford jlinford 44481744 Oct  4 17:07 main/tc.exe
-rwxrwxr-x 1 jlinford jlinford 48876800 Oct  4 17:07 main/wrf.exe

==========================================================================
```

## Run WRF CONUS 12km

Verify that the most recent NVIDIA HPC SDK is available in your environment. The simplest way to do this is to load the nvhpc module file.
```bash
module load nvhpc
```

Configure your environment.
```bash
export BUILD_DIR="$HOME/WRF"
export HDFDIR=$BUILD_DIR/opt
export HDF5=$BUILD_DIR/opt
export NETCDF=$BUILD_DIR/opt
export PATH=$HDFDIR/bin:$PATH
export LD_LIBRARY_PATH=$HDFDIR/lib:$LD_LIBRARY_PATH
```


Download and unpack the CONUS 12km input files into a fresh run directory.

```bash
cd $BUILD_DIR/WRFV4.5.2

# Copy the run directory template
cp -a run run_CONUS12km
cd run_CONUS12km

# Download the test case files and merge them into the run directory
wget https://www2.mmm.ucar.edu/wrf/src/conus12km.tar.gz
tar xvzf conus12km.tar.gz --strip-components=1
```

Configure the environment:
```bash
ulimit -s unlimited
export PATH=$NETCDF/bin:$HDFDIR/bin:$PATH
export LD_LIBRARY_PATH=$NETCDF/lib:$HDFDIR/lib:$LD_LIBRARY_PATH
```

On a Grace CPU Superchip with 144 cores, run WRF with 36 MPI ranks and give each MPI rank 4 OpenMP threads:
```bash
export OMP_STACKSIZE=1G 
export OMP_PLACES=cores 
export OMP_PROC_BIND=close 
export OMP_NUM_THREADS=4 
mpirun -np 36 -map-by ppr:18:numa:PE=4 ./wrf.exe
```

On a Grace Hopper Superchip with 72 cores, run WRF with 18 MPI ranks and give each MPI rank 4 OpenMP threads:
```bash
export OMP_STACKSIZE=1G 
export OMP_PLACES=cores 
export OMP_PROC_BIND=close 
export OMP_NUM_THREADS=4 
mpirun -np 18 -map-by ppr:18:numa:PE=4 ./wrf.exe
```

You can monitor the run progress by watching the output logs for MPI rank 0:
```bash
tail -f rsl.out.0000 rsl.error.0000
```

The benchmark score is the average elapsed seconds per domain for all MPI ranks. You can use the `jq` utility command to calculate this easily from the output logs of all MPI ranks.

```bash
# Quickly calculate the average elapsed seconds per domain as a figure-of-merit
cat rsl.out.* | grep 'Timing for main:' | awk '{print $9}' | jq -s add/length
```

## Reference Results: CONUS 12km 

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

| System              | Capacity (GB) | Ranks | Threads | Average Elapsed Seconds |
| ------------------- | ------------- | ----- | ------- | ----------------------- |
| Grace CPU Superchip | 480           | 36    | 4       | 0.3884                  |
| Grace Hopper        | 120           | 18    | 4       | 0.5761                  |

