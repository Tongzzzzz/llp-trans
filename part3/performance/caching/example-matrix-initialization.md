16.2.5 示例：矩阵初始化

为了演示优秀的内存访问模式，我们将使用一个巨大的矩阵，所有元素值都是 42。该矩阵以行存储。

列表 16-27 中的程序按行初始化该矩阵；列表 16-28 中的程序按列初始化。哪一个程序执行更快？

_**Listing 16-27**.matrix\_init\_linear.c_

```
#include <stdio.h>
#include <malloc.h>
#define DIM (16*1024)

int main( int argc, char** argv ) {
    size_t i, j;
    int* mat = (int*)malloc( DIM * DIM * sizeof( int ) );
    for( i = 0; i < DIM; ++i )
        for( j = 0; j < DIM; ++j )
            mat[i*DIM+j] = 42;
    puts("TEST DONE");
    return 0;
}
```

_**Listing 16-28**.matrix\_init\_ra.c_

```
#include <stdio.h>
#include <malloc.h>
#define DIM (16*1024)
int main( int argc, char** argv ) {
    size_t i, j;
    int* mat = (int*)malloc( DIM * DIM * sizeof( int ) );
    for( i = 0; i < DIM; ++i )
        for( j = 0; j < DIM; ++j )
            mat[j*DIM+i] = 42;
    puts("TEST DONE");
    return 0;
}
```

再次使用 linux 的 time 工具\(不是 shell 自带的\)来测试执行时间。

```
> /usr/bin/time -v ./matrix_init_ra
   Command being timed: "./matrix_init_ra"
   User time (seconds): 2.40
   System time (seconds): 1.01
   Percent of CPU this job got: 86%
   Elapsed (wall clock) time (h:mm:ss or m:ss): 0:03.94
   Average shared text size (kbytes): 0
   Average unshared data size (kbytes): 0
   Average stack size (kbytes): 0
   Average total size (kbytes): 0
   Maximum resident set size (kbytes): 889808
   Average resident set size (kbytes): 0
   Major (requiring I/O) page faults: 2655
   Minor (reclaiming a frame) page faults: 275963
   Voluntary context switches: 2694
   Involuntary context switches: 548
   Swaps: 0
   File system inputs: 132368
   File system outputs: 0
   Socket messages sent: 0
   Socket messages received: 0
   Signals delivered: 0
   Page size (bytes): 4096
   Exit status: 0
```

```
> /usr/bin/time -v ./matrix_init_linear
   Command being timed: "./matrix_init_linear"
   User time (seconds): 0.12
   System time (seconds): 0.83
   Percent of CPU this job got: 92%
   Elapsed (wall clock) time (h:mm:ss or m:ss): 0:01.04
   Average shared text size (kbytes): 0
   Average unshared data size (kbytes): 0
   Average stack size (kbytes): 0
   Average total size (kbytes): 0
   Maximum resident set size (kbytes): 900280
   Average resident set size (kbytes): 0
   Major (requiring I/O) page faults: 4
    Minor (reclaiming a frame) page faults: 262222
   Voluntary context switches: 29
   Involuntary context switches: 449
   Swaps: 0
   File system inputs: 176
   File system outputs: 0
   Socket messages sent: 0
   Socket messages received: 0
   Signals  delivered: 0
   Page size (bytes): 4096
   Exit status: 0
```

因为 cache misses 的关系执行慢了很多，这样的输出可以使用 valgrind 加上 cachgrind 组件来进行观察。valgrind 输出见列表 16-29 所示。

_**Listing 16-29**.cachegrind\_matrix\_bad_

```
> valgrind --tool=cachegrind ./matrix_init_ra
==17022== Command: ./matrix_init_ra
==17022==
--17022-- warning: L3 cache found, using its data for the LL simulation.
==17022==
==17022== I   refs:     268,623,230
==17022== I1  misses:           809
==17022== LLi misses:       804
==17022== I1  miss rate:      0.00%
==17022== Lli miss rate:      0.00%
==17022==
==17022== D   refs:      67,163,682  (40,974 rd  + 67,122,708 wr)
==17022== D1  misses:    67,111,793  ( 2,384 rd  + 67,109,409 wr)
==17022== LLd misses:    67,111,408  ( 2,034 rd  + 67,109,374 wr)
==17022== D1  miss rate:      99.9%  (   5.8%    +      100.0%  )
==17022== LLd miss rate:      99.9%  (   5.0%    +      100.0%  )
==17022==
==17022== LL refs:       67,112,602  ( 3,193 rd  + 67,109,409 wr)
==17022== LL misses:     67,112,212       ( 2,838 rd  + 67,109,374 wr)
==17022== LL miss rate:       20.0%  (   0.0%    +      100.0%  )
```

我们可以看到，线性地访问内存显著地减少了 cache misses。

```
==17023== Command: ./matrix_init_linear
==17023==
--17023-- warning: L3 cache found, using its data for the LL simulation.
==17023==
==17023== I   refs:      336,117,093
==17023== I1  misses:            813
==17023== LLi misses:            808
==17023== I1  miss rate:       0.00%
==17023== LLi miss rate:       0.00%
==17023==
==17023== D   refs:       67,163,675 (40,970 rd  + 67,122,705 wr)
==17023== D1  misses:     16,780,146 ( 2,384 rd  + 16,777,762 wr)
==17023== LLd misses:     16,779,760 ( 2,033 rd  + 16,777,727 wr)
==17023== D1  miss rate:       25.0%  (   5.8%   +       25.0%  )
==17023== LLd  miss rate:      25.0%  (   5.0%   +       25.0%  )
==17023==
==17023== LL refs:       16,780,959   ( 3,197 rd + 16,777,762 wr)
==17023== LL misses:     16,780,568   ( 2,841 rd + 16,777,727 wr)
==17023== LL miss rate:         4.2%  (   0.0%     +      25.0%   )
```

---

**■Question 334** 查看 GCC 的 man page 的 optimizations 小节。

---



