# Fortran on NVIDIA Grace

There are many Fortran compilers available for NVIDIA Grace including:
 * [NVIDIA HPC Compiler](https://developer.nvidia.com/hpc-sdk)
 * [GFORTRAN](https://gcc.gnu.org/)
 * [Arm Compiler for Linux (ACfL)](https://developer.arm.com/downloads/-/arm-compiler-for-linux)
 * [HPE Cray Compiler](https://buy.hpe.com/us/en/software/high-performance-computing-software/high-performance-computing-software/high-performance-computing-software/hpe-cray-programming-environment/p/1012707351)

## Selecting a Compiler

Which compiler you use will depend on your application's needs.  If in doubt, try the NVIDIA HPC Compiler first as this compiler will always have the most recent updates and enhancements for Grace.  GFORTRAN, ACfL, and HPE Cray Compilers also have their own advantages.  As a general strategy, default to an NVIDIA-provided compiler and fall back to a third-party as needed.  

Whenever possible, use the latest compiler version available on your system. Newer compilers provide better support and optimizations for Arm64 processors, and many codes will demonstrate significantly better performance when using newer compilers.  


## Recommended Compiler Flags

The NVIDIA HPC Compilers are a direct decedent of the popular PGI compilers, so they accept PGI flags as well as many GCC or Clang compiler flags. These compilers include the NVFORTRAN, NVC++ and NVC compilers. They work in conjunction with an assembler, linker, libraries and header files on your target system, and include a CUDA toolchain, libraries and header files for GPU computing. [See the users guide for a detailed description of all supported compiler flags](https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-user-guide/index.html).  The [freely-available NVIDIA HPC SDK](https://developer.nvidia.com/hpc-sdk) is the best way to quickly get started with the NVIDIA HPC Compiler.

GCC and Clang/LLVM operate more-or-less the same on Arm64 as on other architectures except for the `-mcpu` flag.  On Arm64, the `-mcpu` flag both specifies the appropriate architecture and the tuning strategy.  It's generally better to use `-mcpu` instead of `-march` or `-mtune` on Arm64.  You can find additional details in [this presentation given at Stony Brook University](https://www.stonybrook.edu/commcms/ookami/_pdf/Linford_OokamiUGM_2022.pdf).

CPU          | Flag                | GFORTRAN version  
-------------|---------------------|-----------------
NVIDIA Grace | `-mcpu=neoverse-v2` | 12.3+            
Ampere Altra | `-mcpu=neoverse-n1` | 9+               
Any Arm64    | `-mcpu=native`      | 9+               

If you're cross compiling, use the appropriate `-mcpu` option for your target CPU, e.g. use `-mcpu=neoverse-v2` to target NVIDIA Grace when compiling on an AWS Graviton 3.
