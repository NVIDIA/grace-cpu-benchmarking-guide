# Python on NVIDIA Grace

Python is an interpreted, high-level, general-purpose programming language, with interpreters available for many operating systems and architectures, including Arm64.  Python 2.7 is EOL since January the 1st 2020, so it is definitely recommended to upgrade to a Python 3.x version.

## Installing Python packages

When `pip` (the standard package installer for Python) is used, it pulls the packages from [Python Package Index](https://pypi.org) and other indexes. If `pip` cannot find a pre-compiled package, it automatically downloads, compiles, and builds the package from source code. Normally it may take a few more minutes to install the package from source code than from pre-built, especially for large packages (e.g. pandas).

For installing common Python packages from source code, we need to install the following development tools:

On **RedHat**:
```bash
sudo yum install "@Development tools" python3-pip python3-devel blas-devel gcc-gfortran lapack-devel
python3 -m pip install --user --upgrade pip
```

On **Debian/Ubuntu**:
```bash
sudo apt update
sudo apt-get install build-essential python3-pip python3-dev libblas-dev gfortran liblapack-dev
python3 -m pip install --user --upgrade pip
```

## Scientific and Numerical Applications

Python relies on native code to achieve high performance.  For scientific and
numerical applications, NumPy and SciPy provide an interface to high performance
computing libraries such as ATLAS, BLAS, BLIS, OpenBLAS, etc.  These libraries
contain code tuned for Arm64 processors, and especially the Arm Neoverse V2 core
found in NVIDIA Grace.

It is recommended to use the latest software versions as much as possible. If the latest
version migration is not feasible, please ensure that it is at least the minimum version
recommended below.  Multiple fixes related to data precision and correctness on
Arm64 went into OpenBLAS between v0.3.9 and v0.3.17 and the below SciPy and NumPy versions
upgraded OpenBLAS from v0.3.9 to OpenBLAS v0.3.17.
 * OpenBLAS:  >= v0.3.17
 * SciPy: >= v1.7.2
 * NumPy: >= 1.21.1

The default SciPy and NumPy binary installations with `pip3 install numpy scipy`
are configured to use OpenBLAS.  The default installations of SciPy and NumPy
are easy to setup and well tested.

## Anaconda and Conda
Anaconda is a distribution of the Python and R programming languages for scientific computing, that aims to simplify package management and deployment.

Anaconda has had [support for Arm64 via AWS Graviton 2](https://www.anaconda.com/blog/anaconda-aws-graviton2) since since 2021, 
so Anaconda works very well on NVIDIA Grace. Instructions to install the full Anaconda package installer can be found at https://docs.anaconda.com/. Anaconda also offers a lightweight version called [Miniconda](https://docs.conda.io/en/latest/miniconda.html) which is a small, bootstrap version of Anaconda that includes only conda, Python, the packages they depend on, and a small number of other useful packages, including pip, zlib and a few others.
