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
### ConcurrentHashMap
- 对于get，put，iterator等方法都遵循和HashMap的语义。不过他们是线程安全的。
- 在JDK6，JDK7和JDK8的实现都不一样。
- 主要看JDK8，在JDK8中，get方法和Iterator都是无锁的，满足happen-before关系。
## BlockingMq
- 这个是我们在并发控制使用比较多的数据结构，常用的场景是使用生产者消费者模式场景，比如一部任务的执行，异步写日志等等。
### LinkedBlockingQueue
- 使用putLock和takeLock两把锁。使用双锁队列算法。减少在put或者take时需要同时获得两把锁的概率，使用cascade 通知算法。只有在临界点需要获得两把锁。同时利用一个count变量来维护整个队列的数量。

## 并发类
java提供两方面的支持，一个是原语的支持，syncronized, object.wait() object.notify。另外是基于AQS为基础建立的一系列类库。例如各种Lock，各种原子类例如AtomicInteger，还有各种并发同步控制类。

## 并发原语
### syncronized
- 互斥关键字。同时只允许一个线程进入，其余的线程只能等待。
- 加在方法上使用this object作为guard
- 也可以直接进行指定。

### Object.wait()
当当前线程拥有monitor时，当不满足某种条件时，把自己block住，主动放弃该monitor。等待另外的线程使用obj.sigalALl或obj.signal来唤醒，唤醒之后重新获得这把锁。
所以使用wait一定要这么使用
``` 
 syncronized(obj){
       while(condition not meet){
          obj.wait()
       }
 }
```

### Object.notify()
当当前线程拥有monitor，发现其他线程可能因为条件不满足主动wait，而由于当前线程的一些操作导致条件已经满足，就会主动唤醒block其他的线程。
```$xslt
syncronized(obj){
    obj.notify()
}
```

## AQS
- AQS 是JDK6引入的，性能比syncronized要好，功能比syncronized要多，所以如果有新的功能能不用syncronized就不用吧，新的JDK很多类都是基于AQS做的。
而AQS的基础是CAS和Unsafe park。
- AQS是一个并发类库的框架，如果依赖int 类型作为state的lock，或者信号量可以使用这个类库作为开发框架。

## ReentrantLock
- 可重入锁。
- 比原生syncronized支持功能更多（支持超时，支持公平调度，支持多个condition）。
- 如果要使用的话，请尽量使用这个类库。

## CountDownLatch
-  是一个同步器，适用于以下场景，分为两种线程组，线程组A调用await，线程组B控制线程组A是否能执行下去，使用countDown，
当countDown状态变为0时，线程组所有的继续执行。
- 底层直接使用AQS。
- 如何使用
```$xslt
CountDownLatch countDownLatch = CountDownLatch(N)//N 代表需要countDown多少次
countDownLatch.await()//等待block住
countDownLatch.countDown()//N = N-1，如果到0，await的线程全部释放。
```

## CyclicBarrier
- 循环使用Syncronizer
- 线程组到达一个共同点出发一个动作执行。
-如何使用
```$xslt
 CyclicBarrier cyclicBarrier  = new CyclicBarrier(10, new Runnable() {
            public void run() {
                    xx
            }
        });
  cyclicBarrier.await();
```

## Future
- 这个对象代表异步任务执行的结果，拿到这个对象，可以得到进程执行的进度也可以通过get方法，把自己block住等待执行结果把自己唤醒。
- 具体实现类是FutureTask类。

## Semaphore
- 信号量，代表最多允许多少线程可以进入。
- 底层使用AQS来实现。
- 比如数据库连接池可以使用信号量类来进行实现。
- 如何使用
```$xslt
Semaphore semaphore = new Semaphore(N,true);
semaphore.acquire()//得到，如果没有足够的就会block住。
semaphore.release()//释放。
```

## ThreadPoolExecutor
- JDK threadpool，一般情况下，我们不需要直接使用，需要使用Executors来工厂模式来生成threadpool。
- 几个参数，coreSize,maximumPoolSize,workQueue比较重要。
    - coreSize，worker<coreSize时，新执行的任务会newThread，而不是放在队列中,当worker数>coreSize
    时，会首先尝试往队列放，当队列放失败时则重新newThread，但是这些thread会在线程数不够的时候回收。
    - maximumPoolSize,最大线程数，当超过这些数目时会reject掉。
    - workQueue
        - 当使用SyncronizedQueue时则，直接使用handoff模式，不会往队列放。
        - 当使用LinkedBlockingQueue，则是unboundedQueue模式。
        - 当使用ArrayBlockingQueue，则是boundedQueue模式。



    






