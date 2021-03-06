16.2.3 示例：二分查找和 prefetch

列表 16-21 是一个例子，我们来研究一下。

_**Listing 16-21**.prefetch\_binsearch.c_

```
#include <time.h>
#include <stdio.h>
#include <stdlib.h>

#define SIZE 1024*512*16

int binarySearch(int *array, size_t number_of_elements, int key) {
    size_t low = 0, high = number_of_elements-1, mid;
    while(low <= high) {
        mid = (low + high)/2;
#ifdef DO_PREFETCH
        // low path
        __builtin_prefetch (&array[(mid + 1 + high)/2], 0, 1);
        // high path
        __builtin_prefetch (&array[(low + mid - 1)/2], 0, 1);
#endif

        if(array[mid] < key)
            low = mid + 1;
        else if(array[mid] == key)
            return mid;
        else if(array[mid] > key)
            high = mid-1;
    }
    return -1;
}

int main() {
    size_t i = 0;
    int NUM_LOOKUPS = SIZE;
    int *array;
    int *lookups;

    srand(time(NULL));
    array =  malloc(SIZE*sizeof(int));

    lookups = malloc(NUM_LOOKUPS * sizeof(int));
    for (i=0;i<SIZE;i++) array[i] = i;
    for (i=0;i<NUM_LOOKUPS;i++) lookups[i] = rand() % SIZE;

    for (i=0;i<NUM_LOOKUPS;i++)
        binarySearch(array, SIZE, lookups[i]);
    free(array);
    free(lookups);
}
```

二分查找算法的访问模式导致我们难以对其要访问的内存进行预测。因为这种算法非线性，从起点跳到终点，然后再跳到中间，然后循环往复。让我们看看执行时间的差别。

列表 16-22 展示了 prefetch 关闭的结果。

_**Listing 16-22**.binsearch\_prefetch\_off_

```
> gcc -O3 prefetch.c -o prefetch_off && /usr/bin/time -v ./prefetch_off
```

```
   Command being timed: "./prefetch_off"   
   User time (seconds): 7.56
   System time (seconds): 0.02
   Percent of CPU  this job got: 100%
   Elapsed (wall clock) time (h:mm:ss or m:ss): 0:07.58
   Average shared text size (kbytes): 0
   Average unshared data size (kbytes): 0
   Average stack size (kbytes): 0
   Average total size (kbytes): 0
   Maximum resident set size (kbytes): 66432
   Average resident set size (kbytes): 0
   Major (requiring I/O) page faults: 0
   Minor (reclaiming a frame) page faults: 16444
   Voluntary context switches: 1
   Involuntary context switches: 51
   Swaps: 0
   File system inputs: 0
   File system outputs: 0
   Socket messages sent: 0
   Socket messages received: 0
   Signals delivered: 0
   Page size (bytes): 4096
   Exit status: 0
```

列表 16-23 展示了 prefetch 打开的执行结果。

_**Listing 16-23**.binsearch\_prefetch\_on_

```
> gcc -O3 prefetch.c -o prefetch_off && /usr/bin/time -v ./prefetch_off
```

```
   Command being timed: "./prefetch_on"
   User time  (seconds):  6.56
   System time (seconds): 0.01
   Percent of CPU  this job got: 100%
   Elapsed (wall clock) time (h:mm:ss or m:ss): 0:06.57
   Average shared text size (kbytes): 0
   Average unshared data size (kbytes): 0
   Average stack size (kbytes): 0
   Average total size (kbytes): 0
   Maximum resident set size (kbytes): 66512
   Average resident set size (kbytes): 0
   Major (requiring I/O) page faults: 0
   Minor (reclaiming a frame) page faults: 16443
   Voluntary context switches: 1
```

用带有 cachegrind 模块的 valgrind 工具，我们可以检查 cache miss 的次数。列表 16-24 列出了关闭 prefetch 的结果，列表 16-25 列出了 prefetch 打开的结果。

大写字母 I 表示指令缓存，D 表示数据缓存，LL 表示 Last Level Cache。数据缓存 miss 几乎是 100%，结果比较糟糕。

_**Listing 16-24**.binsearch\_prefetch\_off\_cachegrind_

```
==25479== Cachegrind, a cache and branch-prediction profiler

==25479== Copyright (C) 2002-2015, and GNU GPL'd, by Nicholas Nethercote et al.

==25479== Using Valgrind-3.11.0 and LibVEX; rerun with -h for copyright info

==25479== Command: ./prefetch\_off

==25479==

--25479-- warning: L3 cache found, using its data for the LL simulation.

==25479==

==25479== I   refs:      2,529,064,580

==25479== I1  misses:              778

==25479== LLi misses:              774

==25479== I1  miss rate:          0.00%

==25479== Lli miss rate:          0.00%

==25479==

==25479== D   refs:         404,809,999   (335,588,367 rd   + 69,221,632 wr)

==25479== D1  misses:       160,885,105   (159,835,971 rd   +  1,049,134 wr)

==25479== LLd misses:       133,467,980   (132,418,879 rd   +  1,049,101 wr)

==25479== D1  miss rate:           39.7%  (       47.6%     +        1.5%  )

==25479== LLd miss rate:           33.0%  (       39.5%     +        1.5%  )

==25479==

==25479== LL refs:          160,885,883   (159,836,749 rd   +  1,049,134 wr)

==25479== LL misses:        133,468,754   (132,419,653 rd   +  1,049,101 wr)

==25479== LL miss rate:             4.5%  (        4.6%     +        1.5%  )
```

_**Listing 16-25**.binsearch\_prefetch\_on\_cachegrind_

```
==26238== Cachegrind, a cache and branch-prediction profiler

==26238== Copyright (C) 2002-2015, and GNU GPL'd, by Nicholas Nethercote et al.

==26238== Using Valgrind-3.11.0 and LibVEX; rerun with -h for copyright info

==26238== Command: ./prefetch_on

==26238==

--26238-- warning: L3 cache found, using its data for the LL simulation.

==26238==

==26238== I   refs:     3,686,688,760

==26238== I1  misses:             777

==26238== LLi misses:             773

==26238== I1  miss rate:         0.00%

==26238== LLi miss rate:         0.00%

==26238==

==26238== D   refs:       404,810,009   (335,588,374  rd   + 69,221,635 wr)

==26238== D1  misses:     160,887,823   (159,838,690  rd   +  1,049,133 wr)

==26238== LLd misses:     133,488,742   (132,439,642  rd   +  1,049,100 wr)

==26238== D1  miss  rate:        39.7%  (       47.6%      +        1.5%  )

==26238== LLd miss rate:         33.0%  (       39.5%      +        1.5%  )

==26238==

==26238== LL refs:        160,888,600   (159,839,467  rd   +  1,049,133 wr)

==26238== LL misses:      133,489,515   (132,440,415  rd   +  1,049,100 wr)

==26238== LL miss rate:           3.3%  (        3.3%      +        1.5%  )
```

可以看到，缓存 miss 极大地降低了。

