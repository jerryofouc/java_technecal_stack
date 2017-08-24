# 非并发List
## ArrayList
底层是对象数组，add元素时，数组会相应地进行动态扩容。remove元素时remove元素后面的元素会做相应的移位。由于是数组，直接index查询是O(1)复杂度。remove和insert复杂度是O(n)的复杂度。
## LinkedList
底层是双向链表，直接index复杂度是O(n)，而remove和insert的复杂度都为O(1)
## Stack
支持push，pop，后进先出的数据结构。
## Queue
先进先出的数据结构，主要用于多线程并发控制。
# 非并发Map
## HashMap
平时用的比较多数据结构，存储key，value数据结构。平均key，value查询，插入，删除的复杂度都是O(1)。底层使用一个hash表，hash表本质上也是一个数组。利用key的hashcode()和一系列计算作为哈希函数。找到hash槽之后在进行查找。由于可能存在冲突，在java8中，当同一个槽数量过多(超过8)的时候会使用红黑树来查找。
## TreeMap
插入，查找和删除都是复杂度都是O(logn)，底层使用红黑树来存储。
# 并发数据结构
相比于非并发数据结构，并发数据设计会更为复杂，分类可以分为在并发容器类和并发控制相关的类。

## 并发容器类
并发容器类是线程安全的容器。
### 并发List
#### CopyOnWriteArrayList
适用于读远远大于写的场景，因为每当修改都会把之前的数组复制一遍，修改成目标版本，最后再进行赋值操作，复制和修改过程都会加锁。而对于读是无锁的。
### Collections.synchronizedList(oldList);
这个代价比较大任何一个操作都会加锁，所以使用时要注意选择。
## 并发map
ConcurrentHashMap
在JDK6，JDK7和JDK8的实现都不一样，