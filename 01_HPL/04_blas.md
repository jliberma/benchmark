Install a BLAS 
==============

HPL relies on the DGEMM kernel as defined by the [BLAS][1] specification.

DGEMM is a general matrix-matrix multiplication routine in double precision.

There are many open source and commercial BLAS implementations including:
- [Atlas][https://github.com/math-atlas/math-atlas]
- [AMD Math Library (LibM)][https://developer.amd.com/amd-cpu-libraries/amd-math-library-libm/]
- [OpenBLAS][https://www.openblas.net/]
- [Intel Math Kernel Library][https://software.intel.com/en-us/mkl]

We will install Intel MKL.

Intel MKL
========

First, register and [download the MKL Library][2] for Linux.

Next, extract and install MKL:

~~~
$ tar zxvf l_mkl_2019.2.187.tgz
$ cd l_mkl_2019.2.187/
$ sed -i 's/=decline/=accept/' silent.cfg
$ sh install.sh --silent silent.cfg
$ sh install.sh --list-components
--------------------------------------------------------------------------------
The install directory path was changed to
/opt/intel
because at least one software product component was detected as having already
been installed on the system.
intel-mkl-core-c-32bit__x86_64, version: 2019.2
intel-mkl-core-c__x86_64, version: 2019.2
intel-mkl-cluster-c__noarch, version: 2019.2
intel-mkl-tbb-32bit__x86_64, version: 2019.2
intel-mkl-tbb__x86_64, version: 2019.2
intel-mkl-pgi-c__x86_64, version: 2019.2
intel-mkl-gnu-c-32bit__x86_64, version: 2019.2
intel-mkl-gnu-c__x86_64, version: 2019.2
intel-mkl-core-f-32bit__x86_64, version: 2019.2
intel-mkl-core-f__x86_64, version: 2019.2
intel-mkl-cluster-f__noarch, version: 2019.2
intel-mkl-gnu-f__x86_64, version: 2019.2
intel-mkl-gnu-f-32bit__x86_64, version: 2019.2
intel-mkl-f95-32bit__x86_64, version: 2019.2
intel-mkl-f__x86_64, version: 2019.2
$ ls /opt/intel
bin                      compilers_and_libraries_2019        conda_channel       lib  parallel_studio_xe_2019        samples_2019
compilers_and_libraries  compilers_and_libraries_2019.2.187  documentation_2019  mkl  parallel_studio_xe_2019.2.057  tbb
~~~

Test MKL
========

~~~
$ cd /opt/intel/mkl/examples
$ tar zxvf examples_core_c.tgz
$ cd cblas
$ make libintel64 function=cblas_dgemm compiler=gnu
$ echo $?
0
$ ls _results/gnu_lp64_parallel_intel64_lib/
$ cat _results/gnu_lp64_parallel_intel64_lib/cblas_dgemmx.res

     C B L A S _ D G E M M  EXAMPLE PROGRAM

     INPUT DATA
       M=2  N=5  K=4
       ALPHA=  0.5  BETA= -1.2
       TRANSA = CblasTrans  TRANSB = CblasTrans  
       LAYOUT = CblasRowMajor  
       ARRAY A   LDA=2
            1.500     2.220  
            6.300     9.000  
            1.000    -4.000  
            0.200     7.500  
       ARRAY B   LDB=4
            1.000     2.000     3.000     4.000  
            1.000     2.000     3.000     4.000  
            1.000     2.000     3.000     4.000  
            1.000     2.000     3.000     4.000  
            1.000     2.000     3.000     4.000  
       ARRAY C   LDC=5
            0.000     0.000     1.000     1.000     1.000  
            0.000     0.000     1.000     1.000     1.000  

     OUTPUT DATA
       ARRAY C   LDC=5
            8.950     8.950     7.750     7.750     7.750  
           19.110    19.110    17.910    17.910    17.910
$ ldd _results/gnu_lp64_parallel_intel64_lib/cblas_dgemmx.out
	linux-vdso.so.1 =>  (0x00007ffe2bfbf000)
	libiomp5.so => /opt/intel/compilers_and_libraries/linux/lib/intel64_lin/libiomp5.so (0x00007f775703d000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f7756e21000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007f7756c1d000)
	libm.so.6 => /lib64/libm.so.6 (0x00007f775691b000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f775654e000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f7756338000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f7757425000)
~~~

MKL Resources
============
1. [Intel MKL Install guide][https://software.intel.com/en-us/articles/intel-math-kernel-library-intel-mkl-2019-install-guide]
2. [MKL: Setting the environment][https://software.intel.com/en-us/articles/intel-math-kernel-library-intel-mkl-2018-getting-started]
3. [MKL Link line advisor][https://software.intel.com/en-us/articles/intel-mkl-link-line-advisor]
4. [MKL DGEMM tutorial][https://software.intel.com/en-us/mkl-tutorial-c-multiplying-matrices-using-dgemm]

[1][http://www.netlib.org/blas/index.html]
[2][https://software.intel.com/en-us/mkl/choose-download/linux]
