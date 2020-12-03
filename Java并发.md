# Java并发（三）：线程安全的集合

目前为止，已经介绍了很多Java中线程相关的东西，这些东西是构建Java并发程序的基石，然而对于实际编程来说，我们应该远离这些底层结构；转而使用由专业人士封装好的高层结构，这样虽然会牺牲一部分性能，但是在易用性、安全性方面来看，都有很大的提升。

## 早期Java中的线程安全集合

### Vector、HashTable、StringBuffer

早期java提供的线程安全集合主要有三个：Vector、HashTable、StringBuffer。

这三个集合都是通过synchronize关键字实现，访问时锁住整个数据结构，效率较低，几乎被弃用。

### Collections的同步包装方法

Vector、HashTable在使用时由于效率原因被弃用，由一些非线程安全的集合（如ArrayList、HashMap）替代。

Collections工具类提供了一些方法可以将给定的非线程安全的集合简单地借助synchronize关键字封装成线程安全的同步集合。

```java
	synchronizedList
    synchronizedCollection
    synchronizedMap
    synchronizedNavigableMap
    synchronizedNavigableSet
    synchronizedSet
    synchronizedSortedMap
    synchronizedSortedSet
```

## Java.util.concurrent包中的并发集合

### Queue

#### 阻塞队列

阻塞队列，在Queue接口的基础上增加了put和take方法，当队列为满或者为空的时候，会陷入阻塞，而不会返回更不会抛异常（但是在阻塞过程中可能会因为中断抛出异常）。

具体的实现有三个：

1. ArrayBlockingQueue。数组实现。需要指定容量。

   还可以指定公平因子，但是会带来额外的开销。

2. LinkedBlockingQueue。链表实现。可以但不是必须指定容量。

3. PriorityBlockingQueue。优先级队列、堆实现。可以但不是必须指定容量。

   还可以指定比较器。

**注意：**在BlockingQueue中一共有三组读写方法：

1. put、take：阻塞队列新增，当队列为空或者满时阻塞。

2. offer、poll、peek：队列方法，不会抛出异常，通过返回值表明状态。

   因此，队列不允许插入null。

3. add、remove、element：队列方法，会抛出异常。

   列表的方法组为add、remove、get。

#### 变异阻塞队列 

**BlockingDeque**

阻塞的双端队列，和阻塞队列基本一样，但是分头尾两端。

**DelayQueue**

延迟队列，实现了阻塞队列的接口，但是只能存储Delayed元素，当Delayed元素延迟到期的时候才可以从队列中移除。

**TransferQueue**

适用于生产者-消费者模型，约定生产者调用transfer方法将产品放入队列之后，需要等到产品被消费之后返回。保证了生产者不会过多地生产消息，导致队列负载过大。