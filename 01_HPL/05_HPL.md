Run HPL
======

Install HPL
==========
http://www.netlib.org/benchmark/hpl/

~~~
$ cd $HOME
$ wget http://www.netlib.org/benchmark/hpl/hpl-2.3.tar.gz
$ tar zxvf hpl-2.3.tar.gz
$ cd ~/hpl-2.3
~~~

Build HPL
========

Prepare the build environment.

~~~
$ cd ~/hpl-2.3/setup/
$ sh make_generic
$ cd ..
$ grep -v \# setup/Make.UNKNOWN > Make.Linux_mpich_MKL
~~~

Set build variables for the make file.

Notice:
1. ARCH is set to Linux_mpich_MKL
2. MP* point to the MPICH libraries
3. LA* point to the MKL libraries. LALIB includes link parameters.
4. The C compiler is mpicc to link with the MPI libraries.
5. CCFLAGS includes compiler optimzations for the Intel Boradwell CPU including avx2 instruction support.
6. Include -lrt in the link flags for Fortran 77 compatability on some systems
7. TOPdir is set to $HOME/hpl-2.3, where the benchmark source was extracted

~~~
$ cat Make.Linux_mpich_MKL
SHELL        = /bin/sh
CD           = cd
CP           = cp
LN_S         = ln -s
MKDIR        = mkdir
RM           = /bin/rm -f
TOUCH        = touch
ARCH         = Linux_mpich_MKL
TOPdir       = $(HOME)/hpl-2.3
INCdir       = $(TOPdir)/include
BINdir       = $(TOPdir)/bin/$(ARCH)
LIBdir       = $(TOPdir)/lib/$(ARCH)
HPLlib       = $(LIBdir)/libhpl.a
MPdir        = $(HOME)/mpich3-install
MPinc        = -I $(MPdir)/include
MPlib        = $(MPdir)/lib/libmpi.a
LAdir        = /opt/intel/mkl
LAinc        = $(LAdir)/include
LAlib        = -Wl,--start-group $(LAdir)/lib/intel64/libmkl_gf_lp64.a \
$(LAdir)/lib/intel64/libmkl_gnu_thread.a \
$(LAdir)/lib/intel64/libmkl_core.a -Wl,--end-group \
-lgomp -lpthread -lm -ldl
F2CDEFS      = -DAdd_ -DF77_INTEGER=int -DStringSunStyle
HPL_INCLUDES = -I$(INCdir) -I$(INCdir)/$(ARCH) $(LAinc) $(MPinc)
HPL_LIBS     = $(HPLlib) $(LAlib) $(MPlib)
HPL_OPTS     =
HPL_DEFS     = $(F2CDEFS) $(HPL_OPTS) $(HPL_INCLUDES)
CC           = mpicc
CCNOOPT      = $(HPL_DEFS)
CCFLAGS      = $(HPL_DEFS) -m64 -march=core-avx2 -lm \
-fomit-frame-pointer -O3 -funroll-loops 
LINKER       = mpif77
LINKFLAGS    = -lrt
ARCHIVER     = ar
ARFLAGS      = r
RANLIB       = echo
~~~

Build the binary executable.

~~~
$ make arch=Linux_mpich_MKL
$ ls bin/Linux_mpich_MKL
$ ldd bin/Linux_mpich_MKL/xhpl 
	linux-vdso.so.1 =>  (0x00007fff76b39000)
	librt.so.1 => /lib64/librt.so.1 (0x00007f5962576000)
	libgomp.so.1 => /lib64/libgomp.so.1 (0x00007f5962350000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f5962134000)
	libgfortran.so.3 => /lib64/libgfortran.so.3 (0x00007f5961e12000)
	libm.so.6 => /lib64/libm.so.6 (0x00007f5961b10000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007f596190c000)
	libmpifort.so.12 => /root/mpich3-install/lib/libmpifort.so.12 (0x00007f59616d5000)
	libmpi.so.12 => /root/mpich3-install/lib/libmpi.so.12 (0x00007f5961194000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f5960f7e000)
	libquadmath.so.0 => /lib64/libquadmath.so.0 (0x00007f5960d42000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f5960975000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f596277e000)
~~~


Configure the Input file

HPL uses an input file called HPL.dat. The input file allows you to specify parameters that control the data distribution.

A few of the important parameters are discussed below:

*Calculate N*

N determines the problem size. It is the length of the common edges of the matrices.

Each matrix element is a double precision float, which is 64b on Intel architecture.

To determine an optimal N size:

1. Calculate memory size in bytes: Memory in GB * 1024 * 1024 * 1024 * node count
2. Divide the result by 8 for double precision: (Memory in GB * 1024 * 1024 * 1024 * node count)/8
3. Take the square root of DP count to find the matrix edge size: sqrt((Memory in GB * 1024 * 1024 * 1024 * node count)/8)
4. Multiply by .9 to leave room for operating system functions: sqrt((Memory in GB * 1024 * 1024 * 1024 * node count)/8) * .9

Optional: find nearest value of N evenly divisible by NB for optimal load balancing

~~~
$ echo "sqrt(($(free -g | awk ' /Mem/ { print $2 }')*1024*1024*1024)/8)*.9" | bc | cut -f1 -d.
~~~

*Process Grid*

The process grid controls how the matrices are decomposed across cores.

HPL uses a block-cyclic distribution scheme to load balance the matrix across the nodes.

Each core will receive approximately N^2/(P*Q) matrix elements.

A few heuristics for determining the process grid:
1. P * Q should equal the physical core count of all systems.
2. A square process grid is more efficient for low memory latency systems
3. A flat process grid is more efficient for systems with high memory latency
4. When P != Q, Q should be larger than P

In our case, we have two 8-core Xeon processors. So, to run HPL on a single system, or P * Q should == 16.
~~~
$ grep -c Xeon /proc/cpuinfo 
16
~~~

*Block distribution*

NB controls the cyclic-block distribution size for each cycle of computation.

1. Optimal block size is dictated by cache characteristics and the speed/latency of the core interconnects.
2. It should be determined experimentally
3. An efficient block distribution is large enough to promote cache re-use and small enough for load balancing.
4. Generally a value between 64 and 256 will give best performance.


Complete HPL.dat input file for a single server HPL run:
~~~
$ cd bin/Linux_mpich_MKL
$ cat HPL.dat
HPLinpack benchmark input file
Innovative Computing Laboratory, University of Tennessee
HPL.out      output file name (if any)
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
32768 29 30  Ns
1            # of NBs
164          NBs
0            PMAP process mapping (0=Row-,1=Column-major)
1            # of process grids (P x Q)
2 4 1        Ps
8 4 16       Qs
16.0         threshold
1            # of panel fact
2 1 2        PFACTs (0=left, 1=Crout, 2=Right)
1            # of recursive stopping criterium
2 4          NBMINs (>= 1)
1            # of panels in recursion
2            NDIVs
1            # of recursive panel fact.
0 1 2        RFACTs (0=left, 1=Crout, 2=Right)
1            # of broadcast
0            BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)
1            # of lookahead depth
0            DEPTHs (>=0)
2            SWAP (0=bin-exch,1=long,2=mix)
64           swapping threshold
0            L1 in (0=transposed,1=no-transposed) form
0            U  in (0=transposed,1=no-transposed) form
1            Equilibration (0=no,1=yes)
8            memory alignment in double (> 0)
~~~

Run HPL.

~~~
mpiexec -hosts 192.168.122.33 -np 16 ./xhpl
…
WR00L2R2      116480   164     4     4            2049.06             5.1418e+02
~~~

Calculate efficiency
===================

Efficiency is a number between 0 and 1 that shows how close the measured performance is to the theoretical performance.

If we use the theoretical performance calculated at 2.1 GHz, our efficiency is as follows:

~~~
$ echo "scale=3; 514.18 / 537.6" | bc -l
.956
~~~

However, with P-states enabled, it is likely that the cores were running at clock rates faster to the 3.0 GHz maximum frequency.

Efficiency accounting for P-states at 3.0 GHz:
~~~
$ echo "scale=3; 514.18 / 793.6" | bc -l
.647
~~~

QUESTION: what is the relationship between P-states, performance, and efficiency?
Higher FLOPs but lower efficiency. Power consumption increases.

Disable hyperthreading and P-states
====

Repeat the same test with hyperthreading and P-states disabled in the BIOS.

~~~
$ echo 1 > /sys/devices/system/cpu/intel_pstate/no_turbo
$ cat /sys/devices/system/cpu/intel_pstate/no_turbo

mpiexec -hosts 192.168.122.33 -np 16 ./xhpl
…
WR00L2R2      116480   164     4     4            2212.81             4.7613e+02

$ echo "scale=3; 476.13 / 537.6" | bc -l
.885
~~~

The efficiency is in the high 80s, which is good for a single node run.

Hybrid mode: MPI + OpenMP
======

HPL supports OpenMP constructs. These can be used to run HPL on a shared memory system either with out without MPI.

In this example we will start one xhpl process on each socket in a single system, but spawn 8 thread per process.

Change the process grid (P/Q) to a multiple of 2 in HPL.dat.

~~~
$ cat HPL.dat
…
32768	  N
1	  Ps
2	  Qs
~~~

Open a separate terminal and run top. Press “1” for per core statistics and “H” to show threads.

~~~
$ top

~~~

Run xhpl with 2 processes and 8 threads per process. After an initial ramp up the DGEMM calculation will invoke 8 threads per process.

~~~
$ export OMP_NUM_THREADS=8; mpiexec -hosts 192.168.122.33 -np 2 ./xhpl
~~~

Rerun single node HPL with the original N plus OpenMP threads and compare the efficiency to the process-only run.

~~~
$ export OMP_NUM_THREADS=8; mpiexec -hosts 192.168.122.33 -np 2 ./xhpl
...
WR00L2R2      116480   164     1     2            2234.60             4.7149e+02

$ echo "scale=3; 471.49 / 537.6" | bc -l
.877
~~~

Run HPL across multiple nodes
====

First, adjust the HPL.dat input file.

- Calculate N based on combined available memory in all nodes.
~~~
$ echo "sqrt((2*$(free -g | awk ' /Mem/ { print $2 }')*1024*1024*1024)/8)*.9" | bc | cut -f1 -d.
164860
$ echo "164860/192" | bc
858
$ echo "860*192" | bc
165120
~~~
- Increase the panel factorization to a multiple of the total core count. (with hyperthreading disabled on both nodes.)
- Disable CPU frequency scaling on both nodes.
~~~
$ ssh 192.168.122.34 "echo 1 > /sys/devices/system/cpu/intel_pstate/no_turbo"
~~~

The multi-node HPL.dat:

~~~
$ cat HPL.dat 
HPLinpack benchmark input file
Innovative Computing Laboratory, University of Tennessee
HPL.out      output file name (if any)
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
165120 32768 116480 65536 Ns
1            # of NBs
128 192          NBs
0            PMAP process mapping (0=Row-,1=Column-major)
1            # of process grids (P x Q)
2          Ps
16          Qs
16.0         threshold
1            # of panel fact
2 1 2        PFACTs (0=left, 1=Crout, 2=Right)
1            # of recursive stopping criterium
2 4          NBMINs (>= 1)
1            # of panels in recursion
2            NDIVs
1            # of recursive panel fact.
1 0 2        RFACTs (0=left, 1=Crout, 2=Right)
1            # of broadcast
0            BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)
1            # of lookahead depth
0            DEPTHs (>=0)
2            SWAP (0=bin-exch,1=long,2=mix)
64           swapping threshold
0            L1 in (0=transposed,1=no-transposed) form
0            U  in (0=transposed,1=no-transposed) form
1            Equilibration (0=no,1=yes)
8            memory alignment in double (> 0)
~~~

Configure the machinefile with one entry per core:

~~~
cat ~/machinefile
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.33
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
192.168.122.34
~~~

Run HPL:
~~~
$ mpiexec -f ~/machinefile ./xhpl
...
WR00C2R2      165120   128     2    16            3604.66             8.3262e+02
~~~

Calculate efficiency:

~~~
$ echo "scale=3; 832.62/(537.6*2)" | bc
.774
~~~

There is a fairly substantial drop off from 1 to 2 machines.

A low latency network between the nodes such as Infiniband would improve the efficiency by minimizing the communication overhead. 
