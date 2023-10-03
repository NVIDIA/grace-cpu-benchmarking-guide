# Fortran on NVIDIA Grace

There are many Fortran compilers available for NVIDIA Grace including the following:

 * [NVIDIA HPC Compiler](https://developer.nvidia.com/hpc-sdk)
 * [GFORTRAN](https://gcc.gnu.org/)
 * [Arm Compiler for Linux (ACfL)](https://developer.arm.com/downloads/-/arm-compiler-for-linux)
 * [HPE Cray Compiler](https://buy.hpe.com/us/en/software/high-performance-computing-software/high-performance-computing-software/high-performance-computing-software/hpe-cray-programming-environment/p/1012707351)

## Selecting a Compiler

The compiler you use depends on your application's needs. If in doubt, try the [NVIDIA HPC Compiler](https://developer.nvidia.com/hpc-sdk) first because this compiler will always have the most recent updates and enhancements for Grace. GFORTRAN, ACfL, and HPE Cray Compilers also have their own advantages. As a general strategy, default to an NVIDIA-provided compiler and fall back to a third-party as needed.

When possible, use the latest compiler version that is available on your system. Newer compilers provide better support and optimizations for Grace, and when using newer compilers, many codes will demonstrate significantly better performance.

## Recommended Compiler Flags

The NVIDIA HPC Compilers accept PGI flags and many GCC or Clang compiler flags. These compilers include the NVFORTRAN, NVC++, and NVC compilers. They work with an assembler, linker, libraries, and header files on your target system, and include a CUDA toolchain, libraries and header files for GPU computing. Refer to the [NVIDIA HPC Compiler's User's Guide](https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-user-guide/index.html) for more information. The freely [NVIDIA HPC SDK](https://developer.nvidia.com/hpc-sdk) is the best way to quickly get started with the NVIDIA HPC Compiler.

Most compiler flags for GCC and Clang/LLVM operate the same on Arm64 as on other architectures except for the `-mcpu` flag.  On Arm64, this flag both specifies the appropriate architecture and the tuning strategy.  It is generally better to use `-mcpu` instead of `-march` or `-mtune` on Grace.  You can find additional details in [this presentation given at Stony Brook University](https://www.stonybrook.edu/commcms/ookami/_pdf/Linford_OokamiUGM_2022.pdf).

| CPU          | Flag                | GFORTRAN version |
| ------------ | ------------------- | ---------------- |
| NVIDIA Grace | `-mcpu=neoverse-v2` | 12.3+            |
| Ampere Altra | `-mcpu=neoverse-n1` | 9+               |
| Any Arm64    | `-mcpu=native`      | 9+               |

If you're cross compiling, use the appropriate `-mcpu` option for your target CPU, for example, to target NVIDIA Grace when compiling on an AWS Graviton 3 use `-mcpu=neoverse-v2`.
