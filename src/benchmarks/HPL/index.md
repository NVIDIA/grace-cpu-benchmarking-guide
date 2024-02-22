# High Performance Linpack (HPL)

The [NVIDIA HPC-Benchmarks](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/hpc-benchmarks) provides a multiplatform (x86 and aarch64) container image based on NVIDIA Optimized Frameworks container images that includes NVIDIA's HPL benchmark.  HPL-NVIDIA solves a random dense linear system in double precision arithmetic on distributed-memory computers and is based on the [netlib HPL benchmark](https://www.netlib.org/benchmark/hpl/).  Please visit the [NVIDIA HPC-Benchmarks page in the NGC Catalog](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/hpc-benchmarks) for detailed instructions.

The HPL-NVIDIA benchmark uses the same input format as the standard Netlib HPL benchmark. Please see the [Netlib HPL benchmark](https://www.netlib.org/benchmark/hpl/) for getting started with the HPL software concepts and best practices.


## Downloading and using the container

The container image works well with Signularity, Docker, or Pyxis/Enroot.  Instructions for running with Singularity are provided below.  For a general guide on pulling and running containers, see [Running A Container](https://docs.nvidia.com/deeplearning/frameworks/user-guide/index.html#runcont) in the NVIDIA Containers For Deep Learning Frameworks User’s Guide. For more information about using NGC, refer to the [NGC Container User Guide](https://docs.nvidia.com/ngc/ngc-catalog-user-guide/index.html).


## Running the benchmarks

The script `hpl-aarch64.sh` can be invoked on a command line or through a Slurm batch script to launch HPL-NVIDIA for NVIDIA Grace CPU.  As of HPC-Benchmarks 23.10, `hpl-aarch64.sh` accepts the following parameters:

 * Required parameters: 
   * `--dat path`: Path to `HPL.dat` input file
 * Optional parameters:
   * `--cpu-affinity <string>`: A colon-separated list of cpu index ranges
   * `--mem-affinity <string>`: A colon separated list of memory indices
   * `--ucx-affinity <string>`: A colon separated list of UCX devices
   * `--ucx-tls <string>`: UCX transport to use
   * `--exec-name <string>`: HPL executable file

Several sample input files are available in the container at `/workspace/hpl-linux-aarch64`.


### Run with Singularity

The instructions below assume Singularity 3.4.1 or later.

Save the HPC-Benchmark container as a local Singularity image file:
```bash
singularity pull --docker-login hpc-benchmarks:23.10.sif docker://nvcr.io/nvidia/hpc-benchmarks:23.10
```

If prompted for a Docker username or password, just press "enter" to continue with guest access:
```bash
Enter Docker Username: # press "enter" key to skip
Enter Docker Password: # press "enter" key to skip
```
This command saves the container in the current directory as `hpc-benchmarks:23.10.sif`.  

Use one of the following commands to run HPL-NVIDIA with a sample input file on one NVIDIA Grace CPU Superchip.

 * To run from a local command line, i.e. _not_ using Slurm:

     ```bash
     singularity run ./hpc-benchmarks:23.10.sif \
          mpirun -np 2 --bind-to none \
          ./hpl-aarch64.sh --dat ./hpl-linux-aarch64/sample-dat/HPL_2mpi.dat \
          --cpu-affinity 0-71:72-143 --mem-affinity 0:1
     ```

 * To run via Slurm:

     ```bash
     srun -N 1 --ntasks-per-node=2 singularity run ./hpc-benchmarks:23.10.sif \
          ./hpl-aarch64.sh --dat ./hpl-linux-aarch64/sample-dat/HPL_2mpi.dat \
          --cpu-affinity 0-71:72-143 --mem-affinity 0:1
     ```



## Reference Results

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

The score below was taken on a Grace CPU Superchip with 480GB of CPU memory:
```
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WC00L2L2      168880   448     1     2             616.41             5.2093e+03
```