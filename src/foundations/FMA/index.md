# Fused Multiply Add

NVIDIA provides an open source suite of benchmarking microkernels for Arm® CPUs. To
allow precise counts of instructions and exercise specific functional units, these kernels
are written in assembly language. To measure the peak floating point capability of a core
and check the CPU clock speed, use a Fused Multiply Add (FMA) kernel.


## Install

To measure achievable peak performance of a core, the fp64_sve_pred_fmla kernel
executes a known number of SVE predicated fused multiply-add operations (FMLA). When
combined with the perf tool, you can measure the performance and the core clock speed.

```bash
git clone https://github.com/NVIDIA/arm-kernels.git
cd arm-kernels
make
```

## Execute

The benchmark score is reported in giga-operations per second (Gop/sec) near the top of
the benchmark output. Grace can perform 16 FP64 FMA operations per cycle, so a Grace
CPU with a nominal CPU frequency of 3.3GHz should report between 52 and 53 Gop/sec.

```bash
./arithmetic/fp64_sve_pred_fmla.x
```
```
4( 16(SVE_FMLA_64b) );
Iterations;100000000
Total Inst;6400000000
Total Ops;25600000000
Inst/Iter;64
Ops/Iter;256
Seconds;0.478488
GOps/sec;53.5019
```

Use the `perf` command to measure CPU frequency.  The CPU frequency is reported in the perf output on the cycles line and after the `#` symbol.

Before running `perf`, check that the value of `/proc/sys/kernel/perf_event_paranoid` is less than 1.  If it is, you can run the command as an unprivileged user.
```bash
perf stat ./arithmetic/fp64_sve_pred_fmla.x
```

If the value of `/proc/sys/kernel/perf_event_paranoid` is greater than 1, you will need to run `perf` as root.
```bash
sudo perf stat ./arithmetic/fp64_sve_pred_fmla.x
```

## Reference Results

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

The key performance metric is giga-operations per second (Gop/sec).  Grace can perform 16 FP64 FMA operations per cycle, so a Grace CPU with a nominal CPU frequency of 3.3GHz should report between 52 and 53 Gop/sec.

Here is an example of benchmark output:
```bash
$ perf stat ./arithmetic/fp64_sve_pred_fmla.x
4( 16(SVE_FMLA_64b) );
Iterations;100000000
Total Inst;6400000000
Total Ops;25600000000
Inst/Iter;64
Ops/Iter;256
Seconds;0.481267
GOps/sec;53.1929


 Performance counter stats for './arithmetic/fp64_sve_pred_fmla.x':


            482.25 msec task-clock                       #    0.996 CPUs utilized
                 0      context-switches                 #    0.000 /sec
                 0      cpu-migrations                   #    0.000 /sec
                65      page-faults                      #  134.786 /sec
     1,607,949,685      cycles                           #    3.334 GHz
     6,704,065,953      instructions                     #    4.17  insn per cycle
   <not supported>      branches
            18,383      branch-misses                    #    0.00% of all branches


       0.484136320 seconds time elapsed


       0.482678000 seconds user
       0.000000000 seconds sys
```
