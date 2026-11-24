# Java 集合

## 概念

### ✔️:数组与集合的区别，用过哪些集合？

##### 数组和集合的区别

1.数组是固定长度的数据结构，一旦创建长度就无法改变，而常见的 ArrayList、HashSet 等集合支持动态增删元素。不过，
Arrays.asList() 返回的列表是固定大小的，并不是所有集合都能增删。

2.数组可以包含基本数据类型和对象，而集合只能包含对象

3.数组通过下标访问元素，List 也支持 get(index) 访问；Set 没有下标、Map 通过 key 查找 value。能不能随机访问，
访问速度看具体实现。

##### 一些 Java 集合类

ArrayList：动态数组，实现了 List 接口，支持动态增长

LinkedList：双向链表，实现了 List 和 Deque 接口，头尾插入、删除很快，按下标操作则要先遍历定位

HashMap：基于哈希表的 Map 实现，存储键值对，通过键快速查找值。

HashSet：基于HashMap实现的Set集合，用于存储唯一元素。

TreeMap：基于红黑树实现的有序Map集合，可以按照键的顺序进行排序。

LinkedHashMap：基于哈希表和双向链表实现的Map集合，保持插入顺序或访问顺序。

PriorityQueue：优先队列，保证队头是优先级最高的元素，但遍历整个队列并不保证有序。

## Map

### ✔️:如何对 map 进行快速遍历

如果同时需要 key 和 value，通常优先使用 entrySet() 或 map.forEach()，不用遍历 key 后再调用一次 get。Stream 
更适合过滤、映射等处理，并不保证遍历更快。

使用 for-each 循环和 entrySet() 方法：这是一种较为常见和简洁的遍历方式，它可以同时获取 Map 中的键和值

```java
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println("Key: " + entry.getKey() + ", Value: " + entry.getValue());
}
```

使用 for-each 循环和 keySet() 方法：如果只要遍历 Map 中的键，可以使用 keySet() 方法，这种方式相对简单，性能也较好

```java
for (String key : map.keySet()) {
    System.out.println("Key: " + key);
}
```

使用迭代器：通过获取 Map 的 entrySet() 或 keySet() 的迭代器，也可以实现对 Map 的遍历，这种方式在需要删除元素等操作
时比较有用

```java
Iterator<Entry<String, Integer>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
    Entry<String, Integer> entry = iterator.next();
    System.out.println("Key: " + entry.getKey() + ", Value: " + entry.getValue());
}
```

使用 Lambda 表达式和 forEach() 方法

```java
map.foeEach((key, value) -> System.out.println("Key: " + key + ", Value: " + value));
```

使用 Stream API：可以将 Map 转换为流，然后进行各种操作

```java
map.entrySet().stream().forEach(entry -> System.out.println("Key: " + entry.getKey() + ", Value: " + entry.getValue()));

// 还可以进行其他操作，如过滤、映射等
Map<String, Integer> filteredMap = map.entrySet().stream()
                                    .filter(entry -> entry.getValue() > 1)
                                    .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
System.out.println(filteredMap);
```

### ✔️:介绍一下 HashMap 的实现原理

在 JDK 1.7 及以前，HashMap 数据结构是数组和链表，HashMap 通过哈希算法将元素的键映射到数组中的槽位。如果多个键
映射到同一个槽位，它们会以链表的形式存储在同一个槽位上。因为链表查询的时间复杂度是 O(n)，所以如果冲突严重，一个
索引上的链表就会非常长，效率就会很低。

在 JDK 1.8 版本的时候做了优化，以 JDK 8 的 put 为例，当桶中已有至少 8 个节点，且哈希表数组长度大于等于 64 时，再插入一个新节点，会把链表
转化为红黑树，改善长链表的查询效率，复杂度通常可以接近 O(logn)；如果数组长度小于 64，则只会触发扩容，并不会立刻树化。在 resize() 过程中，
若某个桶的节点数小于等于 6，红黑树会退化为链表。

### ✖️:HashMap 链表转红黑树、红黑树退化为链表的条件是什么

普通 put 追加链表节点时：桶中已有 8 个节点，再加一个达到 9 个时，会尝试树化；数组容量小于 64 时先扩容，容量至少为 64 才转成树。

### HashMap 是线程安全的吗

hashmap

### ✖️ConcurrentHashMap 怎么实现的

##### JDK 1.7

在 JDK1.7 中它使用的是数组加链表的形式实现的，而数组又分为：大数组 Segment 和小数组 HashEntry。Segment 是
一种可重入锁（ReentrantLock），在 ConcurrentHashMap 里扮演锁的角色；HashEntry 则用于存储键值对数据。一个 ConcurrentHashMap
里包含一个 Segment 数组，一个 Segment 里包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素。

<img width="622" height="330" alt="1721807523151-41ad316a-6264-48e8-9704-5b362bc0083c" src="https://github.com/user-attachments/assets/fbecf0c3-34cc-4be3-a3c6-96899e2f1423" />

JDK 1.7 ConcurrentHashMap 分段锁技术将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中
一个段数据时，其他段的数据也能被其他线程访问。

##### JDK 1.8

在 JDK 1.7 中，ConcurrentHashMap 虽然是线程安全的，但在哈希冲突严重、单个桶链表过长时，查询需要遍历较多节点，而 JDK 1.8 
则使用了数组+链表/红黑树的方式优化了 ConcurrentHashMap 的实现，具体实现结构如下:

<img width="455" height="388" alt="1721807523128-7b1419e7-e6ba-47e6-aba0-8b29423a8ce7" src="https://github.com/user-attachments/assets/82da16c2-d466-4b5a-8df8-e408545b47aa" />

JDK 1.8 ConcurrentHashMap 主要通过 volatile + CAS 或者 synchronized 来实现线程安全。添加元素时首先判断容器是否为空：

如果数组还没初始化，则通过 CAS 修改 sizeCtl，让一个线程负责初始化，其他线程等待或重试

如果容器不为空，则根据存储的元素计算该位置是否为空。如果是空的就用 CAS 设置该节点，如果不为空就用 synchronized，

空桶写入用 CAS，非空桶写入通过 synchronized 协调，读取依赖安全发布和可见性机制

### 介绍一下 HashMap 的扩容机制

hashMap 默认的负载因子是 0.75，即如果 hashmap 中的元素个数超过了总容量的 75%，则会触发扩容，扩容分为两个步骤：先对哈希表长度进行扩展（2倍），再将就哈希表中的数据放到新的哈希表中。

因为是将长度扩为 2 倍，所以元素的位置要么在原位置，要么在原位置再移动 2 次幂的位置

<img width="727" height="67" alt="1713514753772-9467a399-6b18-4a47-89d4-957adcc53cc0" src="https://github.com/user-attachments/assets/043a8ba0-58fb-4022-8d61-208423ebe0ba" />

在扩充 HashMap 的时候，不用重新计算 hash，只要看原来的 hash 值新增的那个 bit 是 1 还是 0 即可

这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。
### HashMap 的大小为什么是 2 的 n 次方大小呢

HashMap 底层是 "数组+链表/红黑树" 的结构，当我们要存一个 key-value 时，第一步就是确定这个 key 存在数组的哪个位置（索引）。

HashMap 用的索引计算公式是：索引 = hash & (length - 1)

这里的 hash 是经过扰动处理后的 key 的哈希值，length 是数组的容量。这个公式的目的是用位运算代替取模运算（因为位运算直接操作二进制位，速度远快于除法/取模），但其生效的前提是 length 必须是 2 的 n 次方。

##### 保证位运算等价于取模，实现高效寻址

##### 让哈希值的低位更均匀，减少碰撞

HashMap 会对 key 的原始 hashCode 做扰动处理（JDK 8 里是 hash = (h = key.hashCode()) ^ (h >>> 16)）。目的是让 hash 值的二进制位尽可能均匀分布。

只有当 length-1 的二进制全是 1，才能"接住"这些均匀分布的位（少了一位来分散 hash，碰撞概率自然就高了）
