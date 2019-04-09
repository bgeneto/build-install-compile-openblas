## 0. Purpose 

This document contains step-by-step instructions to proceed with a (hopefully) successful build and installation of the openblas (optimized BLAS and LAPACK) library. 
Tested with Ubuntu 18.04 LTS.

## 1. Install prerequisite software

*Note: We assume you are running all the commands below with elevated privileges by doing something like `sudo su`.*

```
apt install build-essential g++ gfortran git -y
```

## 2. Create installation directory

```
OPENBLAS_DIR=/opt/openblas
mkdir $OPENBLAS_DIR
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
  NO_AFFINITY=1 NO_WARMUP=1 USE_OPENMP=0 USE_THREAD=0 LIBNAMESUFFIX=nonthreaded
make PREFIX=$OPENBLAS_DIR LIBNAMESUFFIX=nonthreaded install
```

#### 3.3. Build and install openblas threaded version


```
cd $HOME/OpenBLAS
make -j DYNAMIC_ARCH=0 CC=gcc FC=gfortran HOSTCC=gcc BINARY=64 INTERFACE=64 \
  NO_AFFINITY=1 NO_WARMUP=1 USE_OPENMP=0 USE_THREAD=1 NUM_THREADS=64
make PREFIX=$OPENBLAS_DIR install
```

#### 3.4. Build and install openblas openmp version

```
cd $HOME/OpenBLAS
make -j DYNAMIC_ARCH=0 CC=gcc FC=gfortran HOSTCC=gcc BINARY=64 INTERFACE=64 BUILD_RELAPACK=1 NO_AFFINITY=1 NO_WARMUP=1 \
  USE_OPENMP=1 NUM_THREADS=64 LIBNAMESUFFIX=openmp
make PREFIX=$OPENBLAS_DIR LIBNAMESUFFIX=openmp install
```

If you want some more aggressive optimizations, issue the following commands BEFORE making the library: 

```
export COMMON_OPT="-O3 -fexpensive-optimizations -ftree-vectorize -fprefetch-loop-arrays -march=native"
export CFLAGS="-O3 -fexpensive-optimizations -ftree-vectorize -fprefetch-loop-arrays -march=native"
export FCOMMON_OPT="-O3 -fexpensive-optimizations -ftree-vectorize -fprefetch-loop-arrays -march=native"
export FCFLAGS="-O3 -fexpensive-optimizations -ftree-vectorize -fprefetch-loop-arrays -march=native"
```

## 4. Compiling and linking with openblas. Using your openblas library

Required linker options: 
```-L/opt/openblas/lib -lm -lpthread -lgfortran -lopenblas```
Required compiler options with openmp: 
```-I/opt/openblas/include -pthread -fopenmp```

Compilation example (using openmp version): 

```
gfortran -I/opt/openblas/include -pthread -fopenmp -O3 -Wall example.f90 -o example -L/opt/openblas/lib -lm -lpthread -lgfortran -lopenblas
```


##
