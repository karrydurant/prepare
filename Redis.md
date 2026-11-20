# Redis

## 数据结构

### 讲一下 Redis 底层的数据结构

Redis 提供了丰富的数据类型，常见的有 5 种：String、Hash、List、Set、Zset（有序集合）

随着 Redis 版本更新，后面又提供了 BitMap、HyperLogLog、GEO、Stream。

另外数据类型和底层数据结构不是一回事。String、Hash、List 是使用时看到的数据类型，而 SDS、哈希表、跳表、listpack、quicklist 等才是底层实现。同一种数据类型还会根据数据大小和版本选择不同的编码。

| 数据类型 | 常见底层编码 | 什么时候使用 |
| ---- | ---- | ---- |
| String | int、embstr、raw | 可表示为整数时尝试使用 int，短字符串使用 embstr，较长字符串使用 raw |
|  |  |  |

### 有用过 Zset 吗？

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

文章 arcticle:4 新增一个赞，可以使用 ZINCRBY 命令（为有序集合）
