# Benchmarking Software Environment

Begin by installing all available software updates, for example,  `sudo apt update && sudo apt upgrade` on Ubuntu. Use the command `ldd --version` to check that GNU binutils version is 2.38 or later.
For best performance, GCC should be at version 12.3 or later. `gcc --version` will report the GCC version.
 
## A Recommended Software Stack

This guide shows a variety of compilers, libraries, and tools.  Suggested minimum versions of the major software packages used in this guide are shown below,
but any recent version of these tools will work well on NVIDIA Grace.

| Package                                                                                | Minimum Version | Download Link                                                                     |
| -------------------------------------------------------------------------------------- | --------------- | --------------------------------------------------------------------------------- |
| [NVIDIA HPC SDK](https://developer.nvidia.com/hpc-sdk)                                 | 23.11           | <https://developer.nvidia.com/hpc-sdk-downloads>                                  |
| [NVIDIA Clang for Grace](https://developer.nvidia.com/grace/clang)                     | 16.0.5          | <https://developer.nvidia.com/grace/clang/downloads>                              |
| [GNU Binutils](https://ftp.gnu.org/gnu/binutils)                                       | 2.41            | <https://ftp.gnu.org/gnu/binutils/binutils-2.41.tar.xz>                           |
| [GNU GCC](https://ftp.gnu.org/gnu/gcc)                                                 | 12.3            | <https://ftp.gnu.org/gnu/gcc/gcc-12.3.0/gcc-12.3.0.tar.xz>                        |
| [UCX](https://openucx.org/)                                                            | 1.14.1          | <https://github.com/openucx/ucx/releases/tag/v1.14.1>                             |
| [OpenMPI](https://www.open-mpi.org)                                                    | 4.1.5           | <https://www.open-mpi.org/software/ompi/v4.1/>                                    |
| [MVAPICH2](https://mvapich.cse.ohio-state.edu)                                         | 2.3.7           | <https://mvapich.cse.ohio-state.edu/download/mvapich/mv2/mvapich2-2.3.7-1.tar.gz> |
| [PAPI](https://icl.utk.edu/papi/)                                                      | 7.0.1           | <https://github.com/icl-utk-edu/papi>                                             |
| [Arm Compiler for Linux](https://developer.arm.com/downloads/-/arm-compiler-for-linux) | 23.04           | <https://developer.arm.com/downloads/-/arm-compiler-for-linux>                    |

