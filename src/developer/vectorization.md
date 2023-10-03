# Arm Single-Instructon Multiple-Data Instructions: SVE and NEON

NVIDIA Grace implements two single-instruction-multiple-data (SIMD) instruction extensions: 
- Advanced SIMD Instructions (NEON)
- Arm Scalable Vector Extensions (SVE)

Arm Advanced SIMD Instructions (or NEON) is the most common SIMD ISA for Arm64.  It is a fixed-length SIMD ISA that supports 128-bit vectors.  The first Arm-based supercomputer to appear on the Top500 Supercomputers list ([_Astra_](https://www.sandia.gov/labnews/2018/11/21/astra-2/)) used NEON to accelerate linear algebra, and many applications and libraries are already taking advantage of NEON.

More recently, Arm64 CPUs have started supporting [Arm Scalable Vector Extensions (SVE)](https://developer.arm.com/documentation/102476/latest/), which is a length-agnostic SIMD ISA that supports more datatypes than NEON (for example, FP16), offers more powerful instructions (for example, gather/scatter), and supports vector lengths of more than 128 bits.  SVE is currently found in [NVIDIA Grace](https://www.nvidia.com/en-us/data-center/grace-cpu/), the [AWS Graviton 3](https://aws.amazon.com/ec2/graviton/), [Fujitsu A64FX](https://www.fujitsu.com/global/products/computing/servers/supercomputer/a64fx/), and others.  SVE is not a new version of NEON, but an entirely new SIMD ISA.

The following table provides a quick summary of the SIMD capabilities of some of the currently available Arm64 CPUs:

|                      | NVIDIA Grace | AWS Graviton3 | Fujitsu A64FX | AWS Graviton2 | Ampere Altra |
| -------------------- | ------------ | ------------- | ------------- | ------------- | ------------ |
| CPU Core             | Neoverse V2  | Neoverse V1   | A64FX         | Neoverse N1   | Neoverse N1  |
| SIMD ISA             | SVE2 & NEON  | SVE & NEON    | SVE & NEON    | NEON only     | NEON only    |
| NEON Configuration   | 4x128        | 4x128         | 2x128         | 2x128         | 2x128        |
| SVE Configuration    | 4x128        | 2x256         | 2x512         | N/A           | N/A          |
| SVE Version          | 2            | 1             | 1             | N/A           | N/A          |
| NEON FMLA FP64 TPeak | 16           | 16            | 8             | 8             | 8            |
| SVE FMLA FP64 TPeak  | 16           | 16            | 32            | N/A           | N/A          |

Many recent Arm64 CPUs provide the same peak theoretical performance for NEON and SVE. For example, NVIDIA Grace can retire four 128-bit NEON operations or four 128-bit SVE2 operations. Although the theoretical peak performance of SVE and NEON are the same for these CPUs, SVE ([and especially SVE2](https://developer.arm.com/documentation/102340/0001/Introducing-SVE2)) is a more capable SIMD ISA with support for complex datatypes and advanced features that enable the vectorization of complicated code. In practice, kernels that cannot be vectorized in NEON can be vectorized with SVE. So, although SVE will not beat NEON in a performance drag race, it can dramatically improve the overall performance of the application by vectorizing loops that would have otherwise executed with scalar instructions.

Fortunately, auto-vectorizing compilers are usually the best choice when programming Arm SIMD ISAs. The compiler will generally make the best decision on when to use SVE or NEON, and it will take advantage of SVE's advanced auto-vectorization features more easily than a human coding in intrinsics or an assembly can. 

Avoid writing SVE or NEON intrinsics. To realize the best performance for a loop, use the appropriate command-line options with your favorite auto-vectorizing compiler. You might need to use compiler directives or make changes in the high-level code to facilitate auto-vectorization, but this will be much easier and more maintainable than writing intrinsics. Leave the ﬁner details to the compiler and focus on code patterns that auto-vectorize well.

## Compiler-Driven Auto-Vectorization

The key to maximizing auto-vectorization is to allow the compiler to take advantage of the available hardware features. By default, GCC and LLVM compilers take a conservative approach and do not enable advanced features unless explicitly told to do so. The easiest way to enable all available features for GCC or LLVM is to use the `-mcpu` compiler ﬂag. If you are compiling on the same CPU on which the code will run, use `-mcpu=native`. Otherwise, you can use `-mcpu=<target>`,  where `<target>`is one of the CPU identiﬁers, for example, `-mcpu=neoverse-v2`. 

The NVIDIA compilers take a more aggressive approach. By default, these compilers assume that the machine on which you are compiling is the machine on which you will run and enable all available hardware features that were detected at compile time. When compiling wiht the NVIDA compilers natively on Grace, you do not need additional flags.

**Note:** When possible, use the most recent version of your compiler.  For example, GCC9 supported auto-vectorization fairly well, but GCC12 has shown impressive improvement over GCC9 in most cases. GCC13 further improves auto-vectorization.

The second key compiler feature is the compiler vectorization report.  GCC uses the `-fopt-info` flags to report on auto-vectorization success or failure.  You can use the generated informational messages to guide code annotations or transformations that will facilitate autovectorization.  For example, compiling with `-fopt-info-vec-missed` will report on which loops were not vectorized.


## Relaxed Vector Conversions
Arm NEON differentiates between vectors of signed and unsigned types.  For example, GCC will not implicitly cast between vectors of signed and unsigned 64-bit integers:
```c
#include <arm_neon.h>
...
uint64x2_t u64x2;
int64x2_t s64x2;
// Error: cannot convert 'int64x2_t' to 'uint64x2_t' in assignment
u64x2 = s64x2;
```

To perform the cast, you must use NEON's `vreinterpretq` functions:
```c
u64x2 = vreinterpretq_u64_s64(s64x2);
```

Unfortunately, some codes written for other SIMD ISAs rely on these kinds of implicit conversions.  If you see errors about "no known conversion" in a code that builds for AVX but does not build for NEON, you might need to relax GCC's vector conversion rules:
```
/tmp/velox/third_party/xsimd/include/xsimd/types/xsimd_batch.hpp:35:11: note:   no known conversion for argument 1 from ‘xsimd::batch<long int>' to ‘const xsimd::batch<long unsigned int>&'
```
To allow implicit conversions between vectors with differing numbers of elements and/or incompatible element types, use the `-flax-vector-conversions` flag.  This flag should be fine for legacy code, but it should not be used for new code.  The safest option is to use the appropriate `vreinterpretq` calls.


## Runtime Detection of Supported SIMD Instructions

To make your binaries more portable across various Arm64 CPUs, use the Arm64 hardware capabilities to determine the available instructions at runtime. For example, a CPU core that is compliant with Armv8.4 must support dot-product, but dot-products are optional in Armv8.2 and Armv8.3. A developer who wants to build an application or library that can detect the supported instructions in runtime, can follow this example:

```c
#include<sys/auxv.h>
......
  uint64_t hwcaps = getauxval(AT_HWCAP);
  has_crc_feature = hwcaps & HWCAP_CRC32 ? true : false;
  has_lse_feature = hwcaps & HWCAP_ATOMICS ? true : false;
  has_fp16_feature = hwcaps & HWCAP_FPHP ? true : false;
  has_dotprod_feature = hwcaps & HWCAP_ASIMDDP ? true : false;
  has_sve_feature = hwcaps & HWCAP_SVE ? true : false;
```

The full list of Arm64 hardware capabilities is defined in the [glibc header file](https://github.com/bminor/glibc/blob/master/sysdeps/unix/sysv/linux/aarch64/bits/hwcap.h) and in the [Linux kernel](https://github.com/torvalds/linux/blob/master/arch/arm64/include/asm/hwcap.h).

## Porting Codes with SSE/AVX Intrinsics to NEON

### Detecting Arm64 systems

If you see errors like `error: unrecognized command-line option '-msse2'`, it usually means that the build system is
failing to detect Grace as an Arm CPU is incorrectly using the x86 target features compiler flags.

To detect an Arm64 system, the build system can use the following command:
```bash
(test $(uname -m) = "aarch64" && echo "arm64 system") || echo "other system"
```

To detect an Arm64 system, you can compile, run, and check the return value of a C program.
```c
# cat << EOF > check-arm64.c
int main () {
#ifdef __aarch64__
  return 0;
#else
  return 1;
#endif
}
EOF

# gcc check-arm64.c -o check-arm64
# (./check-arm64 && echo "arm64 system") || echo "other system"
```

### Translating x86 Intrinsics to NEON
When programs contain code with x86 intrinsics, drop-in intrinsic translation tools like [SIMDe](https://github.com/simd-everywhere/simde) or [sse2neon](https://github.com/DLTcollab/sse2neon) can be used to quickly obtain a working program on Arm64.  This is a good starting point for rewriting the x86 intrinsics in NEON or SVE and will quickly get a prototype up and running.  For example, to port code using AVX2 intrinsics with SIMDe:
```c
#define SIMDE_ENABLE_NATIVE_ALIASES
#include "simde/x86/avx2.h"
```

SIMDe provides a quick starting point to port performance critical codes to Arm64. It shortens the time needed to get a working program that can be used to extract proﬁles and to identify hot paths in the code. After a proﬁle is established, the hot paths can be rewritten to avoid the overhead of the generic translation.

Since you are rewriting your x86 intrinsics, you might want to take this opportunity to create a more portable version.  Here are some suggestions to consider:

 - Rewrite in native C/C++, Fortran, or another high-level compiled language. Compilers are constantly improving, and technologies like Arm SVE enable the auto-vectorization of codes that formally would not vectorize. You can avoid platform-speciﬁc intrinsics entirely and let the compiler do all the work.
 - If your application is written in C++, use `std::experimental::simd` from the C++ Parallelism Technical Specification V2 by using the `<experimental/simd>` header.
 - Use the [SLEEF Vectorized Math Library](https://sleef.org/) as a header-based set of "portable intrinsics".
 - Instead of Time Stamp Counter (TSC) RDTSC intrinsics, use standards-compliant portable timers, for example, `std::chrono` (C++), `clock_gettime` (C/POSIX), `omp_get_wtime` (OpenMP), `MPI_Wtime` (MPI), and so on.

