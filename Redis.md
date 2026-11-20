# Redis

## 数据结构

### :heavy_multiplication_x:讲一下 Redis 底层的数据结构

Redis 提供了丰富的数据类型，常见的有 5 种：String、Hash、List、Set、Zset（有序集合）

随着 Redis 版本更新，后面又提供了 BitMap、HyperLogLog、GEO、Stream。

另外数据类型和底层数据结构不是一回事。String、Hash、List 是使用时看到的数据类型，而 SDS、哈希表、跳表、listpack、quicklist 等才是底层实现。同一种数据类型还会根据数据大小和版本选择不同的编码。

| 数据类型 | 常见底层编码 | 什么时候使用 |
| ---- | ---- | ---- |
| String | int、embstr、raw | 可表示为整数时尝试使用 int，短字符串使用 embstr，较长字符串使用 raw |
|  |  |  |

### :heavy_check_mark:有用过 Zset 吗？

可以使用 Zset 实现排行榜的功能。

以博文点赞排名为例，小林发表了五篇博文，分别获得赞为 200、40、100、50、150。

```redis
# arcticle:1 文章获得了200个赞
> ZADD user:xiaolin:ranking 200 arcticle:1
(integer) 1
# arcticle:2 文章获得了40个赞
> ZADD user:xiaolin:ranking 40 arcticle:2
(integer) 1
# arcticle:3 文章获得了100个赞
> ZADD user:xiaolin:ranking 100 arcticle:3
(integer) 1
# arcticle:4 文章获得了50个赞
> ZADD user:xiaolin:ranking 50 arcticle:4
(integer) 1
# arcticle:5 文章获得了150个赞
> ZADD user:xiaolin:ranking 150 arcticle:5
(integer) 1
```

文章 arcticle:4 新增一个赞，可以使用 ZINCRBY 命令（为有序集合中对应元素的分值加上 increment）

```redis
> ZINCRBY user:xiaolin:ranking 1 arcticle:4
"51"
```

查看某篇文章的赞数，可以使用 ZSCORE 命令（返回有序集合中指定成员的分值 score）

```redis
> ZSCORE user:xiaolin:ranking arcticle:4
"51"
```

获取小林赞数最多的 3 篇文章，可以使用 ZREVRANGE 命令（倒序获取有序集合从 start 下标到 stop 下标的元素）

```redis
# WITHSCORES 表示把 score 也显示出来
> ZREVRANGE user:xiaolin:ranking 0 2 WITHSCORES
1) "arcticle:1"
2) "200"
3) "arcticle:5"
4) "150"
5) "arcticle:3"
6) "100"
```

获取小林 100 赞到 200 赞的文章，可以使用 ZRANGEBYSCORE 命令（返回有序集合中指定分数区间内的成员，分数由低到高排序）

```redis
> ZRANGEBYSCORE user:xiaolin:ranking 100 200 WITHSCORES
1) "arcticle:3"
2) "100"
3) "arcticle:5"
4) "150"
5) "arcticle:1"
6) "200"
```

### :heavy_multiplication_x:Redis 中 Set 和 Zset 有什么区别？

Redis 中的 Set 和 Zset 都是用于存储多个元素的集合类型，核心区别在于是否对元素进行排序以及排序方式

Set 是无序、唯一元素的集合，适合存储不重复且无需排序的数据（用户ID）。常用命令：

添加元素：`SADD set1 "a" "b" "c"`（向 set1 中添加 3 个元素，重复添加会自动去重）

### :heavy_multiplication_x:Zset 底层是怎么实现的

以 Redis 7.0 为例，Zset 类型有两种底层编码：listpack，以及跳表+哈希表（dict）。

### 跳表是怎么实现的

链表在查找元素时，因为要逐一查找，所以查询效率很低，时间复杂度是 O(n)，于是就出现了跳表。跳表是在链表基础上改进过来的，实现了一种多层的有序链表，这样的好处是能快速定位数据。

比如一个层级为 3 的跳表：

<img width="1164" height="287" alt="1719804939236-89f12a47-b851-4d06-a5f3-399e1119db57" src="https://github.com/user-attachments/assets/a5862083-4922-4178-8c71-edf0bff1e1fe" />

##### 在跳表中查询一个元素的大致步骤

1.根据 score 查找节点（比如 ZRANGESCORE，定位某个 score 的位置）：

目标是找到第一个满足 score 大于等于目标 score 的节点。

先从最高层索引开始，当前节点为头节点；再逐层向右试探，看当前节点的同层后继节点，如果后继存在并且后继节点的 score 小于目标 score，继续在这一层向右找，否则下沉一层，再逐层向右试探，直到降到最底层结束逐层遍历，在最底层向右判断。

2.按排名 rank 查询（ZRANK/ZREVRANK、取第 N 名）

rank 代表：前面有多少个元素比它小，跳表节点维护了 span（这个指针跨越多少个底层节点）

从最高层头节点开始，rank_acc=0，当前层看后继节点，如果 rank_acc + 后继.span <= 目标 rank，向右走，rank_acc += span，否则下沉一层，直到最底层

跳表的平均查找复杂度是 O(log N)，最坏情况下仍可能退化成 O(N)，随机层高让这种退化不易发生。

##### 跳表是怎么实现多层级的呢？

```c
typedef struct zskiplistNode {
    //Zset 对象的元素值
    sds ele
;
    //元素权重值
    double score;
    //后向指针
    struct zskiplistNode *backward;
  
    //节点的level数组，保存每层上的前向指针和跨度
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span;
    } level[];
} zskiplistNode;
```

Zset 对象同时保存元素和元素的权重，每个跳表节点都有一个后向指针，指向前一个节点，目的是为了方便从跳表的尾节点开始访问节点，方便倒叙查找。

跳表是一个带有层级关系的链表，每一层可以包含多个节点，每个节点通过指针连接起来，实现这一特性就是靠跳表节点结构体中的 zskiplistLevel 结构体类型的 level 数组。level 数组中的每一个元素代表跳表的一层

### :heavy_check_mark:跳表是怎么设置层高的

跳表的层高是随机生成的。以 Redis 7.0 为例，生成层高时，初始总层数是 1，循环尝试往上加层，每次 25% 概率成功，失败就停止，总层数上限 32 层。

之所以要随机是因为这样就不用在每次插入、删除之后重新维护严格的层级比例，也能让高层索引保持稀疏，取得平均 O(logN) 的查找效率。
