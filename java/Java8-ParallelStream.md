### Java 8 - parallelStream 并行流

Java多线程编程是越来越容易了。从最早的Thread，Runnable到Java5的ExecutorService 到Java7的ForkJoin框架。现在Java8又提供了并行流来简化这一过程。程序员提供做什么而不是怎么做的信息即可实现并发。

在parallelStream中默认线程池大小是机器的CPU核数，如果改变并行流的默认的Fork/Join池的大小？你可以通过一个JVM参数来修改公用的Fork/Join线程池的大小：
-Djava.util.concurrent.ForkJoinPool.common.parallelism=16 但这个设置是对整个进程都有效的，如果想只指定某个任务的线程数可以参考下面的例子。
* 值得注意的是默认情况下，所有的Fork/Join任务都会共用同一个线程池，线程的数量等于CPU的核数。

#### 设置线程数量
使用默认线程池数量
```java
    List<Integer> cost = Lists.newArrayList(1, 3, 7, 9, 34);
    long total = cost.parallelStream().map(x -> x += 1).reduce((y, z) -> y + z).get();
```
定制线程池数量
```java
    final ForkJoinPool pool = new ForkJoinPool(4);
    List<Integer> cost = Lists.newArrayList(1, 3, 7, 9, 34);
    pool.submit(() -> cost.parallelStream().map(x -> x += 1).reduce((y, z) -> y + z).get());
```

#### Ref links
- JDK8 lambda随笔 - 并行篇 http://www.cnblogs.com/editice/p/4895564.html
- Java中不同的并发实现的性能比较 http://it.deepinmind.com/%E5%B9%B6%E5%8F%91/2015/01/22/forkjoin-framework-vs-parallel-streams-vs-executorservice-the-ultimate-benchmark.html