# NAMD

NAMD is a widely used molecular dynamics software that is used for large scale simulations of biomolecular systems[^Phillips2020]. It is developed by the Theoretical and Computational Biophysics Group in the Beckman Institute for Advanced Science and Technology at the University of Illinois at Urbana-Champaign and was used in the winning submission for the 2020 ACM Gordon Bell Special Prize for COVID-19 Research[^Casalino2021]. As part of the submission, NAMD was used to simulate a 305-million atom SARS-CoV-2 viral envelope on over four thousand nodes of the ORNL Summit supercomputer. The Charm++ framework is used to scale to thousands of GPUs and hundreds of thousands of CPU cores[^Phillips2014].  NAMD has supported Arm since 2014.

## Building the Source Code

To access the NAMD source code, submit a request at <https://www.ks.uiuc.edu/Research/namd/gitlabrequest.html>.
After the request is approved, you can access the source code at <https://gitlab.com/tcbgUIUC/namd>.

### Dependencies

The following script will install NAMD's dependencies to the `./build/` directory. Charm++ version 7.0.0 does not support targeting the Armv9 architecture, so Armv8 is used instead. 

```bash
#!/bin/bash

set -e

if [[ ! -a build ]]; then
    mkdir build
fi
cd build

#
# FFTW
#
if [[ ! -a fftw-3.3.9 ]]; then
    wget http://www.fftw.org/fftw-3.3.9.tar.gz
    tar xvfz fftw-3.3.9.tar.gz
fi

if [[ ! -a fftw ]]; then
    mkdir fftw
    cd fftw-3.3.9
    ./configure CC=gcc --prefix=$PWD/../fftw \
        --enable-float --enable-fma \
        --enable-neon \
        --enable-openmp --enable-threads | tee fftw_config.log
    make -j 8 | tee fftw_build.log
    make install
    cd ..
fi

#
# TCL
#
if [[ ! -e tcl ]]; then
    wget http://www.ks.uiuc.edu/Research/namd/libraries/tcl8.5.9-linux-arm64-threaded.tar.gz
    tar zxvf tcl8.5.9-linux-arm64-threaded.tar.gz
    mv tcl8.5.9-linux-arm64-threaded tcl
fi

#
# Charm++
#
if [[ ! -a charm ]]; then
    git clone https://github.com/UIUC-PPL/charm.git
fi

cd charm
git checkout v7.0.0
if [[ ! -a multicore-linux-arm8-gcc ]]; then
    ./build charm++ multicore-linux-arm8 gcc --with-production --enable-tracing -j 8
fi 
cd ..
```

### NAMD

The following script downloads and compiles NAMD in the `./build/` directory. We recommend that you use GCC version 12.3 or later as it can target the `neoverse-v2` architecture. If version GCC 12.3 is not available, the `sed` command below should be removed because the architecture will not be recognized. 

```bash
#!/bin/bash
set -e
if [[ ! -a build ]]; then
    mkdir build
fi
cd build

#
# NAMD
#
if [[ ! -a namd ]]; then
    git clone git@gitlab.com:tcbgUIUC/namd.git
    cd namd
    git checkout release-3-0-beta-3
    cd ..
fi

cd namd

if [[ ! -a Linux-ARM64-g++ ]]; then
    ./config Linux-ARM64-g++ \
        --charm-arch multicore-linux-arm8-gcc --charm-base $PWD/../charm \
        --with-tcl --tcl-prefix $PWD/../tcl \
        --with-fftw --with-fftw3 --fftw-prefix $PWD/../fftw
    sed -i 's/FLOATOPTS = .*/FLOATOPTS = -Ofast -mcpu=neoverse-v2/g' arch/Linux-ARM64-g++.arch
    cd Linux-ARM64-g++
    make depends
    make -j 8
    cd ..
fi
cd ..

```

## Running Benchmarks on Grace

STMV is a standard benchmark system with 1,066,628 atoms.  To download STMV, run the following command.

```bash
wget https://www.ks.uiuc.edu/Research/namd/utilities/stmv.tar.gz 
tar zxvf stmv.tar.gz
cd stmv
wget http://www.ks.uiuc.edu/Research/namd/2.13/benchmarks/stmv_nve_cuda.namd
wget https://www.ks.uiuc.edu/Research/namd/utilities/ns_per_day.py
chmod +x ns_per_day.py
```

The `stmv_nve_cuda.namd` input file is not specific to CUDA and runs an NVE simulation with a 2 femtosecond timestep and PME evaluated every 4 fs with multi-time stepping. The benchmark can be run with the following command from the `stmv` directory:

```bash
../build/namd/Linux-ARM64-g++/namd3 +p72 +pemap 0-71 stmv_nve_cuda.namd | tee output.txt
./ns_per_day.py output.txt
```

The metric of interest is ns/day (higher is better) corresponding to the number of nanoseconds of simulation time that can be computed in 24 hours. The `ns_per_day.py` script will parse the standard output of a simulation and compute the overall performance of the benchmark. 

## Reference Results

```admonish important 
These figures are provided as guidelines and should not be interpreted as performance targets.
```

The following result was collected on a Grace Hopper Superchip using 72 CPU cores and completed in 121 seconds. As measured by `hwmon``, the average Grace module power was 275 Watts, and the benchmark consumed approximately 9.31 Watt-hours of energy.

```bash
$ ./ns_per_day.py output.txt
Nanoseconds per day:    2.97202

Mean time per step:     0.0581422
Standard deviation:     0.00074301
```

## References

[^Phillips2020]: Phillips, James C., et al. "Scalable molecular dynamics on CPU and GPU architectures with NAMD." The Journal of Chemical Physics 153.4 (2020).

[^Casalino2021]: Casalino, Lorenzo, et al. "AI-driven multiscale simulations illuminate mechanisms of SARS-CoV-2 spike dynamics." The International Journal of High Performance Computing Applications 35.5 (2021): 432-451.

[^Phillips2014]: Phillips, James C., et al. "Mapping to irregular torus topologies and other techniques for petascale biomolecular simulation." SC'14: Proceedings of the International Conference for High Performance Computing, Networking, Storage and Analysis. IEEE, 2014.
