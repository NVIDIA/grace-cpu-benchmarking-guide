# Benchmarking Software Environment

Begin by installing all available software updates, for example,  `sudo apt update && sudo apt upgrade` on Ubuntu. Use the command `ldd --version` to check that GNU binutils version is 2.38 or later.
For best performance, GCC should be at version 12.3 or later. `gcc --version` will report the GCC version.

Many Linux distributions provide packages for GCC 12 compilers that can be installed alongside the system GCC.  For example, `sudo apt install gcc-12` on Ubuntu.  See your Linux distribution's instructions for installing and using various GCC versions.
In case your distribution does not provide these packages, or you are unable to install them, instructions for building and installing GCC are provided below.
 
## A Recommended Software Stack

This guide shows a variety of compilers, libraries, and tools.  Suggested minimum versions of the major software packages used in this guide are shown below,
but any recent version of these tools will work well on NVIDIA Grace.  Installation instructions for each package are provided in the associated link.


| Package                                                                                | Minimum Version | Link                                                                              |
| -------------------------------------------------------------------------------------- | --------------- | --------------------------------------------------------------------------------- |
| [NVIDIA HPC SDK](https://developer.nvidia.com/hpc-sdk)                                 | 23.11           | <https://developer.nvidia.com/hpc-sdk>                                            |
| [NVIDIA Clang for Grace](https://developer.nvidia.com/grace/clang)                     | 16.0.5          | <https://developer.nvidia.com/grace/clang>                                        |
| [GNU Binutils](https://ftp.gnu.org/gnu/binutils)                                       | 2.41            | <https://ftp.gnu.org/gnu/binutils/binutils-2.41.tar.xz>                           |
| [GNU GCC](https://ftp.gnu.org/gnu/gcc)                                                 | 12.3            | <https://ftp.gnu.org/gnu/gcc/gcc-12.3.0/gcc-12.3.0.tar.xz>                        |
| [UCX](https://openucx.org/)                                                            | 1.14.1          | <https://github.com/openucx/ucx/releases/tag/v1.14.1>                             |
| [OpenMPI](https://www.open-mpi.org)                                                    | 4.1.5           | <https://www.open-mpi.org/software/ompi/v4.1/>                                    |
| [MVAPICH2](https://mvapich.cse.ohio-state.edu)                                         | 2.3.7           | <https://mvapich.cse.ohio-state.edu/download/mvapich/mv2/mvapich2-2.3.7-1.tar.gz> |
| [PAPI](https://icl.utk.edu/papi/)                                                      | 7.1.0           | <https://icl.utk.edu/papi/>                                                       |
| [Arm Compiler for Linux](https://developer.arm.com/downloads/-/arm-compiler-for-linux) | 23.04           | <https://developer.arm.com/downloads/-/arm-compiler-for-linux>                    |


## Building and Installing GCC 12.3 from Source

```admonish info "Prefer Linux Distribution GCC 12 Packages"
Many Linux distributions provide packages for GCC 12 compilers that can be installed alongside the system GCC.  For example, `sudo apt install gcc-12` on Ubuntu.  You should prefer those packages over building GCC from source.
```

Follow the instructions below to build GCC 12.3 from source.  Note that filesystem I/O performance can affect compilation time, so we recommend building GCC on a local filesystem or ramdisk, e.g. `/tmp`.

Download and unpack the GCC source code:

```bash
wget https://ftp.gnu.org/gnu/gcc/gcc-12.3.0/gcc-12.3.0.tar.xz
tar xvf gcc-12.3.0.tar.xz
```

Download the GCC prerequisites:

```bash
cd gcc-12.3.0
./contrib/download_prerequisites
```

You should see output similar to:

```
2024-01-24 08:04:44 URL:http://gcc.gnu.org/pub/gcc/infrastructure/gmp-6.2.1.tar.bz2 [2493916/2493916] -> "gmp-6.2.1.tar.bz2" [1]
2024-01-24 08:04:45 URL:http://gcc.gnu.org/pub/gcc/infrastructure/mpfr-4.1.0.tar.bz2 [1747243/1747243] -> "mpfr-4.1.0.tar.bz2" [1]
2024-01-24 08:04:47 URL:http://gcc.gnu.org/pub/gcc/infrastructure/mpc-1.2.1.tar.gz [838731/838731] -> "mpc-1.2.1.tar.gz" [1]
2024-01-24 08:04:49 URL:http://gcc.gnu.org/pub/gcc/infrastructure/isl-0.24.tar.bz2 [2261594/2261594] -> "isl-0.24.tar.bz2" [1]
gmp-6.2.1.tar.bz2: OK
mpfr-4.1.0.tar.bz2: OK
mpc-1.2.1.tar.gz: OK
isl-0.24.tar.bz2: OK
All prerequisites downloaded successfully.
```

Configure, compile, and install GCC.  Remember to set `GCC_INSTALL_PREFIX` appropriately!  This example installs GCC to `/opt/gcc/12.3` but any valid filesystem path can be used:

```bash
export GCC_INSTALL_PREFIX=/opt/gcc/12.3
./configure --prefix="$GCC_INSTALL_PREFIX" --enable-languages=c,c++,fortran --enable-lto --disable-bootstrap --disable-multilib
make -j
make install
```

To use the newly-installed GCC 12 compiler, simply update your `$PATH` environment variable:

```bash
export PATH=$GCC_INSTALL_PREFIX/bin:$PATH
```

Confirm that the `gcc` command invokes GCC 12.3:

```bash
which gcc
gcc --version
```

You should see output similar to:
```
/opt/gcc/12.3/bin/gcc

gcc (GCC) 12.3.0
Copyright (C) 2022 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```