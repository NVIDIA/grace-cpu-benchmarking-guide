# Platform Configuration

Before benchmarking, it's important to check the platform configuration is optimal for the target benchmark.
The optimal configuration may vary by benchmark, but there are some common high-level settings to be aware of.
For a more comprehensive guide to platform tuning, see the NVIDIA Grace Hopper and Grace CPU Platform Tuning Guide.

## Software Environment

The bare minimum requirements are:

- Install all available software updates, e.g. `sudo apt update && sudo apt upgrade` on Ubuntu.
- GNU binutils should be at version 2.38 or later. `ldd --version` will report the binutils version.
- GCC should be at version 12.3 or later. `gcc --version` will report the GCC version.
 
In addition, we strongly recommend installing the following software packages:

| Package                                                                                | Version | Download Link                                                                   |
| -------------------------------------------------------------------------------------- | ------- | ------------------------------------------------------------------------------- |
| [NVIDIA HPC SDK](https://developer.nvidia.com/hpc-sdk)                                 | 23.7    | https://developer.nvidia.com/hpc-sdk-downloads                                  |
| [NVIDIA Clang for Grace](https://developer.nvidia.com/grace/clang)                     | 16.0.5  | https://developer.nvidia.com/grace/clang/downloads                              |
| [GNU Binutils](https://ftp.gnu.org/gnu/binutils)                                       | 2.41    | https://ftp.gnu.org/gnu/binutils/binutils-2.41.tar.xz                           |
| [GNU GCC](https://ftp.gnu.org/gnu/gcc)                                                 | 13.2    | https://ftp.gnu.org/gnu/gcc/gcc-13.2.0/gcc-13.2.0.tar.xz                        |
| [UCX](https://openucx.org/)                                                            | 1.14.1  | https://github.com/openucx/ucx/releases/tag/v1.14.1                             |
| [OpenMPI](https://www.open-mpi.org)                                                    | 4.1.5   | https://www.open-mpi.org/software/ompi/v4.1/                                    |
| [MVAPICH2](https://mvapich.cse.ohio-state.edu)                                         | 2.3.7   | https://mvapich.cse.ohio-state.edu/download/mvapich/mv2/mvapich2-2.3.7-1.tar.gz |
| [PAPI](https://icl.utk.edu/papi/)                                                      | 7.0.1   | https://github.com/icl-utk-edu/papi                                             |
| [Arm Compiler for Linux](https://developer.arm.com/downloads/-/arm-compiler-for-linux) | 23.04   | https://developer.arm.com/downloads/-/arm-compiler-for-linux                    |

## CPU and Memory

Use the `performance` CPU frequency governor:

```bash
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

Disable address space layout randomization (ASLR):

```bash
sudo sysctl -w kernel.randomize_va_space=0
```

Drop caches:

```bash
echo 3 > /proc/sys/vm/drop_caches
```

Set kernel dirty page values to defaults:

```bash
echo 10 > /proc/sys/vm/dirty_ratio
echo 5 > /proc/sys/vm/dirty_background_ratio
```

Check for dirty page writeback every 60 seconds to reduce disk I/O (the default is 5 seconds):

```bash
echo 6000 > /proc/sys/vm/dirty_writeback_centisecs
```

Disable the NMI watchdog

```bash
echo 0 > /proc/sys/kernel/watchdog
```

## Networking

Set networking connection tracking size:

```bash
echo 512000 | sudo tee /proc/sys/net/netfilter/nf_conntrack_max
```

Allow the kernel to reuse TCP ports which may be in a TIME_WAIT state before kicking off the test:

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/tcp_tw_reuse
```

## Device I/O

Full power for generic devices:

```bash
for i in `find /sys/devices/*/power/control` ; do
    echo 'on' > ${i}
done
```

Full power for PCI devices:

```bash
for i in `find /sys/bus/pci/devices/*/power/control` ; do
    echo 'on' > ${i}
done
```
