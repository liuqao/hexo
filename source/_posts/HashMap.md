---
title: HashMap源码分析
date: 2020-03-26 12:00:00
tags: ['知识点','java']
categories: ['学习']
---
### HashMap1.7源码分析
>- HashMap是如何进行初始化的？
>- 哈希碰撞是如何处理的？
>- 如何计算存储数组Table的下标位置？
>- HashMap中哈希表是如何动态扩容的？

<!--more-->

#### 本质
- HashMap = 1个存储Entry类对象的数组 + 多个单链表
```java
 //核心静态内部类Entry 
 //Entry对象本质 = 1个映射（键 - 值对），属性包括：键（key）、值（value） & 下1节点( next) 
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    //因为HashMap是采用链表法处理哈希冲突的，所以Entry需要有一个指向下一个节点的指针
    Entry<K,V> next;
    int hash;

    /**
     * Creates new entry.
     */
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
    //省略...
}
```
#### HashMap中的重要参数
- 主要参数 = 容量、加载因子、扩容阈值
    - 容量（**capacity**）：HashMap中数组长度
        ```java
        // a. 容量范围：必须是2的幂 & <最大容量（2的30次方）
        // b. 初始容量 = 哈希表创建时的容量
        // 默认容量 = 16 = 1<<4 = 00001中的1向左移4位 = 10000 = 十进制的2^4=16
        static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
        // 最大容量 =  2的30次方（若传入的容量过大，将被最大值替换）
        static final int MAXIMUM_CAPACITY = 1 << 30;
        ```
    - 加载因子(**Load factor**)：HashMap在其容量自动增加前可达到多满的一种尺度
        ```java
        // a. 加载因子越大、填满的元素越多 = 空间利用率高、但冲突的机会加大、查找效率变低（因为链表变长了）
        // b. 加载因子越小、填满的元素越少 = 空间利用率小、冲突的机会减小、查找效率高（链表不长）
        // 实际加载因子
        final float loadFactor;
        // 默认加载因子 = 0.75
        static final float DEFAULT_LOAD_FACTOR = 0.75f;
        ```
    - 扩容阈值（threshold）：当哈希表的大小 ≥ 扩容阈值时，就会扩容哈希表（即扩充HashMap的容量）
        ```java
        // a. 扩容 = 对哈希表进行resize操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数
        // b. 扩容阈值 = 容量 x 加载因子
        int threshold;
        ```
#### 源码分析
##### 步骤1：声明一个HashMap对象
1. 声明对象
```java
Map<String,Integer> map = new HashMap<String,Integer>();
```
2. 源码分析
```java
/**
  * 源码分析：主要是HashMap的构造函数 = 4个
  * 仅贴出关于HashMap构造函数的源码
  */
public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable{
    // 省略上节阐述的参数
    /**
     * 构造函数1：默认构造函数（无参）
     * 加载因子 & 容量 = 默认 = 0.75、16
     */
    public HashMap() {
        // 实际上是调用构造函数3：指定“容量大小”和“加载因子”的构造函数
        // 传入的指定容量 & 加载因子 = 默认
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR); 
    }
    /**
     * 构造函数2：指定“容量大小”的构造函数
     * 加载因子 = 默认 = 0.75 、容量 = 指定大小
     */
    public HashMap(int initialCapacity) {
        // 实际上是调用指定“容量大小”和“加载因子”的构造函数
        // 只是在传入的加载因子参数 = 默认加载因子
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
     /**
     * 构造函数3：指定“容量大小”和“加载因子”的构造函数
     * 加载因子 & 容量 = 自己指定
     */
    public HashMap(int initialCapacity, float loadFactor) {
        // HashMap的最大容量只能是MAXIMUM_CAPACITY，哪怕传入的 > 最大容量
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        // 设置 加载因子
        this.loadFactor = loadFactor;
        // 设置 扩容阈值 = 初始容量
        // 注：此处不是真正的阈值，是为了扩展table，该阈值后面会重新计算，下面会详细讲解
        threshold = initialCapacity;   
        init(); // 一个空方法用于未来的子对象扩展
    }

}
```
> - 此处仅用于接收初始容量大小（capacity）、加载因子(Load factor)，但仍无真正初始化哈希表，即初始化存储数组table
> - 此处先给出结论：真正初始化哈希表（初始化存储数组table）是在第1次添加键值对时，即第1次调用put（）时。

##### 步骤2：向HashMap添加数据
1. 调用方法
```java
map.put("Java",1);
map.put("Study",2)
```
2. 源码分析
```java
public V put(K key, V value){
（分析1）// 1. 若 哈希表未初始化（即 table为空) 
        // 则使用 构造函数时设置的阈值(即初始容量) 初始化 数组table  
        if (table == EMPTY_TABLE) { 
            inflateTable(threshold); 
        }  
        // 2. 判断key是否为空值null
（分析2）// 2.1 若key == null，则将该键-值 存放到数组table 中的第1个位置，即table [0]
        // （本质：key = Null时，hash值 = 0，故存放到table[0]中）
        // 该位置永远只有1个value，新传进来的value会覆盖旧的value
        if (key == null)
            return putForNullKey(value);

（分析3） // 2.2 若 key ≠ null，则计算存放数组 table 中的位置（下标、索引）
        // a. 根据键值key计算hash值
        int hash = hash(key);
        // b. 根据hash值 最终获得 key对应存放的数组Table中位置
        int i = indexFor(hash, table.length);

        // 3. 判断该key对应的值是否已存在（通过遍历 以该数组元素为头结点的链表 逐个判断）
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
（分析4）// 3.1 若该key已存在（即 key-value已存在 ），则用 新value 替换 旧value
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue; //并返回旧的value
            }
        }

        modCount++;

（分析5）// 3.2 若 该key不存在，则将“key-value”添加到table中
        addEntry(hash, key, value, i);
        return null;
}
```
###### **分析1**： 初始化容量 
1. 函数使用原型
```java
if (table == EMPTY_TABLE) { 
        inflateTable(threshold); 
    }  
```
2. 分析1：源码内容
```java
 private void inflateTable(int toSize) {  
    // 1. 将传入的容量大小转化为：>传入容量大小的最小的2的次幂
    // 即如果传入的是容量大小是19，那么转化后，初始化容量大小为32（即2的5次幂）
    int capacity = roundUpToPowerOf2(toSize);->>分析1   
    // 2. 重新计算阈值 threshold = 容量 * 加载因子  
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);  
    // 3. 使用计算后的初始容量（已经是2的次幂） 初始化数组table（作为数组长度）
    // 即 哈希表的容量大小 = 数组大小（长度）
    table = new Entry[capacity]; //用该容量初始化table  
    initHashSeedAsNeeded(capacity); 
} 

private static int roundUpToPowerOf2(int number) {  
   
       //若 容量超过了最大值，初始化容量设置为最大值 ；否则，设置为：>传入容量大小的最小的2的次幂
       return number >= MAXIMUM_CAPACITY  ? 
            MAXIMUM_CAPACITY  : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;
}
```
###### 分析2：key为null
- 当 key ==null时，将该 key-value 的存储位置规定为数组table 中的第1个位置，即table [0]
```java
private V putForNullKey(V value) {  
    // 遍历以table[0]为首的链表，寻找是否存在key==null 对应的键值对
    // 1. 若有：则用新value 替换 旧value；同时返回旧的value值
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {  
      if (e.key == null) {   
            V oldValue = e.value;  
            e.value = value;  
            e.recordAccess(this);  
            return oldValue;  
        }  
    }
    modCount++;
    // 2 .若无key==null的键，那么调用addEntry（），将空键 & 对应的值封装到Entry中，并放到table[0]中
    addEntry(0, null, value, 0); 
    // 注：
    // a. addEntry（）的第1个参数 = hash值 = 传入0
    // b. 即 说明：当key = null时，也有hash值 = 0，所以HashMap的key 可为null
    // c. 对比HashTable，由于HashTable对key直接hashCode（），若key为null时，会抛出异常，所以HashTable的key不可为null
    // d. 此处只需知道是将 key-value 添加到HashMap中即可，关于addEntry（）的源码分析将等到下面再详细说明，
    return null;
```
###### 分析3：计算存放数组 table 中的位置（即 数组下标 or 索引）
1. 函数使用原型
```java
    // a. 根据键值key计算hash值 ->> 分析1
    int hash = hash(key);
    // b. 根据hash值 最终获得 key对应存放的数组Table中位置 ->> 分析2
    int i = indexFor(hash, table.length);
```
2. 源码分析
 - hash()
```java
static int hash(int h){
    h ^= k.hashCode(); 
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h>>>4);
}
/**
 * 函数源码分析2：indexFor(hash, table.length)
 * JDK 1.8中实际上无该函数，但原理相同，即具备类似作用的函数
 */
static int indexFor(int h, int length) {  
  return h & (length-1); 
}
```
 - 所有处理的根本目的，都是为了提高 存储key-value的数组下标位置 的随机性 & 分布均匀性，尽量避免出现hash值冲突。即：对于不同key，存储的数组下标位置要尽可能不一样
###### 分析4：若对应的key已存在，则 使用 新value 替换 旧value
**注**：当发生 Hash冲突时，为了保证 键key的唯一性哈希表并不会马上在链表中插入新数据，而是先查找该 key是否已存在，若已存在，则替换即可  
1. 函数使用原型
```java
for (Entry<K,V> e = table[i]; e != null; e = e.next) {
    Object k;
    // 2.1 若该key已存在（即 key-value已存在 ），则用 新value 替换 旧value
    if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
        V oldValue = e.value;
        e.value = value;
        e.recordAccess(this);
        return oldValue; //并返回旧的value
    }
}
```
###### 分析5：若对应的key不存在，则将该“key-value”添加到数组table的对应位置中
**此处有2点需特别注意：键值对的添加方式 & 扩容机制**
1. 键值对的添加方式：单链表的头插法
    - 将该位置（数组上）原来的数据放在该位置的（链表）下1个节点中（next）、在该位置（数组上）放入需插入的数据-> 从而形成链表
2. 扩容机制
    - 触发扩容 --> 转移数组
3. 函数使用原型
```java
void addEntry(int hash, K key, V value, int bucketIndex) { 
  // 1. 插入前，先判断容量是否足够
  // 1.1 若不足够，则进行扩容（2倍）、重新计算Hash值、重新计算存储数组下标
    if ((size >= threshold) && (null != table[bucketIndex])) {  
        resize(2 * table.length); // a. 扩容2倍  --> 分析1
        hash = (null != key) ? hash(key) : 0;  // b. 重新计算该Key对应的hash值
        bucketIndex = indexFor(hash, table.length);  // c. 重新计算该Key对应的hash值的存储数组下标位置
    }
// 1.2 若容量足够，则创建1个新的数组元素（Entry） 并放入到数组中--> 分析2
    createEntry(hash, key, value, bucketIndex);
}
```
4. 源码分析
```java
void resize(int newCapacity) {  
    
    // 1. 保存旧数组（old table） 
    Entry[] oldTable = table;  

    // 2. 保存旧容量（old capacity ），即数组长度
    int oldCapacity = oldTable.length; 

    // 3. 若旧容量已经是系统默认最大容量了，那么将阈值设置成整型的最大值，退出    
    if (oldCapacity == MAXIMUM_CAPACITY) {  
        threshold = Integer.MAX_VALUE;  
        return;  
    }  
  
    // 4. 根据新容量（2倍容量）新建1个数组，即新table  
    Entry[] newTable = new Entry[newCapacity];  

    // 5. 将旧数组上的数据（键值对）转移到新table中，从而完成扩容 ->>分析1.1 
    transfer(newTable); 

    // 6. 新数组table引用到HashMap的table属性上
    table = newTable;  

    // 7. 重新设置阈值  
    threshold = (int)(newCapacity * loadFactor); 
}
```
```java
void transfer(Entry[] newTable) {
      // 1. src引用了旧数组
      Entry[] src = table; 

      // 2. 获取新数组的大小 = 获取新容量大小                 
      int newCapacity = newTable.length;

      // 3. 通过遍历 旧数组，将旧数组上的数据（键值对）转移到新数组中
      for (int j = 0; j < src.length; j++) { 
          // 3.1 取得旧数组的每个元素  
          Entry<K,V> e = src[j];           
          if (e != null) {
              // 3.2 释放旧数组的对象引用（for循环后，旧数组不再引用任何对象）
              src[j] = null; 

              do { 
                  // 3.3 遍历 以该数组元素为首 的链表
                  // 注：转移链表时，因是单链表，故要保存下1个结点，否则转移后链表会断开
                  Entry<K,V> next = e.next; 
                 // 3.4 重新计算每个元素的存储位置
                 int i = indexFor(e.hash, newCapacity); 
                 // 3.5 将元素放在数组上：采用单链表的头插入方式 = 在链表头上存放数据 = 将数组位置的原有数据放在后1个指针、将需放入的数据放到数组位置中
                 // 即 扩容后，可能出现逆序：按旧链表的正序遍历链表、在新链表的头部依次插入
                 e.next = newTable[i]; 
                 newTable[i] = e;  
                 // 3.6 访问下1个Entry链上的元素，如此不断循环，直到遍历完该链表上的所有节点
                 e = next;             
             } while (e != null);
             // 如此不断循环，直到遍历完数组上的所有数据元素
         }
     }
 }
```
> - 设重新计算存储位置后不变，即扩容前 = 1->2->3，扩容后 = 3->2->1
> - 此时若（多线程）并发执行 put（）操作，一旦出现扩容情况，则 容易出现 环形链表，从而在获取数据、遍历链表时 形成死循环（Infinite Loop），即 死锁的状态 = 线程不安全

#### hashmap1.8区别
1. hashmap1.8引入了红黑树
2. hashmap1.8扩容迁移数据时以hash第5bit运算，将一条链表拆分为两条，低链表还在原索引，高链表索引 index + oldcap
3. hashmap1.8采用尾插法，避免hashmap死循环