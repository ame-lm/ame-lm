# Java并发（二）：线程同步

## 线程同步

### 竞争

多个 线程同时对一片内存进行读写，如果不加以限制，可能会发生一些意想不到的情况。这种情况叫做竞争。例如：

``` java
package com.ame;

/*
 * 线程同步
 * */
public class Main2 {
    public static void main(String[] args) {
        Counter c = new Counter();
        Runnable r = () -> {
            for (int i = 0; i < 10000; i++) {
                c.add();
            }
        };
        for (int i = 0; i < 10; i++) {
            Thread t = new Thread(r);
            t.start();
        }

        while (Thread.activeCount() > 2) {
            Thread.yield();
        }
        System.out.println(c.get());
    }

}

class Counter {
    private int val = 0;

    public void add() {
        this.val++;
    }

    public int get() {
        return this.val;
    }
}
```

预期结果：100000

实际结果：<100000

**原因**：val++分为三步：

1. tmp=val
2. tmp=tmp+1
3. val=tmp

并发时，有可能从中间的任何一步被打断，假如val=0，线程1和线程2同时执行，线程1执行到第2步被打断，然后执行线程2，线程2写会val=1，然后继续执行线程1，线程1写入val=1，这就与预期的val=2不合。

在Java中，我们可以通过同步来实现对线程竞争的控制。下面介绍实现同步的一些具体方法：

### 锁（可重入锁）

竞争情况下，出现意料之外的结果，主要是因为val++这个我们以为是一步到位的操作被分成了三步，并被打断。如果我们采取措施保证这三步不被打断，就可以解决这个问题。

为了保证一段代码（临界区）只能同时被最多一个线程运行，Java提供了锁机制，在进入临界区之前，需要获得对应的锁，在退出临界区时应该将锁释放，供其他线程使用 。

将Counter修改为：

``` java
class Counter {
    private int val = 0;
    private Lock lock = new ReentrantLock();

    public void add() {
        lock.lock();
        this.val++;
        lock.unlock();
    }

    public int get() {
        lock.lock();
        return this.val;
        lock.unlock();
    }
}
```

再次运行main方法，可以看到结果为100000，符合预期。

由于现场可能被意外终止（异常），而锁必须被释放，因此使用锁时常常需要用try包裹临界区，并在finally块中释放锁：

``` java
class Counter {
    private int val = 0;
    private Lock lock = new ReentrantLock();

    public void add() {
        lock.lock();
        try {
            this.val++;
        } finally {
            lock.unlock();
        }
    }

    //如果不加锁，调用get方法可能从add方法中间截取到val的值。
    public int get() {
        lock.lock();
        try {
	        return this.val;
        } finally {
            lock.unlock();
        }
    }
}
```

### 条件对象

当某一线程获得了锁，进入了临界区，但是需要满足某一条件才能执行，这时候就需要引入条件对象的概念了。

在一个锁对象中，可以绑定多个条件，在获得了锁之后，如果条件不满足，可以陷入等待状态直到条件满足才继续执行，在等待期间将锁释放。

为了演示条件对象，这里我们规定：get方法只能返回5的倍数值。并修改main方法。如下：

``` java
package com.ame;

import java.util.Random;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/*
 * 线程同步
 * */
public class Main2 {
    public static void main(String[] args) {
        Counter c = new Counter();
        Random random = new Random();
        Runnable r = () -> {
            for (int i = 0; i < 10000; i++) {
                c.add();
                try {
                    Thread.sleep(random.nextInt() % 50 + 50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        for (int i = 0; i < 2; i++) {
            Thread t = new Thread(r);
            t.start();
        }

        new Thread(() -> {
            while (true) {
                try {
                    System.out.println(c.get());
                    Thread.sleep(random.nextInt() % 50 + 50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

}

class Counter {
    private int val = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void add() {
        lock.lock();
        try {
            this.val++;
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    //如果不加锁，调用get方法可能从add方法中间截取到val的值。
    public int get() throws InterruptedException {
        lock.lock();
        try {
            while (val % 5 != 0) {
                condition.await();
            }
            return this.val;
        } finally {
            lock.unlock();
        }
    }
}
```

结果：

``` shell
5
10
15
15
20
25
30
30
35
40
40
...
```

1. 通过lock.newCondition获得条件对象。
2. 通过condition.await陷入等待条件的状态，释放锁。
3. 通过condition.signal和condition.signalAll唤醒等待该条件的线程。
4. 被唤醒只是条件发生了改变，可能满足，因此最好使用while(..){conditon.await()}这样的结构判断。

### synchronized

Lock和Condition提供了高度灵活的同步机制，但是大多数情况下并不需要这么灵活（难以控制）的同步机制，因此Java中提供了Synchronized关键字。

Java中每一个对象都有一个锁（对象锁）和一个条件对象，如果使用synchronize保护一段代码（方法或者代码段），那么在这段代码被调用之前必须先获得对应的对象锁（）。

1. 如果synchronize修饰实例方法，对应的锁是对象的内部锁。

2. 如果synchronize修饰静态方法，对应的锁是Class对象的锁。

3. 如果synchronize修饰代码段：

   ``` java
   synchronized(obj){
       ...
   }
   ```

   对应的是obj的对象锁。

synchronize是一个简化版本的锁机制。

对Object中提供了wait、notify、notifyAll替代await、singal、singalAll方法（否则Condition的这些方法会和Object冲突）。

上述例子使用Synchronize如下：只需修改Counter类即可。

``` java
class Counter {
    private int val = 0;

    public synchronized void add() {
        this.val++;
        notifyAll();
    }

    //如果不加锁，调用get方法可能从add方法中间截取到val的值。
    public synchronized int get() throws InterruptedException {
        while (val % 5 != 0) {
            wait();
        }
        return this.val;
    }
}
```



1. 比直接使用Lock和Condition来得更加简洁。
2. 如果对静态方法使用synchronized，那么锁住的是Class对象的锁，而不是实例对象。

使用前面讲到的锁已经几乎能够解决所有的同步问题（尽管可能很麻烦、效率不高），但是为了更加高效、简洁，Java还提供了一些其他的特性。

### volatile域

多处理机操作系统可能会为变量设置缓冲，当一个处理机更新了变量的值，如果没有特殊的机制保障，很可能别的处理机并不会立马更新，而是过一段时间才会知道更新。这很显然不是我们想要的，可以通过加锁来解决，但是仅仅为了一个变量而加锁，开销会比较大，在Java中，锁的开销并不小。

针对这种情况，我们可以使用volatile关键字。

1. volatile提供可见性（更新立刻可见、不会被缓存干扰）
2. volatile不保证原子性（不保证线程安全）
3. 适用于一写多读，或者是多写多读但是写不依赖于读。

### 原子类

很多时候，我们只是需要一个线程安全的、简单的递增或者递减等方案，如果采用锁机制解决，那么有些大材小用，会带来很多性能上的浪费，为此Java提供了原子类（Atomic）来解决这个问题。

Java中的原子类位于java.util.concurrent.atomic包中，这个包中的原子类并没有使用锁机制来保证操作的原子性，而是使用了CAS机制来完成。

原子类可分为：原子更新基本类型、原子更新数组类型、原子更新引用类型、原子更新域类型、高效原子类。

#### 原子更新基本类型（包括引用类型）

1. 包括AtomicInteger、AtomicLong、AtomicBoolean、AtomicReference类型。
2. 提供了一些常见的原子更新的操作：
   1. addAndGet、updateAndGet、accumulateAndGet等类似于++i的操作。
   2. getAndAdd、getAndUpdate、getAndAccumulate等类似于i++的操作。
   3. 原理是借助于compareAndSet实现。
   4. 当更新值不需要立刻同步到各个线程时，可以使用lazySet，该操作不会立刻刷新，但最终一定会刷新。

##### compareAndSet

简称CAS，提供两个参数，一个是期望值，一个是更新值。如果当前值等于期望值那么更新，返回true，否则不更新，返回false。通过循环使用，直到返回ture，可以实现线程同步，在一般情况下比锁更加高效，但是在大量线程同时并发执行这一操作时可能一直无法返回true，占用大量cpu。

#### 原子更新数组类型（包括引用数组类型）

1. 包括了AtomicIntegerArray、AtomicLongArray、AtomicReference<>类型。
2. 各种操作类似基本类型，但是更新时需要指定下标索引。

#### 原子更新域类型

包括AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater。原子更新基本类型封装了一个基本类型的变量，为他提供了线程安全的操作，同理，原子更新域类型为对象域封装了一个线程安全的updater，通过这个updater去更新域是线程安全的。

原子更新域类型对于所将要封装的域，有以下要求：

1. 必须是可见的。原子类型通过反射获取访问权限时，会进行权限检查，如果当前目标域对于当前对象不可见，会抛出异常。主要是为了安全。
2. 必须是volatile。保证可见性，因为内部采用cas机制进行同步，如果不用volatile修饰，可能会出现同步问题。
3. 只能是实例变量，不能是类变量（static）。
4. 不能被final修饰。
5. 值得注意的是：AtomicIntegerFieldUpdater只能用来封装int，而Integer要用AtomicReferenceFieldUpdater封装。

使用例程：

``` java
package com.ame;

import java.util.concurrent.atomic.*;

public class Main3 {
    publicd static void main(String[] args) {
        AtomicIntegerFieldUpdater a = AtomicIntegerFieldUpdater.newUpdater(Data.class, "a");

        Runnable r = () -> {
            Data data = Data.getInstance();
            for (int i = 0; i < 10000; i++) {
                a.addAndGet(data, 1);
                data.b++;
            }
        };
        try {
            Thread t1 = new Thread(r);
            t1.start();
            Thread t2 = new Thread(r);
            t2.start();
            Thread t3 = new Thread(r);
            t3.start();
            Thread t4 = new Thread(r);
            t4.start();
            t1.join();
            t2.join();
            t3.join();
            t4.join();
        } catch (Exception e) {
            e.printStackTrace();
        }

        System.out.println(Data.getInstance().getA());
        System.out.println(Data.getInstance().b);
    }

}

class Data {
    protected volatile int a = 0;
    public int b = 0;
    private static Data data = new Data();

    private Data() {
        ;
    }

    public static Data getInstance() {
        if (data == null) {
            data = new Data();
        }
        return data;
    }

    public int getA() {
        return a;
    }
}
```

运行结果：

``` powershell
40000
26446
```

#### 高效原子类

高效原子类包括LongAdder、DoubleAdder、LongAccumulater、DoubleAccumulater。AtomicLong等虽然可以借助CAS操作实现原子自加等操作，但是当多个线程并发访问时由于CAS的乐观锁机制，会导致重试次数过多导致性能下降，高效原子类可以避免这个问题。

LongAdder提供add方法和sum方法。采取热点分离技术。

1. add方法尝试对base值进行cas更新，若失败，说明并发量较大，然后在自己的cell里更新，减少冲突。
2. sum方法直接对sum和cells求和。
3. 需要注意的是：相对于AtomicLong而言，LongAdder并发写效率更高，但是读的效率略低（因为需要求和），而且没法执行类似getAndUpdate的操作，必须分开执行。

### 死锁

略

#### 线程局部变量

在线程间共享变量会带来风险，需要各种同步机制，也会带来更大的开销，如非必要，应该尽量避免在线程之间共享变量，甚至有时候不能共享变量。

这时候可以使用ThreadLocal类，ThreadLocal类为每个使用它线程提供了一个对应的变量副本，保证各个副本之间互不干扰，这样可以有效避免因不必要的并发读写带来的各种问题。

获取一个ThreadLocal对象的方法有两种：

1. new。通过这种方式获取的ThreadLocal需要自己手动调用set方法去设置初值。
2. ThreadLocal.withInitial()。通过传入Supplier\<T\>，可以不用set去初始化，ThreadLocal会自动调用Supplier去初始化。

使用如下：

``` java
package com.ame;

public class Main4 {
    public static void main(String[] args) {
        Int a = new Int();
        ThreadLocal<Int> threadLocal = ThreadLocal.withInitial(Int::new);
        Runnable r = () -> {
            a.addVal();
            Int b = threadLocal.get();
            b.addVal();
            System.out.println(Thread.currentThread() + ":" + "a=" + a.getVal() + ",b=" + b.getVal());
        };
        for (int i = 0; i < 3; i++) {
            new Thread(r).start();
        }
    }
}

class Int {
    int val = 0;

    synchronized public int getVal() {
        return val;
    }

    synchronized public void setVal(int val) {
        this.val = val;
    }

    synchronized public void addVal() {
        this.val++;
    }
}
```

结果：

```sheel
Thread[Thread-0,5,main]:a=2,b=1
Thread[Thread-2,5,main]:a=3,b=1
Thread[Thread-1,5,main]:a=2,b=1
```

### tryLock锁测试

当使用lock去获取锁的时候，可能会由于一直获取不到锁而使得程序陷入死等，这样使得程序在获取锁时不可控，tryLock方法试图去获取锁，但是如果获取失败，不会阻塞程序陷入死等，无论获取成功与否，tryLock方法都会返回。

应当注意，一旦获取成功，最好在finally子句中去unlock掉。

tryLock的使用，有两种方式，示例代码如下：

``` java
package com.ame;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/*
 * tryLock
 * */
public class Main5 {
    public static void main(String[] args) {
        Lock lock = new ReentrantLock();
        Runnable r1 = () -> {
            if (lock.tryLock()) {
                try {
                    System.out.println(Thread.currentThread() + "获取锁成功.r1");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            } else {
                System.out.println(Thread.currentThread() + "获取锁失败.r1");
            }
        };
        Runnable r2 = () -> {
            try {
                if (lock.tryLock(3, TimeUnit.SECONDS)) {
                    try {
                        System.out.println(Thread.currentThread() + "获取锁成功.r2");
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                } else {
                    System.out.println(Thread.currentThread() + "获取锁失败.r2");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };

        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r1);
        Thread t3 = new Thread(r1);
        Thread t4 = new Thread(r2);
        Thread t5 = new Thread(r2);
        Thread t6 = new Thread(r2);
        Thread t7 = new Thread(r2);

        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t5.start();
        t6.start();
        t7.start();
    }
}
```

对于有超时参数的tryLock，被阻塞时间控制在超时参数范围内，如果在该范围内获取失败返回false，成功返回true，中断抛出异常。

### 读写锁

在java.util.concurrent.locks包中定义了两个锁类：

1. 前面提到的可重入锁，ReentrantLock (implements Lock)。
2. 读写锁，ReentrantReadWriteLock (implements ReadWriteLock)。

对于可重入锁而言，可以解决几乎所有的并发问题，但是在特定的场景效率可能不高。

如果有多个线程从数据结构中读取数据，而只有一个或者少量线程向该数据结构中写入数据，这时候使用可重入锁，对于频繁的读操作来说，互斥访问是没有必要的，因为读不会造成并发竞争中的问题。这时候可以采用读写锁，多个读线程共享读锁，而写线程则是通过互斥的写锁同步。

1. 读-读共享。
2. 写-写、写-读互斥。

使用如下：

``` java
package com.ame;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/*
 * 读写锁
 * */
public class Main6 {
    static private ReadWriteLock lock = new ReentrantReadWriteLock();
    static private Lock readLock = lock.readLock();
    static private Lock writeLock = lock.writeLock();
    // r1:读线程
    static private Runnable r1 = () -> {
        readLock.lock();
        try {
            System.out.println("    " + Thread.currentThread() + "获取读锁");
            Thread.sleep(3000);
            System.out.println("    " + Thread.currentThread() + "读完毕。");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            readLock.unlock();
        }
    };
    // r2:写线程
    static private Runnable r2 = () -> {
        writeLock.lock();
        try {
            System.out.println(Thread.currentThread() + "获取写锁");
            Thread.sleep(1000);
            System.out.println(Thread.currentThread() + "写完毕。");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            writeLock.unlock();
        }
    };

    public static void main(String[] args) {
        f1();
        try {
            Thread.sleep(1000 * 5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("------------------");
        f2();
    }

    //读锁-读锁
    public static void f1() {
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r1).start();
    }

    //写锁-读锁、写锁-写锁
    public static void f2() {
        new Thread(r2).start();
        new Thread(r2).start();
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r2).start();
        new Thread(r2).start();
        new Thread(r2).start();
        new Thread(r1).start();
        new Thread(r1).start();
    }
}
```

结果：

``` shell
    Thread[Thread-0,5,main]获取读锁
    Thread[Thread-6,5,main]获取读锁
    Thread[Thread-1,5,main]获取读锁
    Thread[Thread-3,5,main]获取读锁
    Thread[Thread-2,5,main]获取读锁
    Thread[Thread-5,5,main]获取读锁
    Thread[Thread-4,5,main]获取读锁
    Thread[Thread-6,5,main]读完毕。
    Thread[Thread-4,5,main]读完毕。
    Thread[Thread-0,5,main]读完毕。
    Thread[Thread-3,5,main]读完毕。
    Thread[Thread-1,5,main]读完毕。
    Thread[Thread-2,5,main]读完毕。
    Thread[Thread-5,5,main]读完毕。
------------------
Thread[Thread-7,5,main]获取写锁
Thread[Thread-7,5,main]写完毕。
Thread[Thread-8,5,main]获取写锁
Thread[Thread-8,5,main]写完毕。
    Thread[Thread-9,5,main]获取读锁
    Thread[Thread-10,5,main]获取读锁
    Thread[Thread-11,5,main]获取读锁
    Thread[Thread-13,5,main]获取读锁
    Thread[Thread-12,5,main]获取读锁
    Thread[Thread-11,5,main]读完毕。
    Thread[Thread-12,5,main]读完毕。
    Thread[Thread-10,5,main]读完毕。
    Thread[Thread-9,5,main]读完毕。
    Thread[Thread-13,5,main]读完毕。
Thread[Thread-14,5,main]获取写锁
Thread[Thread-14,5,main]写完毕。
Thread[Thread-15,5,main]获取写锁
Thread[Thread-15,5,main]写完毕。
Thread[Thread-16,5,main]获取写锁
Thread[Thread-16,5,main]写完毕。
    Thread[Thread-17,5,main]获取读锁
    Thread[Thread-18,5,main]获取读锁
    Thread[Thread-17,5,main]读完毕。
    Thread[Thread-18,5,main]读完毕。
```



## 线程中断和终止

### 终止

当run方法执行完成，返回之后，线程终止，进入终止态。

### 中断

#### 中断状态

在早期的Java中，线程有一个stop方法，调用该方法可以强制终止一个该线程，然而后来的Jdk已经弃用这个方法了。

现在java改成了通过中断，来“通知”线程，请求终止，然而是否要响应中断则是由线程自主决定，可以响应，也可以不响应。

Java中，每一个线程都具有一个中断状态位，当为true时说明该线程处于中断状态（但是可以不响应），false说明处于非中断状态。

关于线程中断，主要由三个方法需要了解：

1. interrupt方法。实例方法，用来中断线程（给中断状态置位）。
2. isInterrupted方法。实例方法，用来检测该线程是否处于中断状态（不会改变中断状态）。
3. static Interrupted方法。静态方法，用来检测当前线程的中断状态，并清除原因状态（会改变中断状态）。

使用实例：

``` java
package com.ame;

public class Main7 {
    private static Runnable r1 = () -> {
        for (int i = 0; i < 500000; i++) {
            System.out.println(i);
        }
        System.out.println("打印完了");
    };
    private static Runnable r2 = () -> {
        for (int i = 0; i < 500000; i++) {
            if (Thread.interrupted()) {
                System.out.println("被中断了");
                break;
            }
            System.out.println(i);
        }
        System.out.println("打印完了");
    };

    public static void main(String[] args) {
        //f1();
        //f2();
    }

    /*
     * 不响应中断
     * */
    public static void f1() {
        Thread t = new Thread(r1);
        t.start();
        try {
            Thread.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        t.interrupt();
    }

    /*
     * 响应中断
     * */
    public static void f2() {
        Thread t = new Thread(r2);
        t.start();
        try {
            Thread.sleep(20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        t.interrupt();
    }
}
```

执行f1的结果：

``` shell
...
499997
499998
499999
打印完了
```

执行f2的结果：

``` shell
1987
1988
被中断了
打印完了
```

#### 中断异常

要想时不时地去检测中断位，就需要线程一直运行，然而很多时候，线程是没有处于运行状态的，当线程不是运行状态而是等待（或时间等待）状态（调用了wait、await、sleep、超时tryLock等）时，线程被中断将会**抛出InterruptException异常并复位**。因此每当调用wait、sleep等方法时，都需要捕获InterruptException异常。

对于捕获到InterruptException之后，要注意不要对这个异常不管不顾，或者仅仅是打印or记录日志，这对线程调用者来说是不礼貌的，线程调用者无法获知线程是否能够顺利地完成任务，因此就算不作更多的处理，至少也应该在catch子句中将中断状态置位，让线程调用者可以侦测到你的状态。

### 关于已经被其用的stop、suspend和resume方法

stop和suspend这两个方法，一个用来终止一个线程，另一个用来挂起一个线程直到resume被调用，这些方法都有一个共同点：**试图控制一个线程的行为**。这些方法天生就**不安全**，并且很容易导致**死锁**：

1. stop方法会立即释放所有的锁并结束run方法。

   获取锁是为了让临界区的操作具有原子性，而stop方法不管不顾立即释放锁，会破坏临界区操作的原子性，使得对象处于不一致状态，因此被弃用。

2. suspend方法会立刻无限制地挂起一个线程。

   锁不会被释放，因此锁也被无限制地挂起，这时候如果resume调用之前需要获取同一个锁，那就会陷入死锁：没有锁无法resume，没有resume无法获得锁。

