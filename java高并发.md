# Java 高并发

## 多线程

### 介绍一下 Java 的内存模型（JMM）

JMM，也就是 Java 内存模型，是 Java 定义的一套多线程访问共享变量的规则。简单地说，就是规定了一个线程修改共享变量
后，其他线程什么时候看到，以及哪些操作的先后顺序是有保证的。

理解 JMM，主要看：可见性、原子性和有序性

##### 可见性

比如两个线程共享一个停止标志，线程 A 把标志改成了 true，希望线程 B 看到后结束循环。但如果没有同步措施，线程 B 可
能就一直看不到这个变化。

为什么会这样？因为处理器有缓存，编译器也会做优化，线程读取共享变量时不一定每次都能观察到其他线程的最新修改。

JMM 用主内存和工作内存来描述这种交互：共享变量放在主内存中，线程通过自己的工作内存来操作变量。不过，这是一种抽象
模型，不能把工作内存直接等同于 CPU 缓存，也不能把主内存直接等同于 JVM 堆。

对于停止标志这种场景，可以使用 volatile：`private volatile boolean stopped = false;`

##### 原子性

比如 `i++`，看着是一行代码，其实包含 "读取i，i+1，写回i" 三个步骤。如果两个线程都读到了 10，然后都把 11 写回去，
就丢失了一次更新。

这时候不能只加 volatile，他不能把这三个步骤变成一个不可分割的操作。可以

### wait 状态下的线程如何恢复到 RUNNABLE 状态

如果这里说的是调用 Object.wait() 后进入等待状态的线程，常见的结束等待方式有：

其他线程对同一个对象（线程调用 wait() 时锁的那个监视器对象）调用 notify() 或 notifyAll()。

等待线程被中断，随后抛出 InterruptedException。

调用了带超时时间的 wait()，等待时间到达

发生虚假唤醒(wait 的线程在没有被 notify 的情况下自行唤醒)

因为会发生虚假唤醒，所以代码不能写成 `if (条件不满足) { wait() }`，必须用 while 循环判断条件

不过结束等待不等于马上恢复执行。线程还需要重新获取调用 wait() 时释放的监视器锁，如果锁还被占用，就可能先处于 BLOCKED。

比如等待一个准备完成的标志，可以这样写（以下代码位于可以抛出 InterruptedException 的方法中）：

```java
synchronized (lock) {
    while (!ready) {
        lock.wait();
    }
}

// 其他线程修改条件，并通知等待线程
synchronized (lock) {
    ready = true;
    lock.notifyAll();
}
// ready 的检查和修改都放在同一把锁的保护下，这样即使通知先于等待发生，等待线程也会看到 ready 已经是 true，不再调用 wait()。 
```

如果说的是更广泛的 WAITING 状态，还要区分具体方法：join() （`t1.join()` 意味着当前线程在这里阻塞，等 t1 跑完）等待的是目标线程结束，park() （LockSupport.park() 意味着阻塞当前线程）可以由 unpark() 或中断等方式使其返回，不能都用 notify() 来唤醒

### notify 和 notifyAll 的区别

两者都是对某个对象的等待线程发出通知，调用时必须持有这个对象的监视器锁，而且调用后不会立即释放锁。

notify()：从这个对象的等待集合里选择一个线程通知，选择顺序没有 Java API 层面的保证。

notifyAll()：通知这个对象等待集合里的所有线程，让他们有机会重新竞争锁。

这里要区分 "被通知" 和 "获得锁"：即使调用 notifyAll() ，同一时刻也只有一个线程能拿到这把监视器锁。线程拿到锁之后，还要重新检查自己的等待条件，条件不满足就继续 wait()。

同一个对象上既有等待 "队列非空" 的消费者，也有等待 "队列未满" 的生产者。如果只调用 notify()，可能通知了一个条件仍不满足的线程，而真正能继续工作的线程还在等待，程序可能无法继续推进。

所以同一把锁对应多个等待条件或者不好判断应该通知谁时，通常用 notifyAll 更稳妥。它的代价是可能唤醒一些暂时不能继续工作的线程，增加锁竞争。

如果希望分别通知不同条件的线程，可以使用 ReentrantLock 配合多个 Condition。无论使用哪种方式，等待条件都应该放在 while 中检查。

### ReentrantLock + Condition 的具体使用方法

ReentrantLock + Condition 可以一把锁分出多个独立等待队列，signal 精准叫醒对应队列里的线程，互不干扰。

lock.newCondition()->创建一个独立等待队列

condition.await()=obj.wait()

condition.signal()=obj.notify()

condition.signalAll()=obj.notifyAll()

```java
class BoundedQueue<T> {
    private final T[] items;
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();
    // 两个独立等待队列
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public BoundedQueue(int capacity) {
        items = (T[]) new Object[capacity];
    }

    // 生产者：放数据
    public void put(T t) throws InterruptedException {
        lock.lock();
        try {
            // 队列满，生产者去 notFull 队列等待
            while (count == items.length) {
                notFull.await();
            }
            items[count++] = t;
            // 唤醒消费者队列的线程
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    // 消费者：拿数据
    public T take() throws InterruptedException {
        lock.lock();
        try {
            // 队列空，消费者去 notEmpty 队列等待
            while (count==0) {
                notEmpty.await();
            }
            T res = items[--count];
            // 拿完数据只唤醒 "生产者队列" 的线程
            notFull.signal();
            return res;
        } finally {
            lock.unlock();
        }
    }
}
```

### notify 选择的是哪个线程

从 Java API 的约定来看，notify() 选择哪个等待线程没有顺序保证，不能认为一定是最先等待的线程，也不能把它当作公平调度。

具体 JVM 的某个版本可能采用某种等待队列处理顺序，但这是实现细节，写程序时不能依赖它。

而且，被选择的线程只是有机会重新竞争监视器锁，并不是直接获得锁。它还要和其他竞争这把锁的线程一起等待，拿到锁后才能从 wait() 继续执行。

如果业务要求线程按指定顺序执行，就应该使用明确的状态变量、独立的 Condition 或 Semaphore 来控制，不能靠 notify() 恰好选中某个线程。

### 不同的线程之间如何通信

共享变量是最基本的线程间通信方式。多个线程可以访问和修改同一个共享变量，从而实现信息的传递。为保证线程安全，通常要使用 synchronized 关键字或 volatile 关键字。

Object 类中的 wait()、notify() 和 notifyAll() 方法可以用于线程间的协作。wait() 方法使当前线程进入等待状态，notify() 方法唤醒在此对象监视器上等待的单个线程，notifyAll() 方法唤醒在此对象监视器上等待的所有线程。

java.util.concurrent.locks 包中的 Lock 和 Condition 接口提供了比 synchronized 更灵活的线程间通信方式。Condition 接口的 await() 方法类似于 wait() 方法，signal() 方法类似于 notify() 方法，signalAll() 方法类似于 notifyAll() 方法。

java.util.concurrent 包中的 BlockingQueue 接口提供了线程安全的队列操作，当队列满时，插入元素的线程会被阻塞；当队列为空时，获取元素的线程会被阻塞。

### 线程间通信方式有哪些

### 如何停止一个线程

在 Java 中，停止线程的正确方式是通过协作式的逻辑控制线程终止，而非强制暴力终止（如已废弃的 Thread.stop()）

以下是实现安全停止线程的多种方法：

1.通过共享标志位主动终止。定义一个可见的状态变量，由主线程控制其值，工作线程循环检测该变量以及决定是否退出。

```java

```

2.使用线程中断机制。通过 Thread.interrupt() 触发线程中断状态，结合中断检测逻辑实现安全停止

3.通过 Future 取消任务。使用线程池提交任务，并通过 Future.cancel(true) 尝试取消任务，任务执行中的退出依赖它是否响应中断

4.处理不可中断的阻塞操作。不是所有等待线程都响应中断。比如普通的 ReentrantLock.lock() 不会因中断取消等待，要改用 `lockInterruptibly()` 。部分传统 IO 等待则需要关闭资源来解除阻塞

## 场景

### 多线程打印奇偶数，怎么控制打印的顺序

可以利用 wait() 和 notify() 来控制线程的执行顺序

```java
public class PrintOddEven {
    private static final Object lock = new Object();
    private static int count = 1;
    private static final int MAX_COUNT = 10;
    
}
```

### 单例模型既然已经用了 synchronized，为什么还要加 volatile?

这里说的是双重检查锁定的单例写法。关键原因是：

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {  // 第一次检查，不持有锁
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

synchronized 的作用是避免多个线程同时创建实例。第一次检查减少已经初始化后的加锁开销，第二次检查则避免多个线程先后进入同步块后重复创建。

volatile 的作用是让这个实例引用被安全地发布给没有获取锁的读线程。

instance = new Singleton() 可以粗略理解为分配内存、初始化对象、把引用赋给 instance。如果缺少安全发布，其他线程可能观察到 instance 已经不为 null，但对象初始化的结果还没有正确对它可见。

这时他会
