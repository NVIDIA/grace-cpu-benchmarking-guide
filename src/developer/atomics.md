# Locks, Synchronization, and Atomics 

Efficient synchronization is critical to achieving good performance in applications with high thread counts, but synchronization is a complex and nuanced topic.  See below for a high level overview (refer to the [Synchronization Overview and Case Study on Arm Architecture](https://developer.arm.com/documentation/107630/1-0/?lang=en) whitepaper from Arm for more information).


## The Arm Memory Model

One of the most signiﬁcant diﬀerences between Arm and the x86 CPUs is their memory model. The Arm architecture has a weak memory model that allows for more compiler and hardware optimization to boost system performance. This differs from the x86 architecture Total Store Order (TSO) model. Diﬀerent memory models can cause low-level codes (for example, drivers) to function well on one architecture but encounter performance problem or failure on the other.

**Note:** You should only be interested in the Arm memory model if you are writing low level code, such as assembly language. 

The details about Arm's memory model are below the application level and will be completely invisible to most users. If you are writing in a high-level language such as C, C++, or Fortran, you do not need to know the nuances of Arm's memory model. The one exception to this general rule is code that uses boutique synchronization constructs instead of standard best practices, for example, using `volatile` as a means of thread synchronization. 

Deviating from established standards or ignoring best practices results in code that is almost guaranteed to be broken.  It should be rewritten using system-provided locks, conditions, etc. and the `stdatomic` tools.  (Refer to <https://github.com/ParRes/Kernels/issues/611> for an example of this type of bug.)

Arm is not the only architecture that uses a weak memory model. If your application already runs on CPUs that are not x86-based, you might encounter fewer bugs that are related to the weak memory model. Specifically, if your application has been ported to a CPU implementing the POWER architecture, for example, IBM POWER9, the application will work on the Arm memory model.


## Large-System Extension (LSE) Atomic Instructions

All server-class Arm64 processors, such as NVIDIA Grace, have support for the Large-System Extension (LSE), which was ﬁrst introduced in Armv8.1. LSE provides low-cost atomic operations that can improve system throughput for thread communication, locks, and mutexes. On recent Arm64 CPUs, the improvement can be up to an order of magnitude when using LSE atomics instead of load/store exclusives. This improvement is not generally true for older Arm64 CPUs like the Marvell ThunderX2 or the Fujitsu A64FX (refer to [these slides from the ISC 2022 AHUG Workshop](https://agenda.isc-hpc.com/media/slides_pdf/0900_Arm_HPC_User_Group_at_ISC22_wxIExtw.pdf) for more information).

When building an application from source, the compiler needs to generate LSE atomic instructions for applications that use atomic operations. For example, the code of databases such as PostgreSQL contain atomic constructs: C++11 code with `std::atomic` statements that translate into atomic operations. Since GCC 9.4, GCC’s `-mcpu=native` ﬂag enables all instructions supported by the host CPU, including LSE. To conﬁrm that LSE instructions are created, the output of objdump command-line utility should contain LSE instructions:
```bash
$ objdump -d app | grep -i 'cas\|casp\|swp\|ldadd\|stadd\|ldclr\|stclr\|ldeor\|steor\|ldset\|stset\|ldsmax\|stsmax\|ldsmin\|stsmin\|ldumax\|stumax\|ldumin\|stumin' | wc -l
```
To check whether the application binary contains load and store exclusives, run the following command:
```bash
$ objdump -d app | grep -i 'ldxr\|ldaxr\|stxr\|stlxr' | wc -l
```