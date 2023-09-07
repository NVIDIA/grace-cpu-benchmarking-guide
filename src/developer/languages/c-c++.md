# C/C++ on NVIDIA Grace

There are many C/C++ compilers available for NVIDIA Grace including:
 * [NVIDIA HPC Compiler](https://developer.nvidia.com/hpc-sdk)
 * [Clang/LLVM](https://developer.nvidia.com/grace/clang)
 * [GCC](https://gcc.gnu.org/)
 * [Arm Compiler for Linux (ACfL)](https://developer.arm.com/downloads/-/arm-compiler-for-linux)
 * [HPE Cray Compiler](https://buy.hpe.com/us/en/software/high-performance-computing-software/high-performance-computing-software/high-performance-computing-software/hpe-cray-programming-environment/p/1012707351)

## Selecting a Compiler

Which compiler you use will depend on your application's needs.  If in doubt, try the NVIDIA HPC Compiler first as this compiler will always have the most recent updates and enhancements for Grace.  Or, if you prefer Clang, [NVIDIA provides builds of Clang for NVIDIA Grace](https://developer.nvidia.com/grace/clang) that are supported by NVIDIA and certified as a CUDA host compiler.  GCC, ACfL, and HPE Cray Compilers also have their own advantages.  As a general strategy, default to an NVIDIA-provided compiler and fall back to a third-party as needed.  

Whenever possible, use the latest compiler version available on your system. Newer compilers provide better support and optimizations for Arm64 processors, and many codes will demonstrate significantly better performance when using newer compilers.  

## Recommended Compiler Flags

The NVIDIA HPC Compilers are a direct decedent of the popular PGI compilers, so they accept PGI flags as well as many GCC or Clang compiler flags. These compilers include the NVFORTRAN, NVC++ and NVC compilers. They work in conjunction with an assembler, linker, libraries and header files on your target system, and include a CUDA toolchain, libraries and header files for GPU computing. [See the users guide for a detailed description of all supported compiler flags](https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-user-guide/index.html).  The [freely-available NVIDIA HPC SDK](https://developer.nvidia.com/hpc-sdk) is the best way to quickly get started with the NVIDIA HPC Compiler.

GCC and Clang/LLVM operate more-or-less the same on Arm64 as on other architectures except for the `-mcpu` flag.  On Arm64, the `-mcpu` flag both specifies the appropriate architecture and the tuning strategy.  It's generally better to use `-mcpu` instead of `-march` or `-mtune` on Arm64.  You can find additional details in [this presentation given at Stony Brook University](https://www.stonybrook.edu/commcms/ookami/_pdf/Linford_OokamiUGM_2022.pdf).

CPU          | Flag                | GCC version | LLVM verison
-------------|---------------------|-------------|-------------
NVIDIA Grace | `-mcpu=neoverse-v2` | 12.3+       | 16+
Ampere Altra | `-mcpu=neoverse-n1` | 9+          | 10+
Any Arm64    | `-mcpu=native`      | 9+          | 10+

If you're cross compiling, use the appropriate `-mcpu` option for your target CPU, e.g. use `-mcpu=neoverse-v2` to target NVIDIA Grace when compiling on an AWS Graviton 3.


## Compiler-supported Hardware Features

The common `-mcpu=native` flag enables all instructions supported by the host CPU.  You can check which Arm features GCC will enable with the `-mcpu=native` flag by using this command:
```bash
gcc -dM -E -mcpu=native - < /dev/null | grep ARM_FEATURE
```

For example, on the NVIDIA Grace CPU with GCC 12.3, we see "`__ARM_FEATURE_ATOMICS 1`" indicating that LSE atomics are enabled:
```c
jlinford@fc01-gg01:~$ gcc -dM -E -mcpu=native - < /dev/null | grep ARM_FEATURE
#define __ARM_FEATURE_ATOMICS 1
#define __ARM_FEATURE_SM3 1
#define __ARM_FEATURE_SM4 1
#define __ARM_FEATURE_RCPC 1
#define __ARM_FEATURE_SVE_VECTOR_OPERATORS 1
#define __ARM_FEATURE_SVE2_AES 1
#define __ARM_FEATURE_AES 1
#define __ARM_FEATURE_SVE 1
#define __ARM_FEATURE_IDIV 1
#define __ARM_FEATURE_JCVT 1
#define __ARM_FEATURE_DOTPROD 1
#define __ARM_FEATURE_BF16_SCALAR_ARITHMETIC 1
#define __ARM_FEATURE_MATMUL_INT8 1
#define __ARM_FEATURE_CRYPTO 1
#define __ARM_FEATURE_BF16_VECTOR_ARITHMETIC 1
#define __ARM_FEATURE_FRINT 1
#define __ARM_FEATURE_FP16_SCALAR_ARITHMETIC 1
#define __ARM_FEATURE_CLZ 1
#define __ARM_FEATURE_SHA512 1
#define __ARM_FEATURE_QRDMX 1
#define __ARM_FEATURE_FMA 1
#define __ARM_FEATURE_SHA2 1
#define __ARM_FEATURE_SVE2_SHA3 1
#define __ARM_FEATURE_COMPLEX 1
#define __ARM_FEATURE_FP16_VECTOR_ARITHMETIC 1
#define __ARM_FEATURE_SVE2_SM4 1
#define __ARM_FEATURE_SVE_MATMUL_INT8 1
#define __ARM_FEATURE_FP16_FML 1
#define __ARM_FEATURE_UNALIGNED 1
#define __ARM_FEATURE_SHA3 1
#define __ARM_FEATURE_CRC32 1
#define __ARM_FEATURE_SVE_BITS 0
#define __ARM_FEATURE_NUMERIC_MAXMIN 1
#define __ARM_FEATURE_SVE2 1
#define __ARM_FEATURE_SVE2_BITPERM 1
```

## Porting SSE or AVX Intrinsics

Header-based intrinsics translation tools like [SIMDe](https://github.com/simd-everywhere/simde) and [SSE2NEON](https://github.com/DLTcollab/sse2neon) are a great way to quickly get a prototype running on Arm64.
These tools automatically translate x86 intrinsics to SVE or NEON intrinsics (see [Arm SIMD Instructions](../vectorization.md)). 
This approach provides a quick starting point in porting performance critical codes
and shortens the time needed to get a working program that then
can be used to extract profiles and to identify hot paths in the code.  Once a profile is
established, the hot paths can be rewritten to
avoid the overhead of the automatic translation of intrinsics.

Note that GCC's `__sync` built-ins are outdated and may be biased towards the x86 memory model.  
Use `__atomic` versions of these functions instead of the `__sync` versions.  
See [the GCC documentation](https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html) for more details.


## Signed vs. Unsigned char
The C standard doesn't specify the signedness of the `char` type. On x86, many compilers assume `char` is signed by
default, but on Arm, compilers often assume it is unsigned by default. This difference can be addressed by using
standard int types that explicitly specify the signedness (e.g. `uint8_t` and `int8_t`) or by explicitly specifying
`char` signedness with compiler flags, e.g. `-fsigned-char`.


## Arm Instructions for Machine Learning
NVIDIA Grace supports [Arm dot-product instructions](https://community.arm.com/developer/tools-software/tools/b/tools-software-ides-blog/posts/exploring-the-arm-dot-product-instructions) commonly used for Machine Learning (quantized) inference workloads, and [Half precision floating point (FP16)](https://developer.arm.com/documentation/100067/0612/Other-Compiler-specific-Features/Half-precision-floating-point-intrinsics).  These features enable performant and power efficient machine learning by doubling the number of operations per second and reducing the memory footprint compared to single precision floating point, all while still enjoying large dynamic range.  Compiling with `-mcpu=native` will enable these features, when available.
