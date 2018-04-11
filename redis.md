# Redis 相关资料总结

#### [Redis的7个应用场景](https://my.oschina.net/architectliuyuanyuan/blog/1791445)

1. 缓存，热数据

**应用场景**：

- SELECT 数据库前查询 Redis，若有则使用 Redis 数据，反之 SELECT 数据库，并插入到 Redis 中

- UPDATE 或 DELETE 数据，查询 Redis 是否存在该数据，若存在则先删除 Redis 中数据，然后 UPDATE 或 DELETE 数据库

**可能问题**：并发量小时，以上操作不易出现问题，但高并发时，UPDATE 或 DELETE 操作时，删除 Redis 与更新数据库之间并非原子操作，在此之间可能发生 SELECT 操作，此时发现 Redis 并没有数据，将一条脏数据（旧数据）重新放回 Redis，导致数据一场。

2. 计数器

应用场景：统计点击数（[INCRBY](https://redis.io/commands/incrby)）

Redis 单进程单线程模型，无并发问题，毫秒级性能。

3. 队列

4. 位操作（大数据处理）

[SETBIT](https://redis.io/commands/setbit), [GETBIT](https://redis.io/commands/getbit), [BITCOUNT](https://redis.io/commands/bitcount)

5. 分布式锁与单线程机制

- 验证前端重复请求

- 秒杀系统

- 全局增量ID

6. 最新列表

如新闻列表，当新闻列表总量很大时，尽量不要使用 `SELECT a FROM A LIMIT 10` 这类操作，尝试使用 Redis 的 [LPUSH](https://redis.io/commands/lpush) 命令构建 list

7. 排行榜

[ZADD](https://redis.io/commands/zadd)：有序集合(sorted set)
