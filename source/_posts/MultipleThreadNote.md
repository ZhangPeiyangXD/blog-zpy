# Java 多线程学习笔记

---

## 1. 基本概念

### 并发与并行

**并发（Concurrency）**：同一时间段内，多个任务交替执行，但在同一时刻只有一个任务在执行。通常发生在单核CPU环境中。

**并行（Parallelism）**：同一时刻，多个任务真正同时执行。通常发生在多核CPU环境中。

### 进程与线程

**进程**：是操作系统资源分配的基本单位，每个进程都有独立的内存空间。

**线程**：是CPU调度的基本单位，同一进程内的线程共享进程的内存空间。

---

## 2. Java中多线程实现方式

Java中实现多线程主要有三种方式：
1. 继承Thread类
2. 实现Runnable接口
3. 实现Callable接口

核心类：Thread

### 2.1 Thread类常用方法

| 方法名           | 签名                               | 说明                          |
|---------------|----------------------------------|-----------------------------|
| start         | `void start()`                   | 启动线程，使其进入就绪状态，等待JVM调度执行     |
| run           | `void run()`                     | 线程的执行体，包含线程要执行的具体任务         |
| getName       | `String getName()`               | 返回该线程的名称                    |
| setName       | `void setName(String name)`      | 改变线程名称                      |
| currentThread | `static Thread currentThread()`  | 返回对当前正在执行的线程对象的引用           |
| setPriority   | `void setPriority(int priority)` | 改变线程的优先级                    |
| getPriority   | `int getPriority()`              | 返回线程的优先级                    |
| isAlive       | `boolean isAlive()`              | 测试线程是否处于活动状态                |
| setDaemon     | `void setDaemon(boolean on)`     | 设置线程为守护线程                   |
| sleep         | `static void sleep(long millis)` | 使当前正在执行的线程休眠指定毫秒数           |
| join          | `void join()`                    | 插入线程在当前线程之前，直到该线程结束或者该线程被中断 |
| yield         | `static void yield()`            | 暂停当前正在执行的线程对象，并执行其他线程       |
| interrupt     | `void interrupt()`               | 中断线程，设置线程的中断状态              |


>1.sleep()选择1毫秒的原因(根据实际目标运行时间修改)：
>- 足够短以不影响整体执行效率
>- 足够长以触发线程调度
>- 在大多数操作系统中都能有效触发线程切换。   
> 
>2.join()方法的作用：    
>- join()方法的主要作用是让当前线程等待调用join()方法的线程执行完毕后再继续执行。    
>- 换句话说，它会阻塞当前线程，直到目标线程完成其任务。    
### 2.2 继承Thread类

**创建新执行线程：**

```java
// 线程类
class MyThread extends Thread{
    public void run(){
        System.out.println("New Thread");
    }
}
```

> 重写run()方法：线程执行体，也就是线程要执行的任务。

**创建并启动线程：**

```java
MyThread myThread = new MyThread();
myThread.start(); // 启动线程
```

### 2.3 实现Runnable接口

**创建新执行线程：**

```java
// 任务类
class MyRunnable implements Runnable{
    long minPrime;
    
    // 构造方法,用来记录线程id
    MyRunnable(long minPrime){
        this.minPrime = minPrime;    
    }

    // 线程执行体
    public void run(){
        // 获取当前线程对象
        Thread t = Thread.currentThread();
        //...
    }
}
```

**创建并启动线程：**

```java
MyRunnable myRunnable = new MyRunnable(minPrime);
Thread t = new Thread(myRunnable);
t.start(); // 启动线程
```

> MyRunnable相当于一个任务，把这个任务传给一个线程t，t执行这个任务。

### 2.4 实现Callable接口（可以获取线程运行结果）

**创建新执行线程：**

```java
class MyCallable implements Callable<Integer>{
    
    @Override
    public Integer call() throws Exception {
        //运行逻辑
        return 1;
    }
}
```

**创建并启动线程：**

```java
MyCallable myCallable = new MyCallable();
FutureTask<Integer> futureTask = new FutureTask<>(myCallable);
Thread t = new Thread(futureTask);
t.start(); // 启动线程
Integer result = futureTask.get(); // 获取异步计算结果
System.out.println("result:" + result);
```

> **FutureTask**：
> 这是 Java 并发包中的一个类，它实现了 Runnable 接口，同时也实现了 Future 接口。    
> 它可以包装 Callable 或 Runnable 对象，使得它们可以在 Executor 框架中执行，并且可以获取异步计算的结果。    
> 
> **futureTask.get()**：阻塞式获取异步计算结果。

---

## 3. 线程相关方法的细节

### 3.1 线程调度

**Java中线程调度方式**：抢占式调度。优先级分为十档（Thread.MIN_PRIORITY到Thread.MAX_PRIORITY），默认是五（Thread.NORM_PRIORITY）。优先级高的线程会概率更高被执行。

### 3.2 守护线程

**守护线程**：守护线程是程序运行时在后台提供服务的线程，比如垃圾回收线程。当所有的非守护线程运行结束，守护线程会自动陆续结束。
> 不管该守护线程有没有运行完任务。

### 3.3 线程礼让

**礼让线程**：静态方法yield()，让当前正在执行的线程暂停，让出CPU。

### 3.4 线程插入

**插入线程**: 在当前线程前插入一个线程，插入的线程会从就绪状态变为运行状态。
> 插入的线程先执行完才执行当前线程。

---

## 4. 线程的生命周期

Java线程在其生命周期中会经历多种状态，每种状态都有其特定的含义和作用。

### 4.1 线程状态概述

Java线程主要有六种状态：
1. NEW（新建状态）
2. RUNNABLE（可运行状态）
3. BLOCKED（阻塞状态）
4. WAITING（等待状态）
5. TIMED_WAITING（限时等待状态）
6. TERMINATED（终止状态）

下面是Java线程的生命周期状态转换图：

```mermaid
graph TD
    A[新建状态<br/>New] -->|start()| B[就绪状态<br/>Runnable]
    B -->|获取CPU执行权| C[运行状态<br/>Running]
    C -->|失去CPU执行权| B
    C -->|wait()/join()| D[阻塞/等待状态<br/>Blocked/Waiting]
    C -->|sleep()/synchronized| E[限时等待状态<br/>Timed Waiting]
    D -->|notify()/notifyAll()| B
    E -->|时间到期| B
    C -->|run()结束| F[终止状态<br/>Terminated]
    D -->|中断/异常| F
    E -->|中断/异常| F
```

### 4.2 线程状态详解

#### NEW（新建状态）
线程刚被创建，但还没有调用start()方法。在这个状态下，线程已经分配了必需的系统资源，并初始化了数据，但还没有进入运行状态。

例如：
```java
Thread thread = new Thread(); // 此时线程处于NEW状态
```

#### RUNNABLE（可运行状态）
线程调用了start()方法之后进入此状态。此状态包含两个子状态：
- 就绪状态：线程已经准备好运行，只等待系统分配CPU时间片
- 运行状态：线程获得了CPU时间片，正在执行run()方法中的代码

例如：
```java
thread.start(); // 调用start()后，线程进入RUNNABLE状态
```

#### BLOCKED（阻塞状态）
线程试图获取对象锁时被阻塞。当一个线程试图进入一个同步块或同步方法时，如果该同步锁被其他线程持有，那么该线程就会进入BLOCKED状态。

#### WAITING（等待状态）
线程无限期等待另一个线程执行特定操作。进入此状态的方法有：
- Object.wait()
- Thread.join()

#### TIMED_WAITING（限时等待状态）
线程等待另一个线程执行操作，但最多只等待指定时间。进入此状态的方法有：
- Thread.sleep(long)
- Object.wait(long)
- Thread.join(long)

#### TERMINATED（终止状态）
线程执行完毕或出现异常退出。线程一旦进入终止状态，就不能再转换为其他状态。

### 4.3 状态转换及触发方法

| 转换方向 | 触发方法 | 说明 |
|---------|----------|------|
| NEW → RUNNABLE | start() | 启动线程 |
| RUNNABLE → BLOCKED | 进入同步块/方法 | 线程试图获取已被占用的对象锁 |
| RUNNABLE → WAITING | wait(), join() | 线程无限期等待 |
| RUNNABLE → TIMED_WAITING | sleep(), wait(timeout), join(timeout) | 线程限时等待 |
| BLOCKED/WAITING/TIMED_WAITING → RUNNABLE | notify()/notifyAll()/时间到期 | 阻塞解除，回到就绪状态 |
| RUNNABLE → TERMINATED | run()方法执行完毕或异常 | 线程正常或异常结束 |

---

## 5. 线程安全

### 5.1 线程安全问题

**问题：访问同一资源的异步操作导致资源被超额访问/同一时间被访问**

当多个线程同时访问和修改共享资源时，可能会出现以下问题：
1. **原子性问题**：多个操作在执行过程中被其他线程打断
2. **可见性问题**：一个线程对共享变量的修改，其他线程不可见
3. **有序性问题**：编译器和处理器对指令重排序导致程序执行顺序不一致

### 5.2 解决线程安全问题的方法

#### 1. synchronized关键字+{同步代码块}
使用synchronized关键字可以保证同一时间只有一个线程执行同步代码块或方法。

> *如果线程1，2，3都是线程t的实例，当1进入同步代码块时，线程2，3会等待，直到线程1执行完同步代码。*

```java
static Object 锁对象 = new Object();//必须是唯一的，也就是static

public void method() {
    synchronized(锁对象) {
        // 同步代码块
    }
}
```

>*常用的锁对象：线程类.class 字节码对象是唯一的。*
>*synchronized作用在方法上时对应的锁对象：*    
> `public synchronized [static] 返回类型 方法名(参数列表){}`
> - *静态方法： 类名.class*
> - *非静态方法：this*


#### 2. volatile关键字
使用volatile关键字可以保证变量的可见性，但不能保证原子性。

```java
private volatile boolean flag = false;
```

#### 3. Lock接口
使用Lock接口可以更灵活地控制锁的获取和释放。

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 临界区代码
} finally {
    lock.unlock();
}
```

#### 4. ThreadLocal类
使用ThreadLocal为每个线程提供独立的变量副本。

```java
private ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
```


### 5.3 线程同步机制

#### synchronized同步机制
- **同步方法**：使用synchronized修饰方法
- **同步代码块**：使用synchronized修饰代码块

#### Lock同步机制
- **ReentrantLock**：可重入锁
- **ReentrantReadWriteLock**：读写锁
- **StampedLock**：戳锁（Java 8新增）

#### 并发工具类
- **CountDownLatch**：倒计时门闩
- **CyclicBarrier**：循环屏障
- **Semaphore**：信号量
- **Exchanger**：交换器

---

## 6. 生产者消费者问题

生产者消费者问题是多线程编程中最经典的问题之一，描述了两类线程共享固定大小缓冲区的情形：
- **生产者线程**：生成数据放入缓冲区
- **消费者线程**：从缓冲区取出数据消费

### 6.1 问题特点

1. **同步关系**：
   - 生产者生产满时不能继续生产（等待）
   - 消费者没有数据时不能继续消费（等待）

2. **互斥关系**：
   - 生产者和消费者对缓冲区的访问是互斥的

### 6.2 常用线程唤醒和等待方法

Java中提供了几个重要的线程通信方法，常用于解决生产者消费者问题：

| 方法           | 说明                                      |
|--------------|-----------------------------------------|
| wait()       | 当前线程等待并释放锁，直到其他线程调用notify()或notifyAll() |
| notify()     | 唤醒在此对象监视器上等待的单个线程                       |
| notifyAll()  | 唤醒在此对象监视器上等待的所有线程                       |
| sleep(long)  | 使当前线程休眠指定时间，但不释放锁                       |

> 注意：这些方法都必须在同步代码块(synchronized)中使用，否则会抛出IllegalMonitorStateException异常。

### 6.3 生产者消费者代码示例

下面是一个经典的生产者消费者模型实现：

```java
import java.util.LinkedList;
import java.util.Queue;

// 共享资源类
class Buffer {
    private Queue<Integer> queue = new LinkedList<>();
    private int capacity;
    
    public Buffer(int capacity) {
        this.capacity = capacity;
    }
    
    // 生产方法
    public synchronized void produce(int item) throws InterruptedException {
        // 如果缓冲区满了，则等待
        while (queue.size() == capacity) {
            System.out.println("缓冲区已满，生产者等待...");
            wait(); // 等待并释放锁
        }
        
        queue.add(item);
        System.out.println("生产者生产了: " + item + "，当前缓冲区大小: " + queue.size());
        
        // 唤醒所有等待的消费者线程
        notifyAll();
    }
    
    // 消费方法
    public synchronized int consume() throws InterruptedException {
        // 如果缓冲区为空，则等待
        while (queue.isEmpty()) {
            System.out.println("缓冲区为空，消费者等待...");
            wait(); // 等待并释放锁
        }
        
        int item = queue.poll();
        System.out.println("消费者消费了: " + item + "，当前缓冲区大小: " + queue.size());
        
        // 唤醒所有等待的生产者线程
        notifyAll();
        return item;
    }
}

// 生产者线程
class Producer extends Thread {
    private Buffer buffer;
    
    public Producer(Buffer buffer) {
        this.buffer = buffer;
    }
    
    @Override
    public void run() {
        try {
            for (int i = 1; i <= 5; i++) {
                buffer.produce(i); // 生产物品
                Thread.sleep(1000); // 模拟生产时间
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

// 消费者线程
class Consumer extends Thread {
    private Buffer buffer;
    
    public Consumer(Buffer buffer) {
        this.buffer = buffer;
    }
    
    @Override
    public void run() {
        try {
            for (int i = 1; i <= 5; i++) {
                buffer.consume(); // 消费物品
                Thread.sleep(1500); // 模拟消费时间
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

// 主类
public class ProducerConsumerDemo {
    public static void main(String[] args) {
        Buffer buffer = new Buffer(3); // 创建容量为3的缓冲区
        
        Producer producer = new Producer(buffer);
        Consumer consumer = new Consumer(buffer);
        
        producer.start(); // 启动生产者线程
        consumer.start(); // 启动消费者线程
    }
}
```

在这个示例中：
1. **Buffer类**：代表共享的缓冲区，包含生产和消费方法
2. **produce方法**：生产者向缓冲区添加数据，缓冲区满时等待
3. **consume方法**：消费者从缓冲区取出数据，缓冲区空时等待
4. **wait()和notifyAll()**：实现线程间协调，确保缓冲区状态正确

### 6.4 使用Lock和Condition实现

除了传统的synchronized + wait/notify方式，还可以使用Lock和Condition来实现更精确的线程控制：

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class BufferWithLock {
    private Queue<Integer> queue = new LinkedList<>();
    private int capacity;
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();  // 缓冲区未满条件
    private final Condition notEmpty = lock.newCondition(); // 缓冲区非空条件
    
    public BufferWithLock(int capacity) {
        this.capacity = capacity;
    }
    
    public void produce(int item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                System.out.println("缓冲区已满，生产者等待...");
                notFull.await(); // 等待缓冲区未满的条件
            }
            
            queue.add(item);
            System.out.println("生产者生产了: " + item + "，当前缓冲区大小: " + queue.size());
            
            notEmpty.signal(); // 唤醒等待非空条件的消费者
        } finally {
            lock.unlock();
        }
    }
    
    public int consume() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                System.out.println("缓冲区为空，消费者等待...");
                notEmpty.await(); // 等待缓冲区非空的条件
            }
            
            int item = queue.poll();
            System.out.println("消费者消费了: " + item + "，当前缓冲区大小: " + queue.size());
            
            notFull.signal(); // 唤醒等待未满条件的生产者
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

使用Lock和Condition相比传统方式的优点：
1. **更精确的控制**：可以针对不同条件创建不同的Condition对象
2. **避免虚假唤醒**：使用while循环检查条件，避免因虚假唤醒导致的问题
3. **更好的可读性**：命名的Condition使代码意图更加清晰

---

## 7. 线程池

### 7.1 为什么使用线程池

1. **降低资源消耗**：通过重复利用已创建的线程降低线程创建和销毁造成的消耗
2. **提高响应速度**：任务到达时，无需等待线程创建就能立即执行
3. **提高线程的可管理性**：线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配、调优和监控

### 7.2 ThreadPoolExecutor

ThreadPoolExecutor是线程池的核心实现类，其构造方法参数包括：
- corePoolSize：核心线程数
- maximumPoolSize：最大线程数
- keepAliveTime：空闲线程存活时间
- unit：时间单位
- workQueue：任务队列
- threadFactory：线程工厂
- handler：拒绝策略

### 7.3 线程池的使用

```java
package threadPool;

import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        /**
         * 创建一个线程池,参数：
         * 核心线程数：5；
         * 最大线程数：10（要大于核心线程数）；
         * 线程空闲时间值：1000；
         * 空闲时间单位：TimeUnit.SECONDS；
         * 任务队列（不指定大小时（）内不填参数）：ArrayBlockingQueue<Runnable>(5)；
         * 线程工厂：Executors.defaultThreadFactory()；
         * 拒绝策略：ThreadPoolExecutor.AbortPolicy()
         */
        ThreadPoolExecutor pool = new ThreadPoolExecutor(
                5,
                10,
                1000,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(5),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );

        pool.execute(new MyRunnable());
        pool.execute(new MyRunnable());

        pool.shutdown();
    }
}
```

```java
package threadPool;

public class MyRunnable implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "正在执行任务");
    }
}
```


---

## 8. 实际应用示例

### 8.1 示例代码分析

我们来看一下项目中的实际代码示例：

```java
public class Mythread extends Thread{
    private String name;

    @Override
    public void run() {
        if(name != null) {
            for(int i = 0; i < 10; i++) {
                try {
                    sleep(500);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println(name);
            }
        }
    }

    public Mythread(String name){
        this.name = name;
    }

    public Mythread(){}
}
```

这段代码演示了一个简单的线程类，继承自Thread类，并重写了run()方法。在run()方法中，线程会循环10次，每次睡眠500毫秒后打印线程名称。

### 8.2 线程启动示例

```java
public class main {
    public static void main(String[] args) {
        System.out.println("Main thread");

        Mythread mt1 = new Mythread("mt1");
        Mythread mt2 = new Mythread("mt2");
        mt1.start();
        mt2.start();
    }
}
```

在主函数中，创建了两个线程实例，并分别调用start()方法启动它们。这两个线程会并发执行，交替打印自己的名称。

### 8.3 生命周期跟踪

让我们分析一下上述示例中线程的生命周期：

1. **创建阶段**：
   ```java
   Mythread mt1 = new Mythread("mt1"); // mt1处于NEW状态
   Mythread mt2 = new Mythread("mt2"); // mt2处于NEW状态
   ```

2. **启动阶段**：
   ```java
   mt1.start(); // mt1调用start()，进入RUNNABLE状态
   mt2.start(); // mt2调用start()，进入RUNNABLE状态
   ```

3. **运行阶段**：
   当线程获得CPU时间片时，开始执行run()方法，进入运行状态。

4. **阻塞阶段**：
   在run()方法中调用sleep(500)时，线程进入TIMED_WAITING状态，持续500毫秒。

5. **终止阶段**：
   当循环执行完毕后，run()方法执行结束，线程进入TERMINATED状态。

### 8.4 使用Lock解决线程安全问题示例

在我们的售票系统示例中，我们使用了synchronized关键字来保证线程安全。下面我们来看看如何使用Lock接口来实现同样的功能：

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class TicketThread extends Thread{
    private static int ticket = 1;
    private static final Lock lock = new ReentrantLock(); // 创建Lock对象

    @Override
    public void run(){
        while (true) {
            try {
                if (!saleTicket()) break;
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }

    private boolean saleTicket() throws InterruptedException {
        lock.lock(); // 获取锁
        try {
            if(ticket <= 100){
                System.out.println(Thread.currentThread().getName() + "正在出售第" + ticket + "张票");
                ticket ++;
                return true;
            }
            else return false;
        } finally {
            lock.unlock(); // 释放锁
        }
    }
}
```

使用Lock接口相比于synchronized关键字的优势：
1. **更灵活的锁定操作**：可以尝试获取锁、可中断地获取锁、超时获取锁等
2. **支持公平锁**：可以指定锁的获取策略
3. **更好的性能**：在高竞争环境下，Lock通常比synchronized有更好的性能表现

在使用Lock时需要注意：
1. 必须手动释放锁，通常在finally块中释放以确保锁一定会被释放
2. Lock.lock()和Lock.unlock()必须成对出现
3. 如果忘记释放锁，会导致其他线程永远无法获取锁，造成死锁

### 8.5作业1：交替打印0-100内的奇数：

在homework3_alternatePrint包中，我们实现了两个线程交替打印奇数的示例。

**NumThread.java：**
```java
package homework3_alternatePrint;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class NumThread extends Thread{
    final static int MAX = 100;
    static int count = 1;
    static Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while (printNum()){};
    }

    private boolean printNum() {
        try {
            Thread.sleep(100);
            lock.lock();
            if(count >= MAX){
                return false;
            }

            System.out.println(getName() + ": " + count);
            count+= 2;
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }

        return true;
    }
}
```

**Main.java：**
```java
package homework3_alternatePrint;

public class Main {
    public static void main(String[] args) {
        NumThread t1 = new NumThread();
        NumThread t2 = new NumThread();

        t1.setName("线程1");
        t2.setName("线程2");

        t1.start();
        t2.start();
    }
}
```

在这个示例中：
1. 两个线程共享一个静态变量`count`和一个静态锁`lock`
2. 每个线程在执行时都会获取锁，打印当前的奇数值，然后将count增加2
3. 通过使用`Thread.sleep(100)`来控制执行节奏，让线程交替执行更加明显
4. 当count达到MAX值时，线程停止执行

### 8.6 作业2：抢红包：

在homework4_grepRedEnv包中，我们实现了一个多线程抢红包的示例。

**RedEnv.java：**

```java
package homework4_grepRedEnv;

public class RedEnv {
    private double money;
    private int count;

    RedEnv(double money, int count) {
        this.money = money;
        this.count = count;
    }

    public double getMoney() {
        return money;
    }

    public int getCount() {
        return count;
    }

    public synchronized double getRandomMoney() {
        // 如果红包已经抢完
        if (count == 0) {
            return 0.0;
        }

        // 如果是最后一个红包，直接返回剩余所有金额
        if (count == 1) {
            count--;
            double result = Math.round(money * 100.0) / 100.0; // 保留两位小数
            money = 0;
            return result;
        }

        // 获取一个小于当前金额的随机值
        double max = money;
        double randomAmount = Math.random() * max;  // 生成0到max之间的随机数

        // 确保至少分到0.01元
        randomAmount = Math.max(randomAmount, 0.01);

        // 保留两位小数（乘100右移小数点2位，四舍五入取整后再除100得到两位小数）
        randomAmount = Math.round(randomAmount * 100.0) / 100.0;

        // 确保不会超出总额度
        if (randomAmount > money) {
            randomAmount = money;
        }

        // 从总金额中扣除已分配的金额
        money = Math.round((money - randomAmount) * 100.0) / 100.0;
        count--;

        return randomAmount;
    }
}
```

**PeopleThread.java：**

```java
package homework4_grepRedEnv;

import java.util.concurrent.locks.ReentrantLock;

public class PeopleThread extends Thread {
    private RedEnv redEnv;

    private static ReentrantLock lock = new ReentrantLock();

    public PeopleThread(RedEnv redEnv) {
        this.redEnv = redEnv;
    }

    @Override
    public void run() {
        while (redEnv.getCount() != 0) {
            getMoney();
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void getMoney() {
        try {
            lock.lock();
            if (redEnv.getCount() == 0) {
                System.out.println(Thread.currentThread().getName() + ": 手慢啦！");
                return;
            }

            System.out.println(Thread.currentThread().getName() + ": 抢到 " + String.format("%.2f", redEnv.getRandomMoney()) + " 元");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

**Main.java：**

```java
package homework4_grepRedEnv;

public class Main {
    public static void main(String[] args) {
        // 创建一个100元的红包，分为10份
        RedEnv redEnv = new RedEnv(100, 3);

        // 创建5个人抢红包
        PeopleThread person1 = new PeopleThread(redEnv);
        PeopleThread person2 = new PeopleThread(redEnv);
        PeopleThread person3 = new PeopleThread(redEnv);
        PeopleThread person4 = new PeopleThread(redEnv);
        PeopleThread person5 = new PeopleThread(redEnv);

        // 设置线程名称
        person1.setName("张三");
        person2.setName("李四");
        person3.setName("王五");
        person4.setName("赵六");
        person5.setName("钱七");

        // 启动线程
        person1.start();
        person2.start();
        person3.start();
        person4.start();
        person5.start();
    }

}
```

在这个示例中：
1. **RedEnv类**：代表红包对象，包含总金额和红包个数，提供获取随机金额的方法
2. **PeopleThread类**：代表抢红包的人，每个线程代表一个人
3. **线程安全**：通过使用`synchronized`关键字和`ReentrantLock`来保证线程安全
4. **随机红包算法**：实现了红包金额的随机分配，确保最后一个红包分配剩余所有金额
5. **浮点数处理**：使用`Math.round(value * 100.0) / 100.0`来保证金额精确到分

---

### 8.7作业3：奖池多奖项抽奖：

在homework5_lottery包中，我们实现了一个多线程抽奖的示例。

**Jackpot.java：**

```java
package homework5_lottery;

import java.util.ArrayList;
import java.util.List;

public class Jackpot {
    private List<Double> moneyList = new ArrayList<>();

    Jackpot(){};

    Jackpot(List<Double>  moneyList){
        this.moneyList = moneyList;
    }

    public List<Double> getMoneyList() {
        return moneyList;
    }


    public double lottery() {
        int random = (int)(Math.random() * moneyList.size());
        if(random >= moneyList.size() || random < 0){
            random = 0;
        }

        double money = moneyList.get(random);
        moneyList.remove(random);

        return money;
    }
}
```

**LotteryThread.java：**

```java
package homework5_lottery;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LotteryThread extends Thread {
    private Jackpot jackpot;
    private static Lock lock = new ReentrantLock();
    private List<Double> moneyList = new ArrayList<>();
    private static double MAX;

    public LotteryThread(Jackpot jackpot, double max) {
        this.jackpot = jackpot;

        MAX = max;
    }

    @Override
    public void run() {
        doLottery();

        System.out.println(this.getName() + "总共产生的奖项：" + moneyList);
    }

    /**
     * 循环,
     * 同步代码块,
     * 判断,
     * 判断,
     */
    private void doLottery() {
        while (true) {
            //try中无论是抛出异常或者正常结束，都会执行finally中的代码
            //若外层循环在try之外，则continue，break也会触发finally中代码
            try {
                lock.lock();

                if (jackpot.getMoneyList().isEmpty()) {
                    //break后跳出try，会触发finally中的代码，再跳出while循环
                    break;
                } else {
                    double money = jackpot.lottery();
                    moneyList.add(money);
                    System.out.println(this.getName() + "产生大奖：" + money);
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }

    }

    public List<Double> getMoneyList() {
        return moneyList;
    }
}
```

**Main.java：**

```java
package homework5_lottery;

import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<Double> moneyList = new ArrayList<>();
        //生成一个容量为100，数据范围在0到1000的整数的Double类型列表
        for (int i = 0; i < 100; i++) {
            moneyList.add(Math.round(Math.random() * 1000 * 100.0) / 100.0);
        }

        System.out.println("奖池初始化完成");

        Jackpot jackpot = new Jackpot(moneyList);

        double max = jackpot.getMoneyList().get(0);
        for(int i = 0; i < jackpot.getMoneyList().size(); i++){
            if(max < jackpot.getMoneyList().get(i)) {
                max = jackpot.getMoneyList().get(i);
            }
        }

        LotteryThread t1 = new LotteryThread(jackpot,max);
        LotteryThread t2 = new LotteryThread(jackpot,max);

        t1.setName("抽奖箱1");
        t2.setName("抽奖箱2");

        t1.start();
        t2.start();

        try {
            // 等待两个线程执行完毕
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("所有奖项已全部产生完毕");
        List<Double> moneyList1 = t1.getMoneyList();
        List<Double> moneyList2 = t2.getMoneyList();

        if(moneyList1.contains(max)){
            System.out.println("抽奖箱1出了最终大奖！" + max);
        }
        else{
            System.out.println("抽奖箱2出了最终大奖！" + max);
        }
    }
}
```

在这个示例中：
1. **Jackpot类**：代表奖池对象，包含奖金列表，提供抽奖方法
2. **LotteryThread类**：代表抽奖箱，每个线程代表一个抽奖箱
3. **线程安全**：通过使用`ReentrantLock`来保证线程安全，确保奖池中的奖项不会被重复抽取
4. **join()方法的应用**：主线程使用join()方法等待两个抽奖线程完全执行完毕后，再进行最终结果的统计和显示
5. **资源同步**：通过锁机制保证了对共享资源（奖池）的安全访问

## 9. 总结

Java多线程编程是Java开发中的重要知识点，掌握线程的生命周期对于编写高质量的并发程序至关重要。通过合理使用线程的各种状态和转换方法，我们可以更好地控制程序的执行流程，提高程序的性能和响应性。

在实际开发中，需要注意线程安全问题，合理使用同步机制，避免出现死锁、活锁等问题。同时，使用线程池可以更好地管理线程资源，提高系统性能和稳定性。

多线程编程需要深入理解线程状态、同步机制、线程池等核心概念，并在实践中不断积累经验，才能编写出高效、稳定的并发程序。