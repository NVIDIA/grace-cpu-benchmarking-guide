# Grace CPU Benchmarking Guide

This guide includes how-to guides, sample code, recommendations, and technical best practices to help achieve optimal performance for key benchmarks and applications (workloads) running on a Grace CPU.

## Contents
* [Introduction to NVIDIA Grace](#introduction-to-nvidia-grace)
* [Foundations](benchmarks/README.md)
  * [Speed-of-light FMA](benchmarks/fma.md)
  * [STREAM](benchmarks/stream-cpu.md)
* [Benchmarks](benchmarks/README.md)
  * [SPEC CPU 2017](benchmarks/speccpu-2k17.md)
  * [HPL](benchmarks/hpl-cpu/hpl-cpu.md)
  * [Spark k-means](benchmarks/spark.md)
  * [Graph500](benchmarks/graph500.md)
  * [NAS Parallel Benchmarks](benchmarks/npb.md)
* [Applications](applications/README.md)
  * [WRF](applications/wrf.md)
  * [OpenFOAM](applications/openfoam.md)
  * [Tensorflow](applications/tensorflow-cpu.md)
  * [NWChem](applications/nwchem.md)
  * [NAMD](applications/namd.md)
* [Transitioning Workloads to NVIDIA Grace](transition-guide.md)
  * [Supported Software](software/README.md)
    * [Machine Learning](software/ml.md)
    * [Compilers](software/compilers.md) and [CUDA](software/cuda.md)
    * [Message Passing (MPI)](software/mpi.md)
    * [Math Libraries](software/mathlibs.md)
    * [Containers](software/containers.md)
    * [Operating Systems](software/os.md)
  * [Optimizing for NVIDIA Grace CPU](optimization/README.md)
    * [Arm SIMD Instructions: SVE and NEON](optimization/vectorization.md)
    * [Arm Atomic Instructions: LSE](optimization/atomics.md)
  * [Language-specific Considerations](languages/README.md)
    * [C/C++](languages/c-c++.md)
    * [Fortran](languages/fortran.md)
    * [Python](languages/python.md)
    * [Rust](languages/rust.md)
    * [Go](languages/golang.md)
    * [Java](languages/java.md)
    * [.NET](languages/dotnet.md)
  * [Additional Resources](#additional-resources)
* [Acknowledgements](#acknowledgements)



## Introduction to NVIDIA Grace

The NVIDIA® Grace CPU Superchip represents a revolution in compute-platform design by integrating the level of performance offered by a flagship that is based on a x86-64 two-socket workstation or a server platform into one superchip. It delivers twice the performance per watt for maximum data center throughput and improved total cost of ownership (TCO). The Grace CPU Superchip uses the NVIDIA NVLink® C2C to pack the 144 flagship Arm Neoverse V2 cores and up to 1TB/s of memory bandwidth in a 500 watt power envelope.

For more details, see the accompanying [NVIDIA Grace CPU Benchmarking Guide](LINK_HERE).



## Additional resources
 * [Grace Platform Tuning Guide](LINK_HERE)
 * [Neoverse V2 Software Optimization Guide](GET_LINK_FROM_ARM)
 * [Armv9 reference manual](LINK_HERE)
 

# License

[![CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](http://creativecommons.org/licenses/by-sa/4.0/)

Unless otherwise indicated, this work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).  Individual examples or attached source code may be under a different license.  Check the related README or LICENSE files.


# Acknowledgements
ACKNOWLEDGEMENTS ...


**Feedback?** jlinford@nvidia.com

