## 0. Purpose 

This document contains step-by-step instructions to proceed with a (hopefully) successful build and installation of the openblas (optimized BLAS and LAPACK) library. 
Tested with Ubuntu 18.04 LTS and 20.04 LTS.

## 1. Install prerequisite software

```
sudo apt install build-essential g++ gfortran git -y
```

## 2. Create installation directory

```
OPENBLAS_DIR=/opt/openblas
sudo mkdir $OPENBLAS_DIR
```

## 3. Build and install openblas library from source

#### 3.1. Download the latest openblas version 

```
cd $HOME
git clone https://github.com/xianyi/OpenBLAS
```

Now you have to decide wich version of openblas you need/want: non-threaded (sequential), threaded (with native parallelism) or threaded (with openmp parallelism/support). 

#### 3.2. Build and install openblas nonthreaded version

```
cd $HOME/OpenBLAS
make -j DYNAMIC_ARCH=0 CC=gcc FC=gfortran HOSTCC=gcc BINARY=64 INTERFACE=64 \
  NO_AFFINITY=1 NO_WARMUP=1 USE_OPENMP=0 USE_THREAD=0 USE_LOCKING=1 LIBNAMESUFFIX=nonthreaded
sudo make PREFIX=$OPENBLAS_DIR LIBNAMESUFFIX=nonthreaded install
```

#### 3.3. Build and install openblas threaded version

```
cd $HOME/OpenBLAS
export USE_THREAD=1
export NUM_THREADS=64
export DYNAMIC_ARCH=0
export NO_WARMUP=1
export BUILD_RELAPACK=0
export COMMON_OPT="-O2 -march=native"
export CFLAGS="-O2 -march=native"
export FCOMMON_OPT="-O2 -march=native"
export FCFLAGS="-O2 -march=native"
make -j DYNAMIC_ARCH=0 CC=gcc FC=gfortran HOSTCC=gcc BINARY=64 INTERFACE=64 LIBNAMESUFFIX=threaded \
sudo make PREFIX=$OPENBLAS_DIR LIBNAMESUFFIX=threaded install
```

#### 3.4. Build and install openblas openmp version

```
cd $HOME/OpenBLAS
export USE_THREAD=1
export NUM_THREADS=64
export DYNAMIC_ARCH=0
export NO_WARMUP=1
export BUILD_RELAPACK=0
export COMMON_OPT="-O2 -march=native"
export CFLAGS="-O2 -march=native"
export FCOMMON_OPT="-O2 -march=native"
export FCFLAGS="-O2 -march=native"
make -j DYNAMIC_ARCH=0 CC=gcc FC=gfortran HOSTCC=gcc BINARY=64 INTERFACE=64 \
  USE_OPENMP=1 LIBNAMESUFFIX=openmp
sudo make PREFIX=$OPENBLAS_DIR LIBNAMESUFFIX=openmp install
```

PS.: do NOT use -march=native or BUILD_RELAPACK=1 (gives too many errors in tests: make -j lapack-test)

## 4. Testing

Ideally you should test before install :-)

```
make -j lapack-test
cd ./lapack-netlib; python3 ./lapack_testing.py -r -b TESTING
```

Example of an acceptable output:

```
 -->   LAPACK TESTING SUMMARY  <--
SUMMARY                 nb test run     numerical error         other error
================        ===========     =================       ================
REAL                    1298655         3       (0.000%)        0       (0.000%)
DOUBLE PRECISION        1309007         0       (0.000%)        0       (0.000%)
COMPLEX                 768366          0       (0.000%)        0       (0.000%)
COMPLEX16               769178          0       (0.000%)        0       (0.000%)

--> ALL PRECISIONS      4145206         3       (0.000%)        0       (0.000%)
```

## 5. Installing 

`sudo make install`

Add the following env variables to your `~/.bashrc` or `~/.zshrc` file:

```
export C_INCLUDE_PATH=$C_INCLUDE_PATH:/opt/OpenBLAS/include
export CPATH=$CPATH:/opt/OpenBLAS/include
export LIBRARY_PATH=$LIBRARY_PATH:/opt/OpenBLAS/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/OpenBLAS/lib
```

## 6. Compiling and linking with openblas. Using your openblas library

Required linker options: 
```
-L/opt/openblas/lib -lm -lpthread -lgfortran -lopenblas
```

Required compiler options for openmp execution: 

```
-I/opt/openblas/include -pthread -fopenmp
```

Compilation example (using openmp version): 

```
ulimit -s unlimited
export MAX_THREADS=4
gfortran -I/opt/openblas/include -pthread -fopenmp -O3 -funroll-all-loops -fexpensive-optimizations -ftree-vectorize -fprefetch-loop-arrays -floop-parallelize-all \
-ftree-parallelize-loops=$MAX_THREADS -m64 -Wall example.f90 -o example -L/opt/openblas/lib -lm -lpthread -lgfortran -lopenblas_openmp
```

gcc compilation example (threaded version, not openmp):

```
gcc -I/opt/openblas/include -pthread -O3 -Wall example.c -o ~/bin/example -L/opt/openblas/lib -lm -lpthread -lgfortran -lopenblas
```

Environment variables are used to specify a maximum number of threads. For example,

```
export MAX_THREADS=4
export OPENBLAS_NUM_THREADS=$MAX_THREADS
export GOTO_NUM_THREADS=$MAX_THREADS
export OMP_NUM_THREADS=$MAX_THREADS
export MKL_NUM_THREADS=$MAX_THREADS
export BLIS_NUM_THREADS=$MAX_THREADS

```

##
