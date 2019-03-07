Calculate Theoretical Performance
--------------------------------

HPL performance is measured in Floating Point Operations per Second. (FLOPs)

FLOPs are typically measured in GigaFLOP units in double precision.

Reference System
- 2 x Dell FC430
- [Intel Xeon E5-2620 v4 Processor][1]
- 2.10 GHz
- 8 core
- Dual socket
- 128 GB RAM

FLOPs per cycle:
- AVX instruction set (256b packed register)
- 2 4-wide FMA units (2 FLOPs per FMA)
- 2 FMA units x 2 FLOPs/cycle x 4 DP floats per register == 16-DP FLOP/s cycle

Theoretical GFLOPs:
- # Systems x # Cores x # Sockets x Clock Frequency in GHz x FLOPs/cycle 
- 1 x 8 x 2 x 2.1 x 16 = 537.6 theoretical @ 2.1 GHz
- 8 x 2 x 3.0 x 16 = 793.6 theoretical system @ 3.0 GHz with P-states enabled


[1]https://ark.intel.com/content/www/us/en/ark/products/92986/intel-xeon-processor-e5-2620-v4-20m-cache-2-10-ghz.html
