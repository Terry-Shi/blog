## 1. 底层数据结构
JDK7 数组+链表组成, JDK8 数组+链表或红黑树。

## 2. 寻址方式
HashMap将Key的哈希值对数组长度取模，结果作为该Entry在数组中的index。在计算机中，取模的代价远高于位操作的代价，因此HashMap要求数组的长度必须为2的N次方。此时将Key的哈希值对2^N-1进行与运算 (tab.len - 1) & hash，其效果即与取模等效。
数组的大小（Capacity）必须是2的n次幂（默认16）。

## 3. 扩容：
当HashMap的size超过Capacity*loadFactor时，需要对HashMap进行扩容。负载因子 loadFactor 默认值0.75。

## * 如果知道数据数量，实例化HashMap时设置初始化容量多少合适
![初始化HashMap的容量的建议](https://img-blog.csdnimg.cn/20181130161505146.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmRlcmx1c3RMZWU=,size_16,color_FFFFFF,t_70)

## 4. Fast-fail
在使用迭代器的过程中如果HashMap被修改，那么ConcurrentModificationException将被抛出，也即Fast-fail策略。

异常原因和解决方法 https://www.cnblogs.com/dolphin0520/p/3933551.html