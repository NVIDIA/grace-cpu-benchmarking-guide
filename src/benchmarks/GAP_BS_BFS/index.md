# GAP Benchmark Suite: BFS

The [GAP Benchmark Suite (Beamer, 2015)][1] was released with the goal of helping standardize graph processing evaluations. Graph algorithms and their applications are currently gaining renewed interest, especially with the growth of social networks and their analysis. Graph algorithms are also important for their applications in science and recognition. The GAP benchmark suite provides high performance (CPU only) reference implementations for various graph operations and provides a standard for graph processing performance evaluations. 

Even though GAP benchmark suite provides real world graphs and more than one kernel (high performance implementation of various graph operation algorithms), we will only look at using synthetic [Kronecker graphs](https://en.wikipedia.org/wiki/Kronecker_graph) and will be focusing on the [Breadth First Search (BFS)](https://en.wikipedia.org/wiki/Breadth-first_search) kernel.

## Initial Configuration

The repo here is the reference implementation for the [GAP Benchmark Suite](http://gap.cs.berkeley.edu/). It is designed to be a portable high-performance baseline that only requires a compiler with support for C++11. It uses OpenMP for parallelism, but it can be compiled without OpenMP to run serially. The details of the benchmark can be found in the [specification][1].

## Quick Start

Clone the reference code from the [github](https://github.com/sbeamer/gapbs) repo and follow the following steps to build from source code:

```bash
git clone https://github.com/sbeamer/gapbs.git 
cd gapbs
make
```

In case you want to override the default C++ compiler and build with an older version:
```bash
CXX=g++-8 make
```

Run BFS kernel on 1024 vertices for 1 iteration:
```bash
./bfs -g 10 -n 1
```
Additional command line flags can be found with `-h`.

## Running the BFS kernel

After successfully building and running a test run, you should be ready to run the bfs kernel.

**Note**: The command line options of interest here are:
* `-g <scale>`: generate Kronecker graph with 2^scale vertices
* `-k <degree>`: average degree for synthetic graph (default value: 16)
* `-n <n>`: perform n trials (default value: 16)

Typically, we select a scale such that the working dataset size for the workload lies outside the Last level cache on the test platforms.

A scale value of 26 means our graph will have ~67.11 M vertices. This size of graph should be large enough so that the working set of the workload will not completely lie within the last level cache of the Grace CPU. 

The default value for average degree is chosen i.e., 16. We have chosen 64 trials for our runs, i.e. -n value of 64.

Run bfs with the following command:
```bash
OMP_NUM_THREADS=72 OMP_PROC_BIND=close numactl -m 0 -C 0-71 ./bfs -g 26 -k 16 -n 64
```
This command will pin our application to CPU socket 0 and physical cores 0-71. 

## Output

When you run bfs using the above command on a Grace machine with atleast 72 cores, using Kronecker graph of scale 26 and degree 16 for 64 trials, we see an Average Time around 0.0395 ± 0.001 ms as shown in the figure below.

[![Example Output](sample_output.png)](sample_output.png)




[1]: <http://arxiv.org/abs/1508.03619> "GAP Benchmark Suite"