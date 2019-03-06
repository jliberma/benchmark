Profile HPL
==========

Use Linux performance counters to explore performance differences between HPL linked with MKL and OpenBLAS.

Enable Linux performance counters
==================

$ sudo yum install perf

Profile OpenBLAS xhpl
==================

Modify input file to run a single process with 16 threads.

~~~
$ head -n 12 HPL.dat 
HPLinpack benchmark input file
Innovative Computing Laboratory, University of Tennessee
HPL.out      output file name (if any)
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
32768 116480 65536 Ns
1 2            # of NBs
164 128 64      NBs
0            PMAP process mapping (0=Row-,1=Column-major)
1            # of process grids (P x Q)
1 4 2            Ps
1 4 8            Qs
~~~

Run OpenBLAS-linked HPL while measuring floating point instructions.

~~~
$ echo $LD_LIBRARY_PATH
/root/openblas/lib

$ OMP_NUM_THREADS=16 perf stat -M GFLOPS ./xhpl
...
WR00L2R2       32768   164     1     1              79.63             2.9458e+02

               300      fp_arith_inst_retired.scalar_single #    129.3 GFLOPs                 
    24,724,472,806      fp_arith_inst_retired.scalar_double                                   
     5,370,827,566      fp_arith_inst_retired.128b_packed_double                                   
                 0      fp_arith_inst_retired.128b_packed_single                                   
 5,880,914,182,784      fp_arith_inst_retired.256b_packed_double                                   
                 0      fp_arith_inst_retired.256b_packed_single 
~~~

Now, profile the MKL-linked xhpl

~~~
$ cd ../Linux_mpich_xhpl
$ export LD_LIBRARY_PATH=/opt/intel/compilers_and_libraries/linux/lib/intel64_lin/
$ ldd xhpl

$ OMP_NUM_THREADS=16 perf stat -M GFLOPS ./xhpl
WR00L2R2       32768   164     1     1              72.57             3.2323e+02

               300      fp_arith_inst_retired.scalar_single #    133.9 GFLOPs                 
    22,551,430,008      fp_arith_inst_retired.scalar_double                                   
        49,684,383      fp_arith_inst_retired.128b_packed_double                                   
                 0      fp_arith_inst_retired.128b_packed_single                                   
 5,884,122,169,210      fp_arith_inst_retired.256b_packed_double                                   
                 0      fp_arith_inst_retired.256b_packed_single 
...
$ echo "scale=3; 1-294.58/323.23" | bc
.089
~~~


QUESTIONS:
- Why are there so many retired scalar_doubles?
- How many flops are each packed double instruction?
- Do the observed performance counter statistics explain the measured performance difference?

Repeat the same test but this time measure performance counters related to cache.

~~~
$ OMP_NUM_THREADS=16 perf stat -e \
cache-misses,L1-dcache-load-misses,dTLB-load-misses,LLC-load-misses,l2_rqsts.miss  ./xhpl
WR00L2R2       32768   164     1     1              74.11             3.1652e+02
...
   11,573,192,488      cache-misses                                                
   174,575,977,223      L1-dcache-load-misses                                       
        74,959,710      dTLB-load-misses                                            
     1,877,313,090      LLC-load-misses                                             
   145,204,636,196      l2_rqsts.miss                                               

     177.452794927 seconds time elapsed

$ cd ../Linux_mpich_OpenBLAS/
$ export LD_LIBRARY_PATH=/root/openblas/lib
$ ldd xhpl
$ OMP_NUM_THREADS=16 perf stat -e \
cache-misses,L1-dcache-load-misses,dTLB-load-misses,LLC-load-misses,l2_rqsts.miss  ./xhpl
WR00L2R2       32768   164     1     1              80.04             2.9307e+02
…
    15,071,065,097      cache-misses                                                
   167,155,884,715      L1-dcache-load-misses                                       
        54,654,167      dTLB-load-misses                                            
     2,596,358,254      LLC-load-misses                                             
   166,375,323,748      l2_rqsts.miss  
~~~

In addition to lower performing instruction miss we see fewer cache misses with MKL, especially LLC misses which are followed by costly memory accesses.
