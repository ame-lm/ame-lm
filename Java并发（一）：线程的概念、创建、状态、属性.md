# Java并发（一）：线程的概念、创建、状态、属性

操作系统可以同时运行多个应用程序，这叫做多进程并发，当然在多处理机系统上也可以并行。而多线程则是指在一个程序（进程）中，可以同时执行多个任务（线程）。

线程和进程的本质区别在于，进程拥有自己独立的内存空间，而线程没有（因此，进程切换开销比线程开销大主要是因为进程地址空间置换（页表））。

## 创建并使用线程

Java的线程用Thread类进行封装，每一个线程都是一个Thread类的实例对象。当线程实例被创建，通过Thread.start()方法来启动线程，线程启动后会自动地执行其中的run方法，但是Thread类默认的run方法什么也不做。

因此想要真正地创建并使用一个线程，需要覆盖掉Thread类中的run方法。这里有两种方式覆盖：继承Thread类和提供Runnable构造参数。

### 继承Thread类（不推荐）

通过继承Thread类，自然能够重写run方法，给线程赋予实质功能。

但是不推荐这种做法，由于继承的单一性，当自定义线程类继承自Thread类之后，将无法在继承其他类，会带来一些不便。

### 提供Runnable构造参数

通过提供Runnable作为构造参数，也可以创建Thread实例，其中的run方法会自动调用Runnable的run方法，完成了run方法覆盖。

### 示例

``` java
package com.ame;

/*
 * 创建并使用线程。
 * */
public class Main1 {
    public static void main(String[] args) {
        Thread t1 = new MyThread1();
        Thread t2 = new Thread(new MyThread2());
        t1.start();
        t2.start();
    }
}

class MyThread2 implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println("" + i + ".hello:" + Thread.currentThread());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class MyThread1 extends Thread {
    @Override
    public void run() {
        super.run();
        for (int i = 0; i < 3; i++) {
            System.out.println("" + i + ".hello:" + Thread.currentThread());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

结果：

``` shell
0.hello:Thread[Thread-0,5,main]
0.hello:Thread[Thread-1,5,main]
1.hello:Thread[Thread-0,5,main]
1.hello:Thread[Thread-1,5,main]
2.hello:Thread[Thread-1,5,main]
2.hello:Thread[Thread-0,5,main]
```

## 线程状态

Java中线程有六个状态：NEW、Runnable、Blocked、Waiting、Timed Waiting、Terminated。

1. 当Thread实例被创建时，线程处于NEW状态。

2. 当start方法被调用时，线程处于Runnable状态。该状态下的线程可能正在运行，也可能没有运行（但随时有可能运行），这取决于操作系统调度。

3. 当线程试图获取对象锁（而不是并发包中的锁），而该锁被别的线程持有时，线程进入Blocked状态，直到成功拿到对象锁才退出Blocked状态（一般是进入Runnable状态）。

4. 当线程通知调度器一个条件（wait方法、join方法、等待并发包中锁）时，陷入Waiting状态。

5. 有些方法（sleep、wait、join、tryLock、await等）有超时参数，当这些方法被调用时，进入Timed Waiting状态，直到超时或者被唤醒（条件满足）时退出。
6. 当线程run方法运行完毕或者run方法抛出异常时，线程会终止，进入Terminated状态。

![image-20200812091431405](https://gitee.com/ame-lm/PicBed/raw/master/img/20200909155244.png)

## 线程属性

线程拥有各种属性，包括：优先级、守护线程、线程组和未捕获异常处理器。

### 线程优先级

在cpu调度线程时，优先级高的优先调度。Java的优先级包含十个等级，然而操作系统可能不是是个优先级，可能更多，也可能更少，Java的十个优先级会映射到操作系统中去。

注意：不要将程序构建为正确性依赖于优先级。

1. 默认优先级继承于父线程（最初是NORM-5）。
2. 设置优先级：setPriority方法。
3. 获得优先级：getPriority方法。
4. 优先级常量：MIN_PRIORITY（1）、MAX_PRIORITY（10）、NORM_PRIORITY（5）。

### 守护线程

通过t.setDaemon(true/false)可以将线程设置为守护线程/普通线程。守护线程的唯一作用是为其他线程提供服务。例如：垃圾回收、计时器等。

当虚拟机中有普通线程的时候，虚拟机不会退出，当虚拟机中普通线程全部终止，虚拟机就可能会退出。因此守护线程极有可能在任何时候被意外终止，所以守护线程不应该访问固有资源（极有可能还没释放就被终止了）。

### 线程组和未捕获异常处理器

线程组是一个可以线程集合，可以方便地管理多个线程，但是现在有更好的特性操作线程集合，因此建议**不要使用线程组**。默认所有的线程都属于一个线程组。

在run方法中，不能抛出任何受查异常，但是有可能抛出非受查异常，当出现非受查异常时，线程将会终止，线程终止前会将异常发送到对应的异常处理器进行处理，然后才终止。

异常处理器接口：Thread.UncaughtExceptionHandler，仅包含一个方法：void uncaughtException(Thread t,Throwable e)。

可以调用setUncaughtExceptionHandler方法为线程安装处理器，也可以调用Thread.setDefaultUncaughtExceptionHandler方法为所有的线程安装默认处理器，如果没有手动安装处理器，那么默认的处理器就是ThreadGroup对象（所属线程组），在默认的ThreadGroup对象中，对异常处理如下：

1. 如果有父线程组，那么由父线程组处理。
2. 如果有默认处理器Thread.getDefault...()，那么交由该处理器处理。
3. 如果Throwable是ThreadDeath的一个实例，那么不做任何事（线程正常死亡）。
4. 将线程名字和Throwable栈轨迹输出到System.err。这就是代码写错时，我们多次看到的。