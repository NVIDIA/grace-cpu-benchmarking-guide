# Locks, Synchronization, and Atomics 

Efficient synchronization is critical to achieving good performance in applications with high thread counts, but synchronization is a complex and nuanced topic.  See below for a high level overview.  If you would like a more detailed look, see this whitepaper from Arm: [Synchronization Overview and Case Study on Arm Architecture](https://developer.arm.com/documentation/107630/1-0/?lang=en).


## The Arm Memory Model
One of the most significant differences between Arm and x86 CPUs is their memory model. The Arm architecture has a weak memory model that allows for more compiler and hardware optimization to boost system performance.  This differs from the x86 architecture Total Store Order (TSO) model.  Different memory models can cause low-level codes (e.g. drivers) to function well on one architecture but encounter performance problem or failure on the other.

**Generally speaking, you should only care about the Arm memory model if you are writing low level code, e.g. assembly language.** The details of Arm's memory model lie well below the application level and will be completely invisible to most users.  If you are writing in a high level language like C/C++ or Fortran, you do not need to know the nuances of Arm's memory model.  The one exception to this general rule is code that uses boutique synchronization constructs instead of standard best practices, e.g. using `volatile` as a means of thread synchronization.  Deviating from established standards or ignoring best practices results in code that is almost guaranteed to be broken.  It should be rewritten using system-provided locks, conditions, etc. and the `stdatomic` tools.  [Here's an example of such a bug](https://github.com/ParRes/Kernels/issues/611).

Arm isn't the only architecture using a weak memory model.  If your application already runs on CPUs that aren't x86-based, you're likely to encounter fewer bugs related to the weak memory model.  In particular, if your application has been ported to a CPU implementing the POWER architecture (e.g. IBM POWER9) then it is likely to work perfectly on the Arm memory model.


## Large-System Extension (LSE) Atomic Instructions
All server-class Arm64 processors (e.g. NVIDIA Grace) have support for the Large-System Extension (LSE) which was first introduced in Armv8.1. LSE provides low-cost atomic operations which can improve system throughput for thread communication, locks, and mutexes. On recent Arm64 CPUs, the improvement can be up to an order of magnitude when using LSE atomics instead of load/store exclusives.  Note that this is not generally true for older Arm64 CPUs like the Marvell ThunderX2 or the Fujitsu A64FX.  Please see [these slides from the ISC 2022 AHUG Workshop](https://agenda.isc-hpc.com/media/slides_pdf/0900_Arm_HPC_User_Group_at_ISC22_wxIExtw.pdf) for more details.

You'll get the best performance from a POSIX threads library that uses LSE atomic instructions.  LSE atomics are important for locking and thread synchronization routines.  Many Linux distributions provide a libc compiled with LSE instructions.  For example:
 - RHEL 8.4 and later
 - Ubuntu 20.04 and later

When building an application from source, the compiler needs to generate LSE atomic instructions for applications that use atomic operations.  For example, the code of databases like PostgreSQL contain atomic constructs; c++11 code with `std::atomic` statements translate into atomic operations.  Since GCC 9.4, GCC's `-mcpu=native` flag enables all instructions supported by the host CPU, including LSE.  To confirm that LSE instructions are created, the output of `objdump` command line utility should contain LSE instructions:
```bash
$ objdump -d app | grep -i 'cas\|casp\|swp\|ldadd\|stadd\|ldclr\|stclr\|ldeor\|steor\|ldset\|stset\|ldsmax\|stsmax\|ldsmin\|stsmin\|ldumax\|stumax\|ldumin\|stumin' | wc -l
```
To check whether the application binary contains load and store exclusives:
```bash
$ objdump -d app | grep -i 'ldxr\|ldaxr\|stxr\|stlxr' | wc -l
```