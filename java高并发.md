# Java 高并发

## 多线程

### ✔️:介绍一下 Java 的内存模型（JMM）

JMM，也就是 Java 内存模型，是 Java 定义的一套多线程访问共享变量的规则。简单说就是规定了一个线程修改变量后，其他线程什么时候能看到，以及哪些线程的先后顺序是有保证的。

理解 JMM，主要看 3 个问题：可见性、原子性和有序型

##### 可见性

比如两个线程共享一个停止标志，线程 A 把标志改成了 true，希望线程 B 看到后结束循环。但如果没有同步措施，线程 B 就可能一直看不到这个变化。因为处理器有缓存，编译器也会做优化，线程读取共享变量时，不一定每次都能观察到其他线程的最新修改。

JMM 用主内存和工作内存来描述这种交互：共享变量存放在主内存中，线程通过自己的工作内存来操作变量。这是一种抽象模型，不能把工作内存直接等同于 CPU 缓存，也不能把主内存直接等同于 JVM 堆。

对于停止标志这种场景，可以使用 volatile：

```java
private volatile boolean stopped = false;

public void run() {
    while (!stopped) {
        // 执行任务
    }
}
public void stop() {
    stopped = true;
}
```

##### 原子性

比如 i++，看着是一行代码，其实包含 "读取i、加 1、写回i" 三个步骤。如果两个线程读取到同一个数再加1写回去就会损失一次更新。

这时候只加 volatile 没用，因为他不能把这三个步骤变成一个不可分割的操作。可以使用 AtomicInteger，或者用 synchronized、ReentrantLock 把整个操作保护起来。

##### 有序性

编译器和处理器为了提高效率可能调整指令的执行顺序，只要不改变单线程程序的执行结果就行。但在多线程环境下，这种调整可能让另一个线程看不到符合预期的状态，比如拿到了对象引用，对象却还没有初始化完成

JMM 通过 happens-before 规则来规定哪些操作之间具有可见性和顺序保证，比如：

volatile 规则：对一个 volatile 变量的写，happens-before 后续对这个变量的读

锁规则：对一把锁的解锁，happens-before 后续对同一把锁的加锁。

线程启动规则：调用线程的 start()，happens-before 被启动线程中的操作。

线程结束规则：线程中的操作，happens-before 另一个线程通过 join() 成功等待它结束之后的操作

传递性：如果 A happens-before B，B happens-before C，那么 A happens-before C。

happens-before 不代表底层指令一定按这个顺序执行，而是要求最终观察到的结果符合这些规则。所以不能把 volatile 理解为禁止所有重排序，他限制的是会破坏这些保证的重排序。

### ✔️保证数据的一致性有哪些方案

事务管理：使用数据库事务来确保一组数据库操作要么全部成功提交，要么全部回滚失败。通过 ACID 属性，数据库事务可以保证数据的一致性

锁机制：使用锁来实现对共享资源的互斥访问。在 Java 中使用 synchronized 关键字、ReentrantLock 或其它锁机制来控制并发访问，从而避免并发操作导致数据不一致。

版本控制：通过乐观锁的方式，在更新数据时记录数据的版本信息，从而避免同时对同一数据进行修改

### ✔️线程的创建方式有哪些

##### 继承 Thread 类

自定义类继承 java.lang.Thread 类，重写其 run() 方法，run() 方法中定义了线程执行的具体任务。创建该类的实例后，通过调用 start() 方法启动线程。

```java
class MyThread extends Thread {
    @Override
    public void run() {
        // 
    }
}

public static void main(String[] args) {
    MyThread t = new MyThread();
    t.start();
}
```

优点：编写简单，如果需要访问当前线程，无需使用 Thread.currentThread() 方法，直接使用 this，即可获得当前线程。

缺点：因为线程类已经继承了 Thread 类，所以不能再继承其它父类

##### 实现 Runnable 接口

如果一个类已经继承了其他类，就不能再继承Thread类，此时可以实现 java.lang.Runnable 接口。实现 Runnable 接口需要重写 run() 方法，然后将此 Runnable 对象作为参数传递给 Thread 类的构造器，创建 Thread 对象后调用其 start() 方法启动线程

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        // 线程执行的代码
    }
}

public static void main(String[] args) {
    Thread t = new Thread(new MyRunnable());
    t.start();
}
```

优点：线程类只是实现了Runable接口，还可以继承其他的类。在这种方式下，可以多个线程共享同一个目标对象，非常适合多个相同线程来处理同一份资源的情况，从而可以将 CPU 代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想

缺点：如果要访问当前线程，必须使用 Thread.currenThread() 方法

##### 实现 Callable 接口与 FutureTask

java.util.concurrent.Callable 接口类似于 Runnable，但 Callable 的 call() 方法可以有返回值并且可以抛出异常。要执行 Callable 任务，需将它包进一个 FutureTask，因为 Thread 类的构造器只接受 Runnable 参数，而 FutureTask 实现了 Runnable 接口

```java
class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        return 1;
    }
}

public static void main(String[] args) {
    MyCallable task = new MyCallable();
    FutureTask<Integer> futureTask = new FutureTask<>(task);
    Thread t = new Thread(futureTask);
    t.start();

    try {
        Integer result = futureTask.get();
        System.out.println("Result: " + result);
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}
```

缺点：编程较为复杂，如果要访问当前线程，必须调用 Thread.currentThread() 方法

优点：线程类只实现 Callable 接口，仍然可以继承其他类；且 call() 方法可以有返回值，也可以抛出异常，配合 FutureTask 能方便地获取异步执行结果

##### 使用线程池

从 Java 5 开始引入的 java.util.concurrent.ExecutorService 和相关类提供了线程池的支持，这是一种更高效的线程管理方式，避免了频繁创建和销毁线程的开销。可以通过 Executors 类的静态方法创建不同类型的线程池。

```java
class Task implements Runnable {
    @Override
    public void run() {
        // 线程执行的代码
    }
}

public static void main(String[] args) {
    ExecutorService executor = Executors.newFixedThreadPool(10);
    for (int i = 0; i < 100; i++) {
        executor.submit(new Task());  // 提交任务到线程池执行
    }
    executor.shutdown();  // 关闭线程池
}
```

缺点：线程池增加了程序的复杂度，特别是当涉及线程池参数调整和故障排查时。错误的配置可能导致死锁、资源耗尽等问题。

优点：线程池可以重用预先创建的线程，避免了线程的创建和销毁开销，提高了程序性能。对于需要快速响应的并发请求，线程池可以迅速提供线程来处理任务，减少等待时间。并且，线程池能够有效控制运行的线程数量，防止因创建过多线程导致的系统资源耗尽。通过合理配置线程池大小，可以最大化 CPU 利用率和系统吞吐量。

### ✔️使用 interrupt 是如何让线程抛出异常的

interrupt() 本身是发出一个中断请求，是否抛出异常，要看目标线程正在做什么。

正在正常运行：通常只是设置中断标志，线程需要主动调用 isInterrupted() 检查，不会在任意一行代码上突然抛异常。

正在执行 sleep()、wait()、join() 等可中断等待：这些方法会响应中断，结束等待，并在目标线程中抛出 InterruptedException，同时清除中断标志。调用这些方法前已经有中断标志，也可能直接抛异常

正在执行 LockSupport.park()：中断可以使它返回，但不会抛出 InterruptedException，也不会清除中断标志

正在等待进入 synchronized，或者调用 ReentrantLock.lock() 等待锁：不能靠中断直接取消这次锁等待。需要可中断的锁等待时，应该用 lockInterruptibly()

所以，中断标志更像一个 "请停止" 的信号，而不是线程的强制退出开关。

处理 InterruptedException 时，如果当前方法不能继续完成任务，可以向上抛出；如果不能向上抛出，通常需要恢复中断标志，并结束当前任务。不要只打印异常，然后继续原来的循环。

另外，isInterrupted() 只检查状态，Thread.interrupted() 检查的是当前线程，并且会清除中断标志，两者不要混用。

### ✔️Java 线程的状态有哪些

调用线程 Thread 中的 getState() 方法获取当前线程的状态。

| 线程状态 | 解释 |
| --- | --- |
| NEW | 尚未启动的线程状态，即线程创建，还未调用 start 方法 |
| RUNNABLE | 就绪状态（调用 start，等待调度）+ 正在运行 |
| BLOCKED | 等待监视器锁时，陷入阻塞状态 |
| WAITING | 等待状态的线程正在等待另一线程执行特定的操作（notify） |
| TIMED_WAITING | 具有指定等待时间的等待状态 |
| TERMINATED | 线程完成执行，终止状态 |

### sleep 和 wait 的区别是什么

sleep() 和 wait() 都可以让线程暂停执行，但用途不一样：sleep 主要是让当前线程休眠，wait 主要是配合锁进行线程间协作。

| 对比项 | Thread.sleep() | Object.wait() |
| 所属类 | Thread，静态方法 | Object，实例方法 |
| 调用前提 | 不需要持有锁 | 必须持有调用对象的监视器锁 |
| 是否释放锁 | 不释放已经持有的锁 | 释放调用对象的监视器锁 |
| 线程状态 | 通常是 TIMED_WAITING | 无超时是 WAITING，有超时是 TIMED_WAITING |
| 结束等待的原因 | 时间到，或者被中断 | 通知、超时、中断，也可能虚假唤醒 |

## 并发安全

### juc 包下你常用的类

##### 线程池相关

ThreadPoolExecutor：最核心的线程池类，用于创建和管理线程池。通过它可以灵活地配置线程池的参数，如核心线程数、最大线程数、任务队列等，以满足不同的并发处理需求

Executors：线程池工厂类，提供了一系列静态方法来创建不同类型的线程池，如 newFixedThreadPool（创建固定线程数的线程池）、newCachedThreadPool（创建可缓存线程池）、newSingleThreadExecutor（创建单线程线程池）等，方便开发者快速创建线程池。

##### 并发集合类

ConcurrentHashMap：线程安全的哈希映射表，用于在多线程环境下高效的存储和访问键值对

CopyOnWriteArrayList：线程安全的列表，在对列表进行修改操作时，会创建一个新的底层数组，将修改操作应用到新数组上，而读操作仍可以在旧数组上进行，从而实现读写分离，提高了并发读的性能，适用于读多写少的场景

##### 同步工具类

CountDownLatch：允许一个或多个线程等待其他一组线程完成操作后再继续执行。它通过一个计数器来实现，计数器初始化为线程的数量，每个线程完成任务后调用 countDown 方法将计数器减一，当计数器为 0 时，等待的线程可以继续执行。常用于多个线程完成各自任务后，再进行汇总或下一步操作的场景。

CycliBarrier：让一组线程互相等待，直到所有线程都到达某个屏障点后，计数器会重置，可以再次用于下一轮的等待。适用于多线程需要协同工作，在某个阶段完成后再一起进入下一个阶段的场景。

Semaphore：信号量，用于控制同时访问某个资源的线程数量。它维护了一个许可计数器，线程在访问资源前需要获取许可，如果有可用许可就获取成功并将许可计数器减一，否则线程需要等待，直到有其他线程释放许可。常用于控制对有限资源的访问，如数据库连接池，线程池中的线程数量等

##### 原子类

AtomicInteger：原子整数类，提供了对证书类的原子操作，比如自增、自减、比较并交换等。通过硬件级别的原子指令来保证操作的原子性和线程安全性，避免了使用锁带来的性能开销，在多线程环境下对整数进行计数、状态标记等操作非常方便。

# AtomicReference：原子引用类，用于对对象引用进行原子操作。

### ✖️:ConcurrentHashMap 是怎么保证线程安全的

以 JDK 8 的实现为例回答：底层使用数组、链表和红黑树，通过 CAS 和局部加锁来协调更新，普通 get() 通常不加锁

##### 写入

一个桶还没有节点时，可以通过 CAS 放入第一个节点；桶已经有节点时，对这个桶的更新通常需要对桶头等进行同步，再处理链表或树中的节点。这样不同桶上的写入在没有其他冲突时可以并行进行，不用每次修改都锁住整个 Map。树结构的内部协调还有自己的机制。

##### 读取

相关节点字段通过 volatile 等机制保证可见性，get() 可以读取对应桶和节点

### ✔️:synchronized 和 ReentrantLock 的工作原理及其应用场景

两者都可实现可重入的互斥访问，并提供相应的内存可见性保证。主要区别在于实现方式和提供的功能。

##### synchronized 工作原理

synchronized 使用对象关联的监视器锁。同步代码块通常对应 monitorenter、monitorexit 字节码指令，同步方法则通过方法上的 ACC_SYNCHRONIZED 标志，由 JVM 在调用和退出时处理加锁、解锁。

同一时刻只有一个线程能持有同一把锁，持有者可以再次进入使用这把锁的同步代码。退出一次同步范围，就释放一次重入，完全释放后其他线程才有机会获取锁。

如果调用 wait()，线程会释放调用对象的监视器锁（就是 obj.wait() 的 obj），进入等待集合，即使之前重入了多次，wait() 也会释放这个监视器的全部持有，被唤醒后还要重新获取锁，一旦拿到锁，JVM 会把重入计数器恢复成 wait 调用前的值，然后 wait 才返回。

JVM 会针对不同的竞争情况优化 synchronized，不能认为每次加锁都要调用操作系统互斥量，或者每次都要发生线程阻塞。

##### ReentrantLock 工作原理

ReentrantLock 基于 AQS 实现，主要依靠同步状态 state、持有锁的线程信息，以及等待队列。

首次获取：锁空闲时，通过 CAS 等方式尝试修改 state，成功后记录持有线程。

再次获取：如果当前线程已经持有锁，就增加 state，记录重入次数

获取失败：根据调用的 API，选择排队等待、响应中断、等待超时，或直接返回失败

释放锁：减少重入次数，完全释放后唤醒正在等待的线程，使其重新尝试获取。

它默认是非公平锁，可以通过 new ReentrantLock(true) 创建公平锁。lockInterruptibly() 支持可中断等待，tryLock(timeout, unit) 支持超时等待，多个 Condition 可以区分不同的等待条件。

##### 应用场景的区别

简单的共享数据保护可以优先用 synchronized。比如对一组字段进行更新，代码块比较短，也没有特殊的等待要求，自动释放锁能减少使用错误。

需要控制获取锁的过程，可以使用 ReentrantLock。比如等待锁超过 100 毫秒就放弃、任务取消后结束锁等待，或者生产者和消费者分别等待 notFull、notEmpty 两个条件。

性能比较要结合实际的 JDK 版本、竞争程度和临界区耗时，不能直接说 ReentrantLock 一定比 synchronized 快。

### ✖️:syncronized 和 ReentrantLock 的区别

synchronized 和 ReentrantLock 都是可重入的互斥锁，都能用于保护共享数据，区别主要有以下几个：

使用方式：synchronized 是关键字，进入同步范围时自动加锁，退出时自动释放；ReentrantLock 是类，需要显示加锁，并在 finally 中解锁

公平性：synchronized 不保证公平，ReentrantLock 默认非公平，但可以配置

中断和超时：等待进入 synchronized 不能通过中断直接取消

等待通知

底层实现：

### synchronized 锁升级的过程

### 介绍一下 AQS

AQS 全称 AbstractQueuedSynchronizer，是 Java 用来构建锁和同步器的基础框架。像 ReentrantLock、Semaphore、CountDownLatch，都使用了 AQS。

简单说，AQS 负责同步状态的管理，以及获取失败后的排队、阻塞和唤醒；具体"怎样才算获取成功"由其子类决定

它的核心可以分为 3 部分

##### 同步状态 state

AQS 使用一个 volatile int 类型的 state 表示同步状态，不同工具里它的含义不同。

ReentrantLock：表示锁的持有次数，0 表示未被持有，正数表示已经持有及其重入次数。

Semaphore：表示可用许可数量。

CountDownLatch：表示还需要完成多少次倒数。

子类可以通过 getState()、setState()、compareAndSetState() 访问状态。这里要注意，volatile 保证可见性，但涉及并发竞争的"读取后修改"，仍然需要 CAS 等措施。

##### 等待队列

获取资源失败后，AQS 会把等待线程组织到一个双向队列里。这个队列借鉴了 CLH 队列的思路，但不是让所有线程一直自旋，而是配合 LockSupport.park()、unpark() 进行等待和唤醒。

比如线程 A 持有锁，线程 B 获取失败，B 就可能进入队列并等待。A 完全释放锁后，会唤醒后继等待线程，让它重新尝试获取。

被唤醒不代表直接拿到了锁，等待线程仍然要再次判断同步状态，并尝试获取资源。

##### 子类实现获取和释放的规则



### ✖️CAS 有什么缺点？

##### ABA 问题

线程读到 A，准备更新时发现还是 A，但中间可能已经被改成 B，再改回 A。CAS 只比较当前值，发现不了这段变化。如果业务关心状态是否经历过修改，可以把版本号一起比较，比如使用 AtomicStampedReference。并不是所有场景都会受到 ABA 影响，要看中间的变化是否会破坏业务判断。

##### 竞争激烈时重试成本高

CAS 本身只是一次尝试，是很多基于 CAS 的算法在失败后循环重试。大量线程更新一个变量时，可能反复失败，消耗 CPU，并产生缓存一致性开销。可以根据场景使用锁，或者像 LongAdder 一样分散计数竞争。

##### 不能直接保护多个独立变量的整体操作

### CAS 有什么问题，Java 是怎么解决的

### ✔️volatile 关键字有什么作用

主要有两个作用：保证可见性以及提供特定的顺序保证，但它不提供互斥，不能把复合操作变成原子操作

##### 保证可见性

一个线程对 volatile 变量的写，happens-before 后续对这个变量的读。读线程不只是能看到这个标志本身的变化，还能看到写线程在这次 volatile 写之前完成的相关修改。

比如下面的代码，假设 publish() 由一个线程执行，read() 由另一个线程执行，发布后不再修改 value：

```java
private int value = 0;
private volatile boolean ready = false;

public void publish() {
    value 
= 42;
    ready 
= true;
}

public void read() {
    if (ready) {
        System.out.println(value); // 读到 ready 为 true 时，value 是 42
    }
}
```

虽然 value 没有加 volatile，但它的写入发生在 ready = true 之前。读线程观察到这次 ready 的写入后，也能看到前面的 value 修改。在本次 volatile 写之后再修改 value，该修改无法依靠这次 volatile 写获得可见性保证。

##### 限制重排序

编译器和处理器不能进行破坏 volatile 内存语义的重排序（但不是完全禁止重排）。比如上面的数据写入，不能让读线程看到 ready 已经是 true、value 却仍是 0。

底层通常通过编译器约束、内存屏障和处理器的缓存一致性机制来实现。提到 StoreStore、StoreLoad、LoadLoad、LoadStore 时，可以把它们理解为限制不同读写操作排序的屏障类型，但具体指令和屏障组合取决于处理器架构、JVM 实现及优化。

不能简单理解成 volatile 读写完全绕过 CPU 缓存，也不能说它禁止所有指令重排序。

它适合停止标志、状态发布，以及双重检查单例中的安全发布。对于 count++ 这种“读取、计算、写回”的操作，仍然需要原子类或锁。

### ✔️指令重排序的原理是什么

在执行程序时，为了提高性能，处理器和编译器常常会对指令进行重排序，但是重排序要满足以下2个条件才能进行：

1.在单线程环境下不能改变程序运行的结果，

2.存在数据依赖关系的时候不允许重排序。

所以重排序不会对单线程有影响，只会破坏多线程的执行语义。

### ✔️volatile 和 synchronized 比较

两者都能用于线程之间的可见性和顺序控制，但 synchronized 还提供互斥，volatile 不提供。

| 对比项目 | volatile | synchronized |
| --- | --- | --- |
| 使用范围 | 修饰字段 | 修饰方法或代码块 |
| 可见性 | volatile 写与后续读建立 happens-before | 同一把锁的解锁与后续加锁建立 happens-before |
| 复合操作 | 不能保证整体原子性 | 可以保护整个临界区，相关访问要使用同一把锁 |
| 是否阻塞 | 不会像互斥锁一样排队等待锁 | 存在竞争时可能自旋或阻塞 |
| 常用用途 | 标志位、状态位发布 | 多个步骤或多个字段的互斥更新 |

比如只是通知线程“任务该停止了”，可以使用 volatile 标志；如果要判断余额够不够，再扣减余额，就需要把判断和修改作为一个整体保护，不能只给余额加 volatile。

开销上，volatile 通常比获取互斥锁更轻，但它提供的能力也更少。选择时先看正确性要求，不能为了减少开销，把需要互斥的操作直接换成 volatile。

### ✔️什么是公平锁和非公平锁

主要区别在于新来的线程是否需要尊重已经排队等待的线程。

公平锁：获取锁时通常会检查前面是否还有等待者，优先让等待时间更长的线程获取，减少插队和长期等待

非公平锁：允许新来的线程先尝试获取锁，获取失败后再排队。这样可能出现后来的线程先拿到锁，但通常能减少交接期间的空闲，提高吞吐量。ReentrantLock 默认就是非公平锁。

不过，公平不代表所有线程严格按调用方法的时间执行，也不代表操作系统的线程调度公平，持有锁的线程可以重入，无参 tryLock() 也可以绕过公平性设置

### ReentrantLock 是怎么实现公平锁的

核心是在首次获取锁时检查当前线程前面是否存在排队的等待者。

从公平锁和非公平锁的两段获取逻辑可以看出来公平锁在锁空闲时多了 hasQueuedPredecessors() 检查：当前线程前面没有排队的等待者才尝试 CAS 获取（不是判断整个队列是否为空，当前线程自己可能已经位于队列的最前面）。

## 线程池

### ✖️线程池怎么使用

线程池的核心作用是复用线程、控制并发数，避免频繁创建销毁线程带来的性能开销。

可以通过 Executors 工具类快速创建线程池，适合简单场景，不用手动配置复杂参数。比如需要一个固定线程数的线程池，处理一批异步任务：

```java
// 创建固定3个线程的线程池
ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);

// 提交 Runnable 任务，无返回值
for (int i=0; i<5; i++) {
    int taskNum = i;
    fixedThreadPool.submit(() -> {
        System.out.println("当前线程 " + Thread.currentThread().getName() + " 处理任务 " + taskNum);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    });
}

// 提交 Callable 任务，有返回值
Future<Integer> future = fixedThreadPool.submit(() -> {
    Thread.sleep(500);
    return 100+200;  // 任务返回结果
});

// 获取 Callable 任务的返回值，会阻塞直到任务完成
try {
    Integer result = future.get();
    System.out.println("任务返回结果: " + result);
} catch (Exception e) {
    e.printStackTrace();
}

// 任务执行完后关闭线程池 避免资源泄露
fixedThreadPool.shutdown();
```

Executors 创建的线程池有潜在问题，比如 newCachedThreadPool 可能创建大量线程导致 OOM，newScheduledThreadPool 的 maximumPoolSize 为 Integer.MAX_VALUE 且内部使用无界的 DelayedWorkQueue，任务堆积时可能导致 OOM，所以复杂业务场景更推荐用 ThreadPoolExecutor 手动配置，能精准控制线程池行为。

手动创建 ThreadPoolExecutor 需要指定 7 个核心参数

### execute 和 submit 有什么区别

两者都能提交任务，主要区别是可提交的任务类型、返回值，以及异常如何被观察到。

execute()：接受 Runnable 任务，没有返回值，无法拿到 Future 对象，后续不能主动获取任务结果，也不能取消任务

submit()：接受 Runnable 或 Callable 任务，返回 Future。Callable 可以返回结果，Runnable 也可以通过 Future 判断完成或取消

对于常见的 ThreadPoolExecutor，任务通过 execute() 执行时，如果有未捕获的运行时异常，会离开任务执行流程，通常导致这个工作线程退出，异常可以交给线程的未捕获异常处理器。线程池会根据状态和配置补充工作线程。

通过 submit() 提交时，任务通常会被包装成 FutureTask。任务异常会保存在 Future 里，调用 get() 时以 ExecutionException 的形式抛出，原始原因可以通过 getCause() 获取。



### ✖️:CompletableFuture 和 Future 有什么区别

Future 主要用于获取异步结果、等待完成和取消任务，CompletableFuture 在这些能力上进一步支持异步任务之间的组合和完成后的回调。

比如先查询用户，再用用户信息查询订单；或者同时查询价格和库存，等两个结果都回来后组合。这些场景用普通 Future 往往要手动调用 get()，用 CompletableFuture 可以表达任务之间的关系。

CompletableFuture 的几个常见方法：

supplyAsync()/runAsync()：启动有返回值或无返回值的异步任务

thenApply()：属于 CompletableFuture 的串行回调方法，上一个异步阶段正常完成并产出结果后，执行一段转换逻辑，该转换是同步计算，接收上一步的结果作为入参，输出一个新的普通对象，框架自动把这个结果包装成新的 CompletableFuture，形成异步链路

thenCompose()：也是 CompletableFuture 的串行回调方法，等待上一个异步阶段正常完成拿到结果，然后用这个结果发起下一个异步任务，函数返回值本身就是 CompletableFuture<R>，它会自动做扁平化，避免出现 CompletableFuture<CompletableFuture<R>> 这种双层嵌套

thenCombine()：是 CompletableFuture 的并行组合回调方法，等待两个互相独立的异步任务都正常执行完成，拿到两者的结果，再执行一段同步合并逻辑，把两个结果加工成一个新值，自动包装成新的 CompletableFuture。

allOf()：等待多个阶段全部完成，返回 CompletableFuture<Void>，各个结果仍需从原来的 Future 获取

exceptionally()/handle()：处理失败，或者把成功和失败都转换成后续结果

```java
import java.util.concurrent.*;

class CompletableFutureExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(2);
        try {
            CompletableFuture<Integer> price =
                    CompletableFuture.supplyAsync(() -> 100, executor);
            CompletableFuture<Integer> stock =
                    CompletableFuture.supplyAsync(() -> 20, executor);
            CompletableFuture<String> result = price.thenCombine(stock,
                    (p, s) -> "价格：" + p + "，库存：" + s);
            System.out.println(result.join());
        } finally {
            executor.shutdown();
        }
    }
}
```

不指定执行器的异步方法，通常使用 ForkJoinPool。



## 场景

### ✔️:多线程打印奇偶数，怎么控制打印的顺序

可以利用wait()和notify()来控制线程的执行顺序

```java
public class PrintOddEven {
    private static final Object lock = new Object();
    private static int count = 1;
    private static final int MAX_COUNT = 10;

    public static void main(String[] args) {
        Runnable printOdd = () -> {
            while (count <= MAX_COUNT) {
                synchronized (lock) {
                    while (count <= MAX_COUNT && count % 2 == 0) {
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                            return;
                        }
                    }
                    if (count > MAX_COUNT) {
                        lock.notify();
                        return;
                    }
                    System.out.println(Thread.currentThread().getName() + ": " + count++);
                    lock.notify();
                }
            }
        };

        Runnable printEven = () -> {
            while (count < MAX_COUNT) {
                synchronized (lock) {
                    while (count <= MAX_COUNT && count % 2 != 0) {
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                            return;
                        }
                    }
                    if (count > MAX_COUNT) {
                        lock.notify();
                        return;
                    }
                    System.out.println(Thread.currentThread().getName() + ": " + count++);
                    lock.notify();
                }
            }
        }

        Thread oddThread = new Thread(printOdd, "OddThread");
        Thread evenThread = new Thread(printEven, "EvenThread");
        oddThread.start();
        evenThread.start();
    }
}
```

synchronized 放在循环内部，锁的粒度最小化，减少持锁时间

wait 外层判断用 while 而不是 if，防止虚假唤醒

### 单例模型既然已经用了 synchronized，为什么还要加 volatile

这里说的是双重检查锁定的单例写法。

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
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

关键原因是：第一次检查和最终读取实例引用并不都在 synchronized 里，所以不能只靠锁来保证对象的安全发布

synchronized 的作用是避免多个线程同时创建实例。

volatile 的作用是让这个实例引用被安全地发布给没有获取锁的读线程

### ✔️:3个线程并发执行，1个线程等待这三个线程全部执行完再执行，怎么实现？

可以使用 CountDownLatch 来实现：

创建一个 CountDownLatch 对象，并将计数器初始化为 3，因为有 3 个线程要等待

创建 3 个并发执行的线程，在每个线程的任务结束时调用 countDown 方法将计数器减 1

创建第 4 个线程（或者用 main 线程也行），使用 await 方法等待计数器为 0，即等待其他 3 个线程完成任务

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    public static void main(String[] args) {
        CountDownLatch latch = new CountDownLatch(3);

        for (int i = 0; i < 3; i++) {
            final int threadNumber = i + 1;
            new Thread(() -> {
                try {
                    System.out.println("Thread " + threadNumber + " is working.");
                    Thread.sleep((long) (Math.random() * 1000));
                    System.out.println("Thread " + threadNumber + " has finished.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    // 恢复中断标记
                    Thread.currentThread().interrupt();
                } finally {
                    // 任务完成后, 计数器减 1
                    latch.countDown();
                }
            }).start();
        }

        new Thread(() -> {
            try {
                 System.out.println("Waiting for other threads to finish.");
                // 等待计数器为 0
                latch.await();
                System.out.println("All threads have finished, this thread starts to work.");
            } catch (InterruptedException e) {
                e.printStackTrace();
                // 恢复中断标记
                Thread.currentThread().interrupt();
            }
        }).start();
    }
}
```

### 
