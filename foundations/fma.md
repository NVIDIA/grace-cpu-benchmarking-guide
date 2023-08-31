# Speed-of-light Fused Multiply Add (FMA)

This is a suite of open source microkernels for benchmarking Arm CPUs.  These kernels measure the machine’s performance while rapidly executing precise sequences of instructions.  In this guide, we measure the number of fused multiply-add (FMA) instructions that your system can complete in a given timespan to confirm the CPU’s core frequency and overall health.  When combined with the perf tool, this measures both the CPU performance and the CPU core clock speed simultaneously.

## Reference Results

A Grace CPU operating at approximately 3.3 GHz scores approximately 52 Gop/sec. 

## Install

```bash
git clone https://github.com/NVIDIA/arm-kernels.git
cd arm-kernels
make
```

## Execute

```bash
perf stat ./arithmetic/fp64_sve_pred_fmla.x
```
