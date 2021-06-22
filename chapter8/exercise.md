1、使用 redis benchmark 工具, 测试 10 20 50 100 200 1k 5k 字节 value 大小，redis get set 性能。
测试结果如下：
```shell
    -bash-4.2$ redis-benchmark -t get,set -r 10000 -n 10000 -d 10 --csv
    "SET","102040.82"
    "GET","100000.00"
    -bash-4.2$ redis-benchmark -t get,set -r 10000 -n 10000 -d 20 --csv
    "SET","103092.78"
    "GET","103092.78"
    -bash-4.2$ redis-benchmark -t get,set -r 10000 -n 10000 -d 50 --csv
    "SET","100000.00"
    "GET","101010.10"
    -bash-4.2$ redis-benchmark -t get,set -r 10000 -n 10000 -d 100 --csv
    "SET","103092.78"
    "GET","102040.82"
    -bash-4.2$ redis-benchmark -t get,set -r 10000 -n 10000 -d 200 --csv
    "SET","104166.66"
    "GET","102040.82"
    -bash-4.2$ redis-benchmark -t get,set -r 10000 -n 10000 -d 1000 --csv
    "SET","106382.98"
    "GET","99009.90"
    -bash-4.2$ redis-benchmark -t get,set -r 10000 -n 10000 -d 5000 --csv
    "SET","99009.90"
    "GET","96153.84"
    -bash-4.2$ redis-benchmark -t get,set -r 10000 -n 10000 -d 50000 --csv
    "SET","26246.72"
    "GET","52083.33"
```
测试结果可见，在一定范围内，value比较相近的时候，Redis的Set和Get的性能变动不大。当key到5k字节的时候，相应操作的性能出现了较大服务的下降。
若继续增加到50k，则可见Redis相关操作的性能进一步下降，只有1k时的五分之一左右的效率。

2、写入一定量的 kv 数据, 根据数据大小 1w-50w 自己评估, 结合写入前后的 info memory 信息, 分析上述不同 value 大小下，平均每个 key 的占用内存空间。
无数据时占用13.7M内存；
写入10w的1KB的kv数据，内存占用约163.1MB,平均占用1.53KB;
写入10w的10KB的kv数据，内存占用约310.4MB,平均占用3.03KB.
