# How to Use This Guide

This guide is for end users and application developers working with the [NVIDIA® Grace CPU](https://www.nvidia.com/en-us/data-center/grace-cpu/) who want to achieve optimal performance for key benchmarks and applications (workloads).  It includes procedures, sample code, reference performance numbers, recommendations, and technical best practices directly related to the NVIDIA Grace CPU.  

This guide is a is a living document and frequently updated with the latest recommendations, so it is best read online at <https://nvidia.github.io/grace-cpu-benchmarking-guide/>.  If you want to help improve the guide, you can [create a Github issue](https://github.com/NVIDIA/grace-cpu-benchmarking-guide/issues/new) at <https://github.com/NVIDIA/grace-cpu-benchmarking-guide/issues/new>.

**Note**: The performance numbers presented here are guidelines, not absolute figures.  Workload performance depends on many aspects of the system (for example, memory bandwidth), so your measured performance may be different.  The relevant system specifications are provided alongside the performance numbers to help compare your results.  Following the instructions given in this guide will help you realize the best possible performance for your particular system.

The guide is divided into the following sections:

* [Platform Configuration](platform.md): This section helps you tune your system for benchmarking. The instructions will help optimize the platform configuration.
* [Foundational Benchmarks](foundations/index.md): After checking the platform configuration, this section helps you complete a [sanity check](foundations/index.md) and confirm that the system is healthy.
* [Common Benchmarks](benchmarks/index.md): This section has information about the industry-recognized benchmarks and mini-apps that represent the performance of key workloads.
* [Applications](applications/index.md): This section has information about maximizing the performance of full applications. 
* [Developer Best Practices](developer/index.md): This section has general best practices information to develop for NVIDIA Grace.

The sections can be read in any order, but we strongly recommend you begin by [tuning](platform.md) and [sanity checking](foundations/index.md) your platform.
