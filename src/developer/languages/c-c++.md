# C/C++ on NVIDIA Grace

There are many C/C++ compilers available for NVIDIA Grace including:
 * [NVIDIA HPC Compiler](https://developer.nvidia.com/hpc-sdk)
 * [Clang/LLVM](https://developer.nvidia.com/grace/clang)
 * [GCC](https://gcc.gnu.org/)
 * [Arm Compiler for Linux (ACfL)](https://developer.arm.com/downloads/-/arm-compiler-for-linux)
 * [HPE Cray Compiler](https://buy.hpe.com/us/en/software/high-performance-computing-software/high-performance-computing-software/high-performance-computing-software/hpe-cray-programming-environment/p/1012707351)

## Selecting a Compiler

The compiler you use depends on your application's needs. If in doubt, try the [NVIDIA HPC Compiler](https://developer.nvidia.com/hpc-sdk) first because this compiler will always have the most recent updates and enhancements for Grace. If you prefer Clang, NVIDIA provides builds of [Clang for NVIDIA Grace](https://developer.nvidia.com/grace/clang) that are supported by NVIDIA and certified as a CUDA host compiler. GCC, ACfL, and HPE Cray Compilers also have their own advantages. As a general strategy, default to an NVIDIA-provided compiler and fall back to a third-party as needed.

When possible, use the latest compiler version that is available on your system. Newer compilers provide better support and optimizations for Grace, and when using newer compilers, many codes will demonstrate significantly better performance.

## Recommended Compiler Flags

The NVIDIA HPC Compilers accept PGI flags and many GCC or Clang compiler flags. These compilers include the NVFORTRAN, NVC++, and NVC compilers. They work with an assembler, linker, libraries, and header files on your target system, and include a CUDA toolchain, libraries and header files for GPU computing. Refer to the [NVIDIA HPC Compiler's User's Guide](https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-user-guide/index.html) for more information. The freely [NVIDIA HPC SDK](https://developer.nvidia.com/hpc-sdk) is the best way to quickly get started with the NVIDIA HPC Compiler.

Most compiler flags for GCC and Clang/LLVM operate the same on Arm64 as on other architectures except for the `-mcpu` flag.  On Arm64, this flag both specifies the appropriate architecture and the tuning strategy.  It is generally better to use `-mcpu` instead of `-march` or `-mtune` on Grace.  You can find additional details in [this presentation given at Stony Brook University](https://www.stonybrook.edu/commcms/ookami/_pdf/Linford_OokamiUGM_2022.pdf).

CPU          | Flag                | GCC version | LLVM verison
-------------|---------------------|-------------|-------------
NVIDIA Grace | `-mcpu=neoverse-v2` | 12.3+       | 16+
Ampere Altra | `-mcpu=neoverse-n1` | 9+          | 10+
Any Arm64    | `-mcpu=native`      | 9+          | 10+

If you're cross compiling, use the appropriate `-mcpu` option for your target CPU, for example, to target NVIDIA Grace when compiling on an AWS Graviton 3 use `-mcpu=neoverse-v2`.


## Compiler-Supported Hardware Features

The common `-mcpu=native` flag enables all instructions supported by the host CPU.  You can check which Arm features GCC will enable with the `-mcpu=native` flag by running the following command:
```bash
gcc -dM -E -mcpu=native - < /dev/null | grep ARM_FEATURE
```

For example, on the NVIDIA Grace CPU with GCC 12.3, we see "`__ARM_FEATURE_ATOMICS 1`" indicating that LSE atomics are enabled:
```c
$ gcc -dM -E -mcpu=native - < /dev/null | grep ARM_FEATURE
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

Header-based intrinsics translation tools, such as [SIMDe](https://github.com/simd-everywhere/simde) and [SSE2NEON](https://github.com/DLTcollab/sse2neon) are a great way to quickly get a prototype running on Arm64. These tools automatically translate x86 intrinsics to SVE or NEON intrinsics (refer to [Arm Single-Instruction Multiple-Data Instructions](../vectorization.md)). This approach provides a quick starting point when porting performance critical codes and shortens the time needed to get a working program that can be used to extract profiles and to identify hot paths in the code. After a profile is established, the hot paths can be rewritten to avoid the overhead of the automatic translation of intrinsics.

**Note:** GCC's `__sync` built-ins are outdated and might be biased towards the x86 memory model.  
Use `__atomic` versions of these functions instead of the `__sync` versions.  
[Refer to the GCC documentation for more information](https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html).


## Signedness of the `char` Type

The C standard does not specify the signedness of the `char` type. On x86, many compilers assume that `char` is signed by default, but on Arm64, compilers often assume it is unsigned by default. This difference can be addressed by using standard integer types that specify signedness (for example, `uint8_t` and `int8_t`) or by specifying `char` signedness with compiler flags, for example, `-fsigned-char` or `-funsigned-char`.

## Arm Instructions for Machine Learning

NVIDIA Grace supports [Arm dot-product instructions](https://community.arm.com/developer/tools-software/tools/b/tools-software-ides-blog/posts/exploring-the-arm-dot-product-instructions) (commonly used for Machine Learning (quantized) inference workloads) and [half precision floating point (FP16)](https://developer.arm.com/documentation/100067/0612/Other-Compiler-specific-Features/Half-precision-floating-point-intrinsics). These features enable performant and power efficient machine learning by doubling the number of operations per second and reducing the memory footprint compared to single precision floating point, all while  enjoying large dynamic range. These features are enabled automatically in the NVIDIA compilers. To enable these features in GNU or LLVM compilers, compile with `-mcpu=native` or `-mcpu=neoverse-v2`.
