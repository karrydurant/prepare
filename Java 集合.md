# Java 集合

## 概念

### ✔️:数组与集合的区别，用过哪些集合？

##### 数组和集合的区别

1.数组是固定长度的数据结构，一旦创建长度就无法改变，而常见的 ArrayList、HashSet 等集合支持动态增删元素。不过，Arrays.asList() 返回的列表是固定大小的，也不是所有集合都能增删。

2.数组可以包含基本数据类型和对象，而集合只能包含对象

3.数组通过下标访问元素，List 也支持 get(index) 访问；Set 没有下标、Map 通过 key 查找 value。能不能随机访问、访问速度要看具体实现。

##### 一些 Java 集合类

ArrayList：动态数组，实现了 List 接口，支持动态增长

LinkedList：双向链表，实现了 List 和 Deque 接口，头尾插入、删除很快，按下标操作则要先遍历定位

HashMap：基于哈希表的 Map 实现，存储键值对，通过键快速查找值

HashSet：基于 HashMap 实现的Set集合，用于存储唯一元素。

TreeMap：基于红黑树实现的有序 Map 集合，可以按照键的顺序进行排序。

LinkedHashMap：基于哈希表和双向链表实现的 Map 集合，保持插入顺序或访问顺序。

PriorityQueue：优先队列，保证队头是优先级最高的元素，但遍历整个队列并不保证有序。

### 说说 Java 中的集合

<img width="877" height="450" alt="1717481094793-b8ffe6ae-2ee6-4de5-b61b-8468e32bf269" src="https://github.com/user-attachments/assets/f6716c59-1329-4e37-9622-5e4944484727" />

List 是有序的 Collection，使用此接口能够精确地控制每个元素的插入位置，用户能根据索引访问 List 中元素。常用的实现 List 的类有：LinkedList、ArrayList、Vector、Stack。

Set 不允许存在重复的元素，但是否有序取决于实现类：HashSet 不保证顺序，LinkedHashSet 保留插入顺序，TreeSet 按比较规则排序。

Map 是一个键值对集合，存储键、值和之间的映射。key 唯一，value 可以重复，遍历顺序由实现类决定，比如 HashMap 不保证顺序，TreeMap 按 key 排序。Map 没有继承于 Collection 接口，从 Map 集合中检索元素时，给出键对象就会返回对应的值对象。主要实现有 TreeMap、HashMap、Hashtable、LinkedHashMap、ConcurrentHashMap

## List

### ArrayList 和 LinkedList 的区别，哪个集合是线程安全的

ArrayList 和 LinkedList 都是Java中常见的集合类，它们都实现了List接口。

底层数据结构不同：

ArrayList 使用动态数组实现，通过索引可以快速定位到元素。LinkedList 使用双向链表实现，每个节点都存储了元素本身以及指向前一个和后一个节点的指针，通过节点之间的指针关联来访问和操作元素。

插入和删除操作的效率不同：

ArrayList 在尾部进行插入和删除操作时效率较高，因为不需要移动其它元素，但如果是在中间或开头插入、删除，就要移动后面的所有元素，效率会比较低。
LinkedList 在头部或尾部进行插入、删除操作时效率很高，只要调整节点的指针即可，但如果要在中间位置操作，需要先从头或尾遍历链表找到目标位置，时间复杂度也是 O(N)，不过找到位置后只需要调整指针，不用像 ArrayList 那样移动大量元素，LinkedList 实现了 Deque 接口，还可以当作双端队列、栈来使用。

随机访问的效率不同：

ArrayList 支持通过索引直接快速访问元素，时间复杂度为 O(1)，LinkedList 不支持随机访问，想要获取某个位置的元素，必须从头节点或尾节点开始逐个遍历，时间复杂度为 O(N)

空间占用：

ArrayList 使用数组保存元素引用，虽然会有一定的容量浪费（比如实际元素没装满数组），但不需要为每个元素额外分配链表节点，元素对象本身也不一定连续存放。无参构造在 JDK 8 中会延迟分配实际的存储数组。LinkedList 每个节点除了存储元素引用，还要额外存储两个指针，所以在存储相同数量元素的情况下，LinkedList 的空间占用通常会比 ArrayList 更大一些。

### ArrayList 线程安全吗？把 ArrayList 变成线程安全有哪些方法？

不是线程安全的，ArrayList 变成线程安全的方式有：

使用 Collections 类的 synchronizedList 方法将 ArrayList 包装成线程安全的 List：

```java
List<String> synchronizedList = Collections.synchronizedList(arrayList);
```

使用 CopyOnWriteArrayList 类代替 ArrayList，它是一个线程安全的 List 实现：

```java
CopyOnWriteArrayList<String> copyOnWriteArrayList = new CopyOnWriteArrayList<>(arrayList);
```



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

hashmap 不是线程安全的，hashmap 在多线程会存在下面的问题：

JDK 1.7 HashMap 采用数组 + 链表的数据结构，多线程背景下，在数组扩容时，存在 Entry 链表死循环和数据丢失问题。

JDK 1.8 HashMap 采用数组 + 链表 + 红黑树的数据结构，并在扩容时将同一个桶的节点拆成低位链表和高位链表，按原顺序连接，因此解决了 JDK 1.7 头插法在并发扩容时可能形成环形链表、导致死循环的问题。但 JDK 1.8 并没有解决多线程操作 HashMap 时的数据丢失问题，resize() 和 put() 都没有同步保护，并发扩容可能因多个线程竞争 table、清空旧桶或改写节点的 next 引用而丢失节点，并发 put() 也可能发生数据覆盖

如果要保证线程安全，可以通过这些方法来保证：

多线程环境可以使用 Collections.synchronizedMap 包装普通 Map，所有访问都应经过这个包装对象。遍历和复合操作还要在同一把锁下执行。Hashtable 也能提供同步。但它们的主要操作共用一把对象锁，竞争较多时通常不如 ConcurrentHashMap 适合。

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

### LinkedHashMap 怎么保证有序，怎么用它实现 LRU

LinkedHashMap 在 HashMap 的基础上，增加了一条贯穿所有节点的双向链表。哈希表负责快速查找，双向链表负责记录遍历顺序。

它支持两种顺序：

插入顺序；默认模式，按 key 第一次插入的顺序遍历，更新已有 key 的 value 不会把它移到末尾

访问顺序：构造时把 accessOrder 设置为 true，get、put 等访问已有映射的操作会将对应节点移到末尾，头部就是最久没访问的节点

实现简单的 LRU，可以使用访问顺序，再重写 removeEldestEntry，超过容量时删除最老的节点

```java

```

### TreeMap 和 HashMap 有什么区别？什么场景使用 TreeMap？

底层结构：HashMap 是哈希表，JDK 8 中桶内可以是链表或红黑树；TreeMap 整个结构就是红黑树

查找方式：HashMap 依赖 hashCode 和 equals；TreeMap 依赖 key 的自然顺序或 Comparator，比较结果为 0 就认为是同一个 key。

顺序：HashMap 不保证遍历顺序；TreeMap 按 key 的比较规则排序，并不是按 value 排序，也不是按插入顺序

复杂度：哈希分布良好时，HashMap 的基本操作平均是 O(1)；TreeMap 的 get、put、remove 是 O(logN)

如果只需要按 key 快速查找，通常选 HashMap，如果需要排序、范围查询、寻找相邻 key，TreeMap 更合适

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

### ConcurrentHashMap 是线程安全的，先 get 再 put 就一定安全吗

不一定，单个方法线程安全，不代表多个方法组成的操作也具有原子性。

比如下面这个计数逻辑，两个线程可能同时读到 0，然后都写入 1，最终少算一次：

```java
ConcurrentHashMap<String, Integer> counts = new ConcurrentHashMap<>();
// 并发调用下面这行，可能丢失更新
counts.put("java", counts.getOrDefault("java", 0) + 1);
```

如果是累加可以使用 merge，让同一个 key 的这次更新具有原子性：

```java
counts.merge("java", 1, Integer::sum);
```

其它常见场景也有对用方法：

没有映射才添加：putIfAbsent

没有映射才计算并添加：computeIfAbsent

只有 value 仍是预期值才替换：replace(key, oldValue, newValue)

只有 key-value 仍匹配才删除：remove(key, value)

### ConcurrentHashMap 用了悲观锁还是乐观锁

都有用到。

添加元素时首先会判断容器是否为空：

如果数组还没初始化，则通过 CAS 竞争初始化资格，由一个线程完成初始化

如果容器不为空，则根据存储的元素计算该位置是否为空

如果是空，就利用 CAS(乐观锁) 设置该节点

如果不为空，就使用 synchronized(悲观锁) ，然后，遍历桶中的数据，并替换或新增节点到桶中，最后再判断是否要转为红黑树，这样就能保证并发访问时的线程安全了

