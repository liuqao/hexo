---
title: JUC多线程
date: 2020-03-29 11:41:00
tags: ['知识点','java']
categories: ['学习']
---
#### 基础
##### 线程与进程
- **进程**：有独立的内存空间，进程中的数据存放空间(栈空间和堆空间)是独立的，至少有一个线程
- **线程**：堆空间是共享的，栈空间是独立的，线程消耗的资源比进程小得多
- *扩展*：java程序的进程至少包含两个线程，主线程也就是main()方法的线程，另外一个是垃圾回收机制线程
<!--more-->
##### 多线程的创建
1. 继承Thread
```java
public class ThreadDemo3 extends Thread {
    public static void main(String[] args) {
        new ThreadDemo3().start();
    }

    @Override
    public void run() {
        System.out.println("Thread begin");
    }
}

```
2. 实现Runnable接口
```java
public class ThreadDemo3 {
    public static void main(String[] args) {
        new Thread(new CreateRunnable()).start();
    }
}
class CreateRunnable implements Runnable {
    public void run() {
        System.out.println("Thread begin");
    }
}
```
> - 优势
> 1. 适合多个相同的程序代码的线程去共享同一个资源。
> 2. 可以避免java中的单继承的局限性。
> 3. 增加程序的健壮性，实现解耦操作，代码可以被多个线程共享
> 4. 线程池只能放入实现Runable或callable类线程

3. 实现Callable<T>接口，带返回值的线程
```java
public class ThreadDemo3 implements Callable<String> {
    public String call() throws Exception {
        return "This is thread";
    }
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> task = new FutureTask<String>(new ThreadDemo3());
        Thread thread = new Thread(task);
        thread.start();
        System.out.println("线程结果:" + task.get());
    }
}
```
##### 线程安全
1. **问题**：线程安全问题都是由全局变量及静态变量引起的。若有多个线程同时执行写操作，一般都需要考虑线程同步，否则的话就可能影响线程安全。
2. **线程同步**：
- 同步代码块:
```java
Object lock = new Object(); //创建锁
synchronized(lock){
//可能会产生线程安全问题的代码
}
```
- 同步方法:
```java
//同步方法
public synchronized void method(){
    //可能会产生线程安全问题的代码
}
```
- Lock锁:
```java
Lock lock = new ReentrantLock();
lock.lock();//需要同步操作的代码
lock.unlock();
```
- 同步代码块与同步方法的区别
    - 同步代码块锁的对象可以指定
    - 同步方法锁的对象为线程类字节码文件
##### 线程状态
1. **NEW**      新建
2. **RUNNABLE**     可运行
3. **BLOCKED**      锁阻塞
4. **WAITING**      无限等待
5. **TIMED_WAITING**        计时等待
6. **TERMINATED**       被终止
- **API** 
1. **wait**  等待
2. **notify**  唤醒     notifyAll  唤醒所有
3. **sleep**    睡眠，睡眠过程不会释放锁
4. **interrupt**    终止线程
5. **setPriority**  设置优先级
    - 默认5，存在子类线程继承特性
6. **join** 等待线程死亡，当前线程等待调用线程执行完再执行
7. **yield**    资源让步
8. **setDaemon**    设置守护线程，守护线程当进程不存在或主线程停止，守护线程也会被停止。
---
#### 高级
##### 并发的三个特性
1. **原子性**：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要
么就都不执行
2. **可见性**：当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即
看得到修改的值
3. **有序性**：程序执行的顺序按照代码的先后顺序执行
    - ***注***：一般来说，处理器为了提高程序运行效率，可能会对输入代码进
行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致。
##### Java内存模型

```
graph LR
A[PC寄存器]-->B[JVM栈]
B-->C[本地方法栈]
C---各个线程独享的数据区域
D[JVM堆]-->E[方法区]
E---所有线程共享的数据区域
```
##### Java对象模型
- Java是一种面向对象的语言，而Java对象在JVM中的存储也是有一定的结构的。而这个关于Java对象自身的存储模型称之为Java对象模型。
- 每一个Java类，在被JVM加载的时候，JVM会给这个类创建一个 instanceKlass
对象，保存在方法区，用来在JVM层表示该Java类。当我们在Java代码中，使用new
创建一个对象的时候，JVM会创建一个 instanceOopDesc 对象，这个对象中包含了
对象头以及实例数据。
##### Java内存模型：一种符合内存模型规范的机制及规范
**第一条**：关于线程与主内存：线程对共享变量的所有操作都必须在自己的工作内存
（本地内存）中进行，不能直接从主内存中读写  
**第二条**：关于线程间本地内存：不同线程之间无法直接访问其他线程本地内存中的变
量，线程间变量值的传递需要经过主内存来完成。
##### synchronized
**概念**: 同一时刻只有一
个线程执行，这部分代码块的重排序也不会影响其执行结果。也就是说使用了synchronized可以保证并发的原子性，可见性，有序性。
###### 解决可见性问题
> - JMM关于synchronized的两条规定：  
> -  线程解锁前（退出同步代码块时）：必须把自己工作内存中共享变量的最新值刷新到主内存中  
> - 线程加锁时（进入同步代码块时）：将清空本地内存中共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值（加锁与解锁是同一把锁）
- synchronized实现可见性的过程
    1. 获取互斥锁(同步获取锁)
    2. 清空本地内存
    3. 从本地内存拷贝变量的最新副本到本地内存
    4. 执行代码
    5. 将更改后的共享变量的值刷新到主内存
    6. 释放互斥锁
###### 同步原理
> - synchronized的同步操作主要是monitorenter和monitorexit这两个jvm指令实现
##### Volatile
- 如果某个线程对**volatile**修饰的共享变量进行更新，那么其他
线程可以立马看到这个更新，这就是内存可见性。
1. 线程写Volatile变量的过程：
    1. 改变线程本地内存中Volatile变量副本的值；
    2. 将改变后的副本的值从本地内存刷新到主内存
2. 线程读Volatile变量的过程：
    1. 从主内存中读取Volatile变量的最新值到线程的本地内存中
    2. 从本地内存中读取Volatile变量的副本
###### Volatile实现内存可见性原理：
1. 写操作时：通过在写操作指令后加入一条store屏障指令，让本地内存中变量的值能够刷新到主内存中
2. 读操作时：通过在读操作前加入一条load屏障指令，及时读取到变量在主内存的值
>PS: 内存屏障（Memory Barrier）是一种CPU指令，用于控制特定条件下的重排序和内存可见性问题。Java编译器也会根据内存屏障的规则禁止重排序
- StoreLoad屏障的作用是避免volatile写与后面可能有的volatile读/写操作重排
序。
###### 原子性问题
- 虽然Volatile关键字可以让变量在多个线程之间可见，但是Volatile不具备原子性。
- 解决方案：
    1. 使用synchronized
    2. 使用ReentrantLock（可重入锁）
    3. 使用AtomicInteger（原子操作）
###### Volatile 适合使用场景
- 变量真正独立于其他变量和自己以前的值，在单独使用的时候，适合用volatile
###### synchronized和volatile比较
1. volatile不需要加锁，比synchroized更轻便，不会阻塞线程
2. synchronized能保证原子性、可见性，而volatile只能保证可见性，不能保证原子性
#### JUC  java.util.concurrent
> - 包含内存
> 1. AbstractQueuedSynchronizer（AQS框架），J.U.C 中实现锁和同步机制的基
> 2. Locks & Condition（锁和条件变量），比 synchronized、wait、notify 更细粒度的锁机制；
> 3. Executor 框架（线程池、Callable、Future），任务的执行和调度框架；
> 4. Synchronizers（同步器），主要用于协助线程同步，有 CountDownLatch、CyclicBarrier、Semaphore、Exchanger；
> 5. Atomic Variables（原子变量），方便程序员在多线程环境下，无锁的进行原子操作，核心操作是 CAS 原子操作，所谓的 CAS 操作，即 compare and swap当前变量，否则不作操作；
> 6. BlockingQueue（阻塞队列），阻塞队列提供了可阻塞的入队和出对操作，如果队列满了，入队操作将阻塞直到有空间可用，如果队列空了，出队操作将阻塞直到有元素可用；
> 7. Concurrent Collections（并发容器），说到并发容器，不得不提同步容器。在JDK1.5 之前，为了线程安全，我们一般都是使用同步容器，同步容器主要的缺点是：对所有容器状态的访问都串行化，严重降低了并发性；某些复合操作，仍然需要加锁来保护；迭代期间，若其它线程并发修改该容器，会抛出ConcurrentModificationException 异常，即快速失败机制；
> 8. Fork/Join 并行计算框架，这块内容是在 JDK1.7 中引入的，可以方便利用多核平台的计算能力，简化并行程序的编写，开发人员仅需关注如何划分任务和组合中间结果；
> 9. TimeUnit 枚举，TimeUnit 是 java.util.concurrent 包下面的一个枚举类，TimeUnit 提供了可读性更好的线程暂停操作，以及方便的时间单位转换方法；
##### CAS
- **介绍**：CAS，Compare And Swap，即比较并交换。同步组件中大量使用CAS技术实现了Java多线程的并发操作。整个AQS同步组件、Atomic原子类操作等等都是以CAS实现的，甚至ConcurrentHashMap在1.8的版本中也调整为了CAS+Synchronized。可以说CAS是整个JUC的基石。
###### synchronized同步分析
- 相比CAS慢的原因：
1. monitorenter和monitorexit这两个jvm指令实现锁的使用，主要是基于 MarkWord和、monitor。
    -  Mark Word用于存储对象自身的运行时数据，它是synchronized实现轻量级锁和偏向锁的关键。
    -  Monitor 是线程私有的数据结构，每一个线程都有一个可用monitor record列表，同时还有一个全局的可用列表。
    -  同时monitor中有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。
2. 使用锁底层会用到很多CPU操作
###### CAS原理
- **思想**：三个参数，一个当前内存值V，旧的预期值A，即将更新的值B，当且仅当旧的预期值A与内存值相同时，将内存值修改为B并返回ture，否则什么不做返回false。如果CAS操作失败，通过自旋方式等待并再次尝试，直到成功。
- **先比较后修改**并没有获取锁，释放锁的过程，用的资源少
- 使用的其他语言实现比较这些
###### 多CPU的CAS处理
- CPU提供了两种方法来实现多处理器的原子
操作：总线加锁或者缓存加锁。
 1. **总线加锁**：使用处理器提供的一个LOCK#信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住,那么该处理器可以独占使用共享内存。
 2. **缓存加锁**：其实针对于上面那种情况我们只需要保证在同一时刻对某个内存地址的操作是原子性的即可，处理器不在输出LOCK#信号，而是修改内部的内存地址，利用缓存一致性协议来保证原子性。缓存一致性机制可以保证**同一个内存区域的数据仅能被一个处理器修改。** ***原地址失效重新获取***
 ###### CAS缺陷
1. 循环时间太长：所以JUC有些地方就限制了CAS自旋的次数。
2. 只能保证一个共享变量的原子操作：如果是多个共享变量就只能使用锁了
3. ABA问题：解决方案是每个变量都加上一个版本号。
    - A->B->A，变为 1A->2B->3A
    - java提供了AtomicStampedReference来解决
##### Atomic包
- AtomicInteger在进行一些原子操作的时候，依赖Unsafe类里面的CAS方法，原子操作就是通过自旋方式，不断地使用CAS函数进行尝试直到达到自己的目的。
###### 基本类型
- AtomicInteger：整形原子类
- AtomicLong：长整型原子类 
- AtomicBoolean ：布尔型原子类
###### 引用类型
- AtomicReference：引用类型原子类
- AtomicStampedReference：原子更新引用类型里的字段原子类
- AtomicMarkableReference ：原子更新带有标记位的引用类型，解决ABA，区别不是用版本号，而是用true，false
###### 数组类型
- AtomicIntegerArray：整形数组原子类
- AtomicLongArray：长整形数组原子类 
- AtomicReferenceArray ：引用类型数组原子类
###### 对象的属性修改类型
- AtomicIntegerFieldUpdater:原子更新整形字段的更新器
- AtomicLongFieldUpdater：原子更新长整形字段的更新器
- AtomicReferenceFieldUpdater ：原子更新引用类形字段的更新器
- 
- 使用限制
1. 目标不能是static类型
2. 目标不能是final类型
3. 必须是volatile类型的数据，也就是数据本身是读一致
4. 属性必须是当前的updater的所在区域可见的，不可见private等

```java
public class AtomicIntegerFieldUpdaterTest {
    public static void main(String[] args) {
        AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");
        User user = new User("Java", 22);
        System.out.println(a.get(user));
        System.out.println(a.getAndAdd(user,10));
        System.out.println(a.get(user));
    }
}
```
###### JDK1.8新增类
- DoubleAdder：双浮点型原子类
- LongAdder：长整型原子类
- DoubleAccumulator：类似DoubleAdder，但要更加灵活(要传入一个函数式接口)
- LongAccumulator：类似LongAdder，但要更加灵活(要传入一个函数式接口)
- 在高并发下N多线程同时去操作一个变量会造成大量线程CAS失败然后处于自旋状态，这大大浪费了cpu资源，降低了并发性。
- 既然AtomicLong性能由于过多线程同时去竞争一个变量的更新而降低的，那么如果把一个变量分解为多个变量，让同样多的线程去竞争多个资源那么性能问题不就解决了？
- 在并发比较低的时候，LongAdder和AtomicLong的效果非常接近。但是当并发较高时，两者的差距会越来越大。上图中在线程数为1000，每个线程循环数为100000时，LongAdder的效率是AtomicLong的6倍左右。
##### AQS队列同步器
###### 作用
- AQS解决了实现同步器时涉及到的大量细节问题，例如获取同步状态、FIFO同步队列。基于AQS来构建同步器可以带来很多好处。
###### state状态
> - AQS维护了一个volatile int类型的变量state表示当前同步状态。当state>0时表示已经获取了锁，当state = 0时表示释放了锁。
- 它提供了三个方法来对同步状态state进行操作：
    - getState()：返回同步状态的当前值
    - setState()：设置当前同步状态
    - compareAndSetState()：使用CAS设置当前状态，该方法能够保证状态设置的原子性
- 这三种操作均是CAS原子操作，其中compareAndSetState的实现依赖于Unsafe的compareAndSwapInt()方法
###### 资源共享方式
1. Exclusive（独占，只有一个线程能执行，如ReentrantLock）
2. Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）
###### CLH同步队列
- AQS内部维护着一个FIFO队列，该队列就是CLH同步队列，遵循FIFO原则（ First Input First Output先进先出）。CLH同步队列是一个FIFO双向队列，AQS依赖它来完成同步状态的管理。
##### 锁
###### 互斥锁
- 在编程中，引入了对象互斥锁的概念，来保证共享数据操作的完整性。每个对象都对应于一个可称为" 互斥锁" 的标记，这个标记用来保证在任一时刻，只能有一个线程访问该对象。
###### 阻塞锁
- 阻塞锁，可以说是让线程进入阻塞状态进行等待，当获得相应的信号（唤醒，时间） 时，才可以进入线程的准备就绪状态，准备就绪状态的所有线程，通过竞争，进入运行状态。
###### 自旋锁
- 自旋锁只是将当前线程不停地执行循环体，不进行线程状态的改变，所以响应速度更快。但当线程数不停增加时，性能下降明显，因为每个线程都需要执行，占用CPU时间。如果线程竞争不激烈，并且保持锁的时间段。适合使用自旋锁。
###### 读写锁
- 读写锁相对于自旋锁而言，能提高并发性，因为在多处理器系统中，它允许同时有多个读者来访问共享资源，最大可能的读者数为实际的逻辑CPU数。写者是排他性的，一个读写锁同时只能有一个写者或多个读者（与CPU数相关），但不能同时既有读者又有写者。
1. 公平性：支持公平性和非公平性。
2. 重入性：支持重入。读写锁最多支持65535个递归写入锁和65535个递归读取锁。
3. 锁降级：写锁能够降级成为读锁，遵循获取写锁、获取读锁在释放写锁的次序。读锁不能升级为写锁。
4.  读写锁ReentrantReadWriteLock实现接口ReadWriteLock，该接口维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。
```java
/** 内部类  读锁 */
private final ReentrantReadWriteLock.ReadLock readerLock;
/** 内部类  写锁 */
private final ReentrantReadWriteLock.WriteLock writerLock;
final Sync sync;
/** 使用默认（非公平）的排序属性创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock() {
    this(false);
}
/** 使用给定的公平策略创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
/** 返回用于写入操作的锁 */
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
/** 返回用于读取操作的锁 */
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
```
- 所以读写锁采用“按位切割使用”的方式来维护这个变量，将其切分为两部分，高16为表示读，低16为表示写。
###### 公平锁
- 公平锁（Fair）：加锁前检查是否有排队等待的线程，优先排队等待的线程，先来先得
- 非公平锁（Nonfair）：加锁时不考虑排队等待问题，直接尝试获取锁，获取不到自动到队尾等待
- 非公平锁性能比公平锁高，因为公平锁需要在多核的情况下维护一个队列。
###### ReentrantLock*
- ReentrantLock，可重入锁，是一种递归无阻塞的同步机制。它可以等同于synchronized的使用，但是ReentrantLock提供了比synchronized更强大、灵活的锁机制，可以减少死锁发生的概率。
##### Condition
- Condition提供了一系列的方法来对阻塞和唤醒线程：
    1. await() ：造成当前线程在接到信号或被中断之前一直处于等待状态。
    2. await(long time, TimeUnit unit) ：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
    3. awaitNanos(long nanosTimeout) ：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。返回值表示剩余时间，如果在nanosTimesout之前唤醒，那么返回值 = nanosTimeout – 消耗时间，如果返回值 <= 0 ,则可以认定它已经超时了。
    4. awaitUninterruptibly() ：造成当前线程在接到信号之前一直处于等待状态。【注意：该方法对中断不敏感】。
    5. awaitUntil(Date deadline) ：造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回true，否则表示到了指定时间，返回返回false。
    6. signal()：唤醒一个等待线程。该线程从等待方法返回前必须获得与Condition相关的锁。
    7. 唤醒所有等待线程。能够从等待方法返回的线程必须获得与Condition相关的锁。
```java
public class Demo11Condition {
​
    private Lock reentrantLock = new ReentrantLock();
    private Condition condition1 = reentrantLock.newCondition();
    private Condition condition2 = reentrantLock.newCondition();
    public void m1() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入执行等待。。。");
            condition1.await();
            System.out.println("线程 " + Thread.currentThread().getName() + " 已被唤醒，继续执行。。。");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }
    public void m2() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入执行等待。。。");
            condition1.await();
            System.out.println("线程 " + Thread.currentThread().getName() + " 已被唤醒，继续执行。。。");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }
    public void m3() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入执行等待。。。");
            condition2.await();
            System.out.println("线程 " + Thread.currentThread().getName() + " 已被唤醒，继续执行。。。");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }
    public void m4() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入发出condition1唤醒信号。。。");
            condition1.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }
    public void m5() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入发出condition2唤醒信号。。。");
            condition2.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }
    public static void main(String[] args) throws Exception {
        final Demo11Condition useCondition = new Demo11Condition();
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                useCondition.m1();
            }
        }, "t1");
        Thread t2 = new Thread(new Runnable() {
            public void run() {
                useCondition.m2();
            }
        }, "t2");
        Thread t3 = new Thread(new Runnable() {
            public void run() {
                useCondition.m3();
            }
        }, "t3");
        Thread t4 = new Thread(new Runnable() {
            public void run() {
                useCondition.m4();
            }
        }, "t4");
        Thread t5 = new Thread(new Runnable() {
            public void run() {
                useCondition.m5();
            }
        }, "t5");
        t1.start();
        t2.start();
        t3.start();
        Thread.sleep(2000);
        t4.start();
        Thread.sleep(2000);
        t5.start();
    }
}
```
#### 并发工具类
##### CyclicBarrier
**概念**：同步屏障，允许一组线程全部等待彼此达到共同屏障点的同步辅助。 循环阻塞在涉及固定大小的线程方的程序中很有用，这些线程必须偶尔等待彼此。 屏障被称为循环，因为它可以在等待的线程被释放之后重新使用。
1. 构造方法：
    - CyclicBarrier(int parties)：它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动屏障时执行预定义的操作。parties表示拦截线程的数量。
    - CyclicBarrier(int parties, Runnable barrierAction) ：创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动屏障时执行给定的屏障操作，该操作由最后一个进入屏障的线程执行。
2. 案例：田径比赛：
```java
public class Demo1CyclicBarrier {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
        List<Thread> threadList = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            Thread t = new Thread(new Athlete(cyclicBarrier, "运动员" + i));
            threadList.add(t);
        }
        for (Thread t : threadList) {
            t.start();
        }
    }
    static class Athlete implements Runnable {
        private CyclicBarrier cyclicBarrier;
        private String name;
        public Athlete(CyclicBarrier cyclicBarrier, String name) {
            this.cyclicBarrier = cyclicBarrier;
            this.name = name;
        }
        @Override
        public void run() {
            System.out.println(name + "就位");
            try {
                cyclicBarrier.await();
                System.out.println(name + "跑到终点。");
            } catch (Exception e) {
            }
        }
    }
}
```
##### CountDownLatch
###### 概念
1. CountDownLatch是一个计数的闭锁，作用与CyclicBarrier有点儿相似。
2. 用给定的计数 初始化 CountDownLatch。由于调用了 countDown() 方法，所以在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await 的所有后续调用都将立即返回。
这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 CyclicBarrier。
- CountDownLatch：一个或者多个线程，等待其他多个线程完成某件事情之后才能执行；
- CyclicBarrier：多个线程互相等待，直到到达同一个同步点，再继续一起执行。
###### 构造方法
```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```
1. CountDownLatch提供await()方法来使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。内部使用AQS的getState方法获取计数器，如果计数器值不等于0，则会以自旋方式会尝试一直去获取同步状态。
2. CountDownLatch提供countDown() 方法递减锁存器的计数，如果计数到达零，则释放所有等待的线程。内部调用AQS的releaseShared(int arg)方法来释放共享锁同步状态。
###### 案例：接力赛跑
```java
public class Demo2CountDownLatch {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
        List<Thread> threadList = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            CountDownLatch countDownLatch = new CountDownLatch(1);
            //起点运动员
            Thread t1 = new Thread(new Athlete(cyclicBarrier, countDownLatch, "起点运动员" + i));
            //接力运动员
            Thread t2 = new Thread(new Athlete(countDownLatch, "接力运动员" + i));
            threadList.add(t1);
            threadList.add(t2);
        }
        for (Thread t : threadList) {
            t.start();
        }
    }
    static class Athlete implements Runnable {
        private CyclicBarrier cyclicBarrier;
        private String name;
        CountDownLatch countDownLatch;
        //起点运动员
        public Athlete(CyclicBarrier cyclicBarrier, CountDownLatch countDownLatch, String name) {
            this.cyclicBarrier = cyclicBarrier;
            this.countDownLatch = countDownLatch;
            this.name = name;
        }
        //接力运动员
        public Athlete(CountDownLatch countDownLatch, String name) {
            this.countDownLatch = countDownLatch;
            this.name = name;
        }
        @Override
        public void run() {
            //判断是否是起点运动员
            if (cyclicBarrier != null) {
                System.out.println(name + "就位");
                try {
                    cyclicBarrier.await();
                    System.out.println(name + "到达交接点。");
                    //已经到达交接点
                    countDownLatch.countDown();
                } catch (Exception e) {
                }
            }
            //判断是否是接力运动员
            if (cyclicBarrier == null) {
                System.out.println(name + "就位");
                try {
                    countDownLatch.await();
                    System.out.println(name + "到达终点。");
                } catch (Exception e) {
                }
            }
        }
    }
}
```
##### Semaphore
###### 介绍
1. Semaphore是一个控制访问多个共享资源的计数器，和CountDownLatch一样，其本质上是一个“共享锁”。
2. Semaphore常用于约束访问一些（物理或逻辑）资源的线程数量。
###### 构造方法
1. Semaphore(int permits) ：创建具有给定的许可数和非公平的 Semaphore。
2. Semaphore(int permits, boolean fair) ：创建具有给定的许可数和给定的公平设置的 Semaphore。
###### 方法
1. Semaphore提供了acquire()方法来获取一个许可。
2. 当用完之后就需要释放，Semaphore提供release()来释放许可。
###### 案例：停车
```java
public class Demo3Semaphore {
    public static void main(String[] args) {
        Parking parking = new Parking(3);
        for (int i = 0; i < 5; i++) {
            new Car(parking).start();
        }
    }
    static class Parking {
        //信号量
        private Semaphore semaphore;
        Parking(int count) {
            semaphore = new Semaphore(count);
        }
        public void park() {
            try {
                //获取信号量
                semaphore.acquire();
                long time = (long) (Math.random() * 10);
                System.out.println(Thread.currentThread().getName() + "进入停车场，停车" + time + "秒...");
                Thread.sleep(time);
                System.out.println(Thread.currentThread().getName() + "开出停车场...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                //释放信号量
                semaphore.release();
            }
        }
    }
    static class Car extends Thread {
        Parking parking;
        Car(Parking parking) {
            this.parking = parking;
        }
        @Override
        public void run() {
            //进入停车场
            parking.park();
        }
    }
}
```
##### 了解ConcurrentSkipListMap
#### 并发容器
##### ConcurrentHashMap
###### 介绍
- Hashtable和Collections.synchronizedMap(hashMap)两种解决方案，但是这两种方案都是对读写加锁，独占式。
- 在1.8版本以前，ConcurrentHashMap采用分段锁的概念，使锁更加细化，但是1.8已经改变了这种思路，而是利用CAS+Synchronized来保证并发更新的安全，当然底层采用数组+链表+红黑树的存储结构。
###### 结构
- 整个 ConcurrentHashMap 由一个个 Segment 组成，Segment 代表”部分“或”一段“的意思，所以很多人都会将其描述为分段锁。简单的说，ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的。
- 再具体到每个 Segment 内部，其实每个 Segment 很像之前介绍的 HashMap，每次操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的。
###### 总结
- ConcurrentHashMap线程安全的，允许一边更新、一边遍历，也就是说在对象遍历的时候，也可以进行remove,put操作，且遍历的数据会随着remove,put操作产出变化，
##### ConcurrentSkipListMap
#### 队列

Queue | 阻塞与否 | 是否有界 | 线程安全保障 | 适用场景 | 注意事项
---|---|---|---|---|---
ConcurrentLinkedQueue | 非阻塞 | 无界 | CAS | 对全局的集合进行操作的场景 | size()是要遍历一遍集合，慎用
ArrayBlockingQueue | 阻塞 | 有界 | 一把全局锁 | 生产消费模型，平衡两边处理速度
LinkedBlockingQueue | 阻塞 | 可配置 | 存取采用2把锁 | 生产消费模型，平衡两边处理速度 | 无界的时候注意内存溢出问题
PriorityBlockingQueue | 阻塞 | 无界 | 一把全局锁 | 支持优先级排序
SynchronousQueue | 阻塞 | 无界 | CAS | 不存储元素的阻塞队列
##### 非阻塞队列ConcurrentLinkedQueue
- ConcurrentLinkedQueue是一个基于链接节点的无边界的线程安全队列，遵循队列的FIFO原则，队尾入队，队首出队。采用CAS算法来实现的。
- 注意：
    - ConcurrentLinkedQueue的.size() 是要遍历一遍集合的，很慢的，所以尽量要避免用size
    - 使用了这个ConcurrentLinkedQueue 类之后还是需要自己进行同步或加锁操作。例如queue.isEmpty()后再进行队列操作queue.add()是不能保证安全的，因为可能queue.isEmpty()执行完成后，别的线程开始操作队列。
##### 阻塞队列BlockingQueue
- BlockingQueue
1. 被阻塞的情况主要有如下两种：
    1. 当队列满了的时候进行入队列操作
    2. 当队列空了的时候进行出队列操作
2. BlockingQueue 对插入操作、移除操作、获取元素操作提供了四种不同的方法用于不同的场景中使用：
- 抛出异常
- 返回特殊值（null 或 true/false，取决于具体的操作）
- 阻塞等待此操作，直到这个操作成功
- 阻塞等待此操作，直到成功或者超时指定时间。

操作 | 抛出异常 | 特殊值 | 阻塞 | 超时
--- | --- | --- | --- | ---
插入|add(e)|offer(e)|put(e)|offer(e,time,unit)
移除|remove()|poll()|take()|poll(time,unit)
检查|element()|peek()|不可用|不可用
#### JUC线程池
##### 线程池状态
- 变量ctl定义为AtomicInteger ，记录了“线程池中的任务数量”和“线程池的状态”两个信息。共32位，其中高3位表示”线程池状态”，低29位表示”线程池中的任务数量”。
1. RUNNING：处于RUNNING状态的线程池能够接受新任务，以及对新添加的任务进行处理。
2. SHUTDOWN：处于SHUTDOWN状态的线程池不可以接受新任务，但是可以对已添加的任务进行处理。
3. STOP：处于STOP状态的线程池不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。
4. TIDYING：当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。
5. TERMINATED：线程池彻底终止的状态。

```
graph LR
A(RUNNING)-->|shutdown|B(SHUTDOWN)
A-->|shutdownNow|C(STOP)
B-->|阻塞队列为空,且线程池中执行的任务也为空时|D(TIDYING)
C-->|线程池中执行的任务为空|D
D-->|terminated|F(TERMINATED)
```

##### 构造方法
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
        null :
    AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```
###### 线程池七大参数
1. **corePoolSize**：线程池中核心线程的数量（也称为线程池的基本大小）
2. **maximumPoolSize**：线程池中允许的最大线程数。线程池的阻塞队列满了之后，如果还有任务提交，如果当前的线程数小于maximumPoolSize，则会新建线程来执行任务。
3. **keepAliveTime**：线程空闲的时间。线程的创建和销毁是需要代价的。线程执行完任务后不会立即销毁，而是继续存活一段时间：keepAliveTime。
4. **unit**：keepAliveTime的单位。TimeUnit
5. **workQueue**：用来保存等待执行的任务的BlockQueue阻塞队列，等待的任务必须实现Runnable接口。
    - 选择如下：
        1. ArrayBlockingQueue：基于数组结构的有界阻塞队列
        2. LinkedBlockingQueue：基于链表结构的有界阻塞队列，FIFO。
        3. PriorityBlockingQueue：具有优先级别的阻塞队列。
        4. SynchronousQueue：不存储元素的阻塞队列，每个插入操作都必须等待一个移出操作。
6. **threadFactory**：用于设置创建线程的工厂。ThreadFactory的作用就是提供创建线程的功能的线程工厂。他是通过newThread()方法提供创建线程的功能，newThread()方法创建的线程都是“非守护线程”而且“线程优先级都是默认优先级”。
7. **handler**：RejectedExecutionHandler，线程池的拒绝策略。所谓拒绝策略，是指将任务添加到线程池中时，线程池拒绝该任务所采取的相应策略。当向线程池中提交任务时，如果此时线程池中的线程已经饱和了，而且阻塞队列也已经满了，则线程池会选择一种拒绝策略来处理该任务。
    - **四种拒绝策略**：
        1. AbortPolicy：直接抛出异常，默认策略；
        2. CallerRunsPolicy：用调用者所在的线程来执行任务；
        3. DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；
        4. DiscardPolicy：直接丢弃任务；
        5. 当然我们也可以实现自己的拒绝策略，例如记录日志等等，实现RejectedExecutionHandler接口即可。
##### 四种线程池
###### FixedThreadPool
- FixedThreadPool是复用固定数量的线程处理一个共享的无边界队列
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```
###### SingleThreadExecutor
- SingleThreadExecutor只会使用单个工作线程来执行一个无边界的队列。
```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```
###### CachedThreadPool
- CachedThreadPool会根据需要，在线程可用时，重用之前构造好的池中线程，否则创建新线程：
```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```
###### ScheduledThreadPool
- 继承ThreadPoolExecutor且实现了ScheduledExecutorService接口，它就相当于提供了“延迟”和“周期执行”功能的ThreadPoolExecutor。
- 它可另行安排在给定的延迟后运行命令，或者定期执行命令。需要多个辅助线程时，或者要求 ThreadPoolExecutor 具有额外的灵活性或功能时，此类要优于Timer。