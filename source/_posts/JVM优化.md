---
title: JVM优化
date: 2020-03-29 18:12:00
tags: ['知识点','java']
categories: ['学习']
---
#### JVM运行参数
##### 三种参数类型
###### 标准参数
- -help
- -version
###### -X参数（非标准参数）
- -Xint
- -Xcomp
###### -XX参数（使用率较高）
- -XX:newSize
- -XX:+UseSerialGC
<!--more-->
#### -server与-client
##### 区别
- server VM的初始堆空间大一些，默认使用并行垃圾回收器，启动慢，运行快
- client VM保守一些，使用串行垃圾回收器，目的让JVM启动速度更快，但是运行速度慢些
- JVM启动时会根据硬件及操作系统自动选择使用server与client
- 32位默认client，64位默认server

##### -X参数
- 非标准参数，在不同JVM版本，参数可能有所不同，通过`java -X`查看非标准参数
###### -Xint    仅解释模式执行
- 强制JVM执行所有的字节码，降低运行速度
###### -Xcomp
- 把所有字节码编译成本地代码
###### -Xmined  混合模式执行（默认）
##### -XX   非标准参数，主要用于调优DEBUG
1. boolean类型
    - 格式：`-XX:[+-]<name>` 表示启用或禁用name属性
    - `-XX:+DisableExplicitGC` 表示启动禁用手动GC操作，system.gc()无效
2. 非boolean类型
    - 格式：`-XX:<name>=<value>` 表示name属性值为value
    - `-XX:NewRatio=1` 表示新生代和老年代比值
##### -Xms与-Xmx
- -Xms，jvm堆内存的初始大小，eg：-Xms512m
    - 等于`-XX:InitialHeapSize=<value>`
- -Xmx，jvm最大堆内存大小，eg：-Xmx2048m
    - 等于`-XX:MaxHeapSize=<value>`
##### 查看JVM的运行参数
- 执行命令时候添加：`-XX:+PrintFlagsFinal`
    - =表示默认值，:=表示值已经被修改
- 查看正在运行JVM参数
    1. 部署一个tomcat项目
    2. `jinfo -flags <进程id>`
        - jps -l，查看进程或者ps -ef
#### JVM内存模型
##### JDK1.7堆内存
- 年轻区(代)
    - Eden
    - Survivor
- 年老区(代)
- 永久代
##### JDK1.8堆内存
- 年轻区(代)
    - Eden
    - Survivor
- 年老区(代)
- 元空间
#### jstat命令的使用
- jstat命令可以查看堆内存各部分的使用量，以及加载类的数量
- `jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]`
##### 查看class加载统计
- `jstat class [运行id]`
- **说明：**
    - loaded：加载class的数量
    - Bytes：所占用空间大小
    - Unloaded：未加载数量
    - Bytes：未加载占用空间
    - Time：时间
##### 查看编译统计
- `jstat -compiler [运行id]`
- **说明：**
    - Compiler：编译数量
    - Failed：失败数量
    - Invalid：不可用数量
    - Time：时间
    - FailedType：失败类型
    - FailedMethod：失败的方法
##### 垃圾回收统计
- `jstat -gc [运行id]`
- **说明：**
    - S0C：第一个Survivor区的大小（KB）
    - S1C：第二个Survivor区的大小（KB）
    - S0U：第一个Survivor区的使用大小（KB）
    - S1U：第二个Survivor区的使用大小（KB）
    - EC：Eden区的大小（KB）
    - EU：Eden区的使用大小（KB）
    - OC：Old区大小（KB）
    - OU：Old区使用大小（KB）
    - MC：方法区大小（KB）
    - MU：方法区使用大小（KB）
    - CCSC：压缩类空间大小（KB）
    - CCSU：压缩类空间使用大小（KB）
    - YGC：年轻代垃圾回收次数
    - YGCT：年轻代垃圾回收消耗时间
    - FGC：老年代垃圾回收次数
    - FGCT：老年代垃圾回收消耗时间
    - GCT：垃圾回收消耗总时间
#### jmap的使用以及内存溢出分析
- 前面通过jstat可以堆jvm堆的内存进行统计分析
- 而jmap可以获取到更加详细的内容
- 如：内存使用情况的汇总，对内存溢出的定位与分析
##### 查看内存使用情况
- `jmap -heap [进程id]`
```bash
PS C:\Windows\system32> jmap -heap 12712
Attaching to process ID 12712, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.202-b08

using thread-local object allocation.
Parallel GC with 8 thread(s)

Heap Configuration: # 堆内存配置信息
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 2107637760 (2010.0MB)
   NewSize                  = 44040192 (42.0MB)
   MaxNewSize               = 702545920 (670.0MB)
   OldSize                  = 88080384 (84.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage: # 堆内存使用情况
PS Young Generation # 年轻代
Eden Space:
   capacity = 33554432 (32.0MB)
   used     = 5875576 (5.603385925292969MB)
   free     = 27678856 (26.39661407470703MB)
   17.510581016540527% used
From Space:
   capacity = 5242880 (5.0MB)
   used     = 5236992 (4.994384765625MB)
   free     = 5888 (0.005615234375MB)
   99.8876953125% used
To Space:
   capacity = 5242880 (5.0MB)
   used     = 0 (0.0MB)
   free     = 5242880 (5.0MB)
   0.0% used
PS Old Generation   # 老年代
   capacity = 88080384 (84.0MB)
   used     = 11080376 (10.567070007324219MB)
   free     = 77000008 (73.43292999267578MB)
   12.579845246814546% used

13186 interned Strings occupying 1817312 bytes.
PS C:\Windows\system32>
```
##### 查看内存中对象数量及大小
- 查看所有对象，包括活跃以及非活跃的
    - `jmap -histo [进程id] | more`
- 查看活跃对象
    - `jmap -histo:live [进程id] | more`
- 对象说明
    - B,C,D,F,I,J,Z,[,[L+类名   分别对应
    - byte,char,double,float,int,long,boolean,数组，其他对象
##### 将内存使用情况dump到文件中
```bash
# 用法：
jmap -dump:format=b,file=dumpFileName [进程id]
# 实例：    b表示二进制
jmap -dump:format=b,file=d:/dump.dat 12712
```
#### 通过jhat堆dump文件进行分析
- dump文件是一个二进制文件，不便查看，这时我们可以借助于jhat工具进行查看
```bash
# 用法：
jhat -port <port> <file>
# 示例：
PS C:\Windows\system32> jhat -port 9999 d:/dump.dat
Reading from d:/dump.dat...
Dump file created Sat Feb 29 18:55:35 CST 2020
Snapshot read, resolving...
Resolving 135005 objects...
Chasing references, expect 27 dots...........................
Eliminating duplicate references...........................
Snapshot resolved.
Started HTTP server on port 9999
Server is ready.
#然后浏览器根据端口访问
localhost:9999
localhost:9999/oql
```
#### 通过MAT工具对dump文件进行分析
##### 模拟内存泄漏
```java
public class TestJvmOutOfMemory {
    public static void main(String[] args) {
        List<Object> list = new ArrayList<Object>();
        for (int i = 0; i < 1000000; i++) {
            String str = "";
            for (int j = 0; j < 1000; j++) {
                str += UUID.randomUUID().toString();
            }
            list.add(str);
        }
        System.out.println("ok");
    }
}
```
##### 添加启动参数
`-Xms8m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError`
##### 使用Jprofiler分析内存快照
#### jstack的使用
- jstack的作用是将正在运行的jvm线程情况进行快照，并且打印出来
##### 线程的状态
1. 初始态（NEW）
    - 创建一个对象，但还未调用start启动线程时
2. 运行态（RUNNABLE）
    - 就绪态，只要CPU分配执行劝就能运行
    - 运行态，正在执行的线程
3. 阻塞态（BLOCKED）
    - 当一条正在执行的线程请求某一资源失败的时候，就会进入阻塞态
    - 而在java中，阻塞态专指请求锁失败时进入的状态
    - 由一条阻塞队列存放所有阻塞态的线程
    - 处于阻塞态的线程会不断请求资源，一旦请求成功就会进入就绪队列，等待执行
4. 等待态（WAITING）
    - 当线程调用wait，join，park函数时
    - 也有一个等待队列存放所有等待态的线程
    - 进入等待态的线程会释放CPU执行权，并释放资源，如（锁）
5. 超时等待态（TIMED_WAITING）
    - 与等待态的区别：到了超时时间后自动进入阻塞队列，开始竞争锁
6. 终止态（TERMINATED）
    - 线程结束后的状态
##### 实战：死锁问题
1. 构造死锁
```java
public class TestDeadLock {
    private static Object obj1 = new Object();
    private static Object obj2 = new Object();

    public static void main(String[] args) {
        new Thread(new Thread1()).start();
        new Thread(new Thread2()).start();
    }

    private static class Thread1 implements Runnable {

        public void run() {
            synchronized (obj1){
                System.out.println("Thread1 拿到了 obj1 的锁 ！");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (obj2){
                    System.out.println("Thread1 拿到了 obj2 的锁 ！");
                }
            }
        }
    }

    private static class Thread2 implements Runnable {

        public void run() {
            synchronized (obj2){
                System.out.println("Thread2 拿到了 obj2 的锁 ！");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (obj1){
                    System.out.println("Thread2 拿到了 obj1 的锁 ！");
                }
            }
        }
    }
}
```
2. 使用jstack进行分析
```bash
# 命令
jstack [进程id]
# 查看到死锁
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x0000000017f20d38 (object 0x00000000d6344828, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x0000000017f235c8 (object 0x00000000d6344838, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
        at cn.liuqao.pojo.TestDeadLock$Thread2.run(TestDeadLock.java:40)
        - waiting to lock <0x00000000d6344828> (a java.lang.Object)
        - locked <0x00000000d6344838> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)
"Thread-0":
        at cn.liuqao.pojo.TestDeadLock$Thread1.run(TestDeadLock.java:23)
        - waiting to lock <0x00000000d6344838> (a java.lang.Object)
        - locked <0x00000000d6344828> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```
#### VisualVM工具的使用
##### 启动
1. 进入jdk安装目录的bin目录下，找到jvisualvm.exe，运行
2. 可以查看本地或者远程，本地会自动监视
##### 监控远程的jvm
- **JMX**：java管理扩展，可以跨越一系列操作系统平台，系统体系结构和网络传输协议，灵活的开发无缝集成的系统，网络和服务管理应用
1. tomcat配置文件添加参数。。。
2. visualvm添加远程主机