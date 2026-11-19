# Redis

## 数据结构

### 讲一下 Redis 底层的数据结构

### 有用过 ZSet 吗

可以用 ZSet 实现排热榜的功能，以博文点赞排名为例，小林发表了五篇博文，分别获得赞为 200、40、100、50、150。

```redis
# arcticle:1 文章获得了200个赞
> ZADD user:xiaolin:ranking 200
 arcticle:1
(integer) 1
# arcticle:2 文章获得了40个赞
> ZADD user:xiaolin:ranking 40
 arcticle:2
(integer) 1
# arcticle:3 文章获得了100个赞
> ZADD user:xiaolin:ranking 100
 arcticle:3
(integer) 1
# arcticle:4 文章获得了50个赞
> ZADD user:xiaolin:ranking 50
 arcticle:4
(integer) 1
# arcticle:5 文章获得了150个赞
> ZADD user:xiaolin:ranking 150
 arcticle:5
(integer) 1
```

文章 article:4 新增一个赞，可以使用 ZINCRBY 命令（为有序集合key中元素member的分值加上increment）：

```redis
> ZINCRBY user:xiaolin:ranking 1 article:4
"51"
```

查看某篇文章的赞数，可以使用 ZSCORE 命令（返回有序集合中指定成员的分值 score）：

```redis
> ZSCORE user:xiaolin:ranking arcticle:4
"51"
```

获取小林文章赞数最多的 3 篇文章，可以使用 ZREVRANGE 命令（先把有序集合倒叙，然后取出排序后，下标从 start 到 stop 
这一段的元素）

```redis
# WITHSCORES 表示把 score 也显示出来
> ZREVRANGE user:xiaolin:ranking 0 2
 WITHSCORES
1) "arcticle:1"
2) "200"
3) "arcticle:5"
4) "150"
5) "arcticle:3"
6) "100"
```

获取小林 100 赞到 200 赞的文章，可以使用 ZRANGEBYSCORE 命令（返回有序集合中指定分数区间内的成员，分数由低到高排序）：

```redis
> ZRANGEBYSCORE user:xiaolin:ranking 100 200
 WITHSCORES
1) "arcticle:3"
2) "100"
3) "arcticle:5"
4) "150"
5) "arcticle:1"
6) "200"
```

### Redis 中 Set 和 ZSet 的区别是什么

区别在于是否对元素进行排序以及排序方式

### ZSet 底层是怎么实现的

