# Grace CPU Benchmarking Guide
[![License](https://img.shields.io/badge/License-BSD_3--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)

This guide includes how-to guides, sample code, recommendations, and technical best practices to help achieve optimal performance for key benchmarks and applications (workloads) running on a NVIDIA Grace CPU.

For more details, **see the accompanying [NVIDIA Grace CPU Benchmarking Guide](LINK_HERE).**

* [Foundations](foundations/README.md)
  * [Speed-of-light FMA](foundations/fma.md)
  * [STREAM](foundations/stream-cpu.md)
* [Benchmarks](benchmarks/README.md)
  * [SPEC CPU 2017](benchmarks/speccpu-2k17.md)
  * [HPL](benchmarks/hpl-cpu/hpl-cpu.md)
  * [Spark k-means](benchmarks/spark_k-means.md)
  * [Graph500](benchmarks/gapbs_bfs.md)
  * [pyperf](benchmarks/pyperf.md)
  * [NAS Parallel Benchmarks](benchmarks/npb.md)
* [Applications](applications/README.md)
  * [WRF](applications/wrf.md)
  * [OpenFOAM](applications/openfoam.md)
  * [NWChem](applications/nwchem.md)
  * [NAMD](applications/namd.md)
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
* Additional Resources
  * [Grace Platform Tuning Guide](LINK_HERE)
  * [Neoverse V2 Software Optimization Guide](GET_LINK_FROM_ARM)
  * [Armv9 reference manual](LINK_HERE)

## License

Unless otherwise indicated, this work is licensed under
[The 3-Clause BSD License](https://opensource.org/license/bsd-3-clause/).  Individual examples or attached source code may be under a different license.  Check the related README or LICENSE files.

## Acknowledgements

ACKNOWLEDGEMENTS ...
