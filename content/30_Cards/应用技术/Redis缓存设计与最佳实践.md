# Redis缓存设计与最佳实践

type: procedure

## 1. 缓存设计核心原则

### 1.1 缓存读写策略

**Cache-Aside（旁路缓存）** — 最常用的策略

- 读：应用先查缓存，命中则返回；未命中则查数据库并写入缓存
- 写：先更新数据库，再删除缓存（而非更新缓存）

**Write-Through（穿透写）**

- 写操作同时更新缓存和数据库
- 保证强一致性，但写入延迟较高

**Write-Behind（回写）**

- 先写缓存，异步批量写回数据库
- 性能高，但存在数据丢失风险

### 1.2 缓存粒度设计

| 数据类型 | 缓存粒度 | 适用场景 |
|---------|---------|---------|
| 单条记录 | Key-Value | 用户信息、配置项 |
| 列表/集合 | 集合粒度 | 排行榜、feed流 |
| 页面级 | 页面片段 | 静态页面、模板渲染 |

### 1.3 TTL设计

- 热点数据：TTL设置为数据更新周期的 1/10 ~ 1/3
- 会话数据：TTL = 会话超时时间
- 计数类：TTL 可适当延长，通过版本号或时间戳控制刷新
- 永不过期数据：使用空字符串 + 逻辑过期时间，内部另存实际时间戳

## 2. 缓存一致性方案

### 2.1 缓存与数据库双写一致性

**先删缓存再更新数据库**（不推荐）

- 并发场景下可能导致旧数据被重新写入缓存

**先更新数据库再删缓存**（推荐）

- 配合延迟双删处理并发问题：
  1. 更新数据库
  2. 删除缓存
  3. 延迟 N ms 后再次删除缓存

```
更新数据库 → 删除缓存 → sleep(N) → 删除缓存
```

### 2.2 分布式锁方案

使用 Redisson 或 Redis SETNX 实现分布式锁，保证缓存更新和数据库更新的原子性。

```
SET lock_key unique_id NX PX 30000
```

### 2.3 订阅 Binlog 方案

通过 Canal 或 Debezium 订阅 MySQL Binlog，解析后更新 Redis缓存。适合数据量大且一致性要求高的场景。

## 3. 缓存击穿、穿透、雪崩

### 3.1 缓存击穿（Cache Miss Storm）

**定义**：热点key过期瞬间，大量请求同时穿透到数据库。

**解决方案**：

- **互斥锁**：只允许一个线程重建缓存，其他线程等待
- **逻辑过期**：存储逻辑过期时间，不设置实际TTL，获取时异步更新
- **永不过期**：热点数据不设过期时间，由版本号控制更新

### 3.2 缓存穿透（Cache Penetration）

**定义**：请求的数据既不在缓存也不在数据库中。

**解决方案**：

- **布隆过滤器**：将所有合法数据Key存入BloomFilter，拦截非法请求
- **空值缓存**：对不存在的数据也缓存短TTL的空值（如 "NULL"），TTL=60s
- **参数校验**：对请求参数进行基础合法性校验

### 3.3 缓存雪崩（Cache Avalanche）

**定义**：大量缓存Key同时过期，或Redis实例宕机，导致数据库压力骤增。

**解决方案**：

- **过期时间随机化**：在固定TTL基础上加随机偏移量，如 `baseTTL + rand(0, 600)`
- **多级缓存**：本地缓存 + Redis，减少对Redis的直接依赖
- **Redis高可用**：使用主从 + 哨兵或集群架构
- **流量控制**：使用Sentinel或Hystrix对数据库访问进行限流降级

## 4. 键值设计规范

### 4.1 Key命名规范

```
业务模块:实体类型:业务标识
```

示例：

```
user:info:12345
order:detail:98765
product:stock:SKU0001
```

### 4.2 Value设计

- 优先使用JSON字符串存储结构化数据
- 大对象考虑压缩（zlib）或拆分
- 避免存储过大的Value（建议 < 10KB）
- 频繁计数的场景使用 Redis Hash 而非 String

### 4.3 避免Big Key

- String类型：value 控制在 10KB 以内
- Collection类型：元素数量控制在 1万 以内
- 使用 SCAN 而非 KEYS 遍历

## 5. 性能优化

### 5.1 批量操作

- 使用 MGET/MSET 替代多次 GET/SET
- 使用 Pipeline 减少网络往返次数
- 使用 Lua 脚本保证批量操作的原子性

### 5.2 连接池配置

```
最大连接数 = (核心CPU数 * 2) + 主从数
等待超时：100~300ms
空闲连接：设置 min-idle，避免频繁创建销毁
```

### 5.3 合理选择数据结构

| 场景 | 推荐数据结构 |
|-----|------------|
| 基础KV | STRING |
| 对象存储 | HASH |
| 排行/积分 | ZSET |
| 标签/去重 | SET |
| 消息队列 | LIST（BLPOP）/ Stream |
| 分布式锁 | STRING（SETNX + EXPIRE） |

## 6. 内存管理

### 6.1 内存淘汰策略（maxmemory-policy）

- `allkeys-lru`：所有Key按LRU淘汰（推荐）
- `volatile-lru`：带过期Key按LRU淘汰
- `allkeys-random`：所有Key随机淘汰
- `noeviction`：不淘汰，返回错误（默认）

### 6.2 内存碎片控制

- 合理设置 `maxmemory`，预留 20%~30% 空间
- 定期执行 MEMORY PURGE 释放碎片（需重启或AOF重写）
- 生产环境慎用 DEBUG SEGFAULT

## 7. 持久化与恢复

### 7.1 RDB + AOF 混合持久化

```
# 开启混合持久化
aof-use-rdb-preamble yes

# AOF重写触发策略
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

### 7.2 数据恢复顺序

1. 若开启混合持久化，先加载RDB部分
2. 再重放AOF增量部分
3. 确保恢复完整性

## 8. 高可用架构

### 8.1 主从 + 哨兵

- 至少 1 主 2 从 3 哨兵
- 哨兵负责自动故障转移
- 客户端连接哨兵获取主节点地址

### 8.2 Redis Cluster

- 数据自动分片（16384个槽位）
- 每个分片至少 1 主 1 从
- 支持故障自动转移
- 客户端需支持 MOVED 重定向

### 8.3 跨机房容灾

- 异地部署主从，延迟控制在 30ms 以内
- 使用 Twemproxy 或 Codis 做跨机房访问层代理

## 9. 监控与运维

### 9.1 关键监控指标

- 内存使用率：`used_memory / maxmemory`
- QPS：`instantaneous_ops_per_sec`
- 连接数：`connected_clients`
- 缓存命中率：`keyspace_hits / (keyspace_hits + keyspace_misses)`
- 慢查询：`slowlog get`

### 9.2 常见运维命令

```bash
# 查看内存使用前10的Key
redis-cli MEMORY USAGE $(redis-cli KEYS "*" | head -1)

# 诊断大Key
redis-cli --bigkeys

# 实时延迟监控
redis-cli --latency

# 内存碎片率
redis-cli INFO memory | grep mem_fragmentation_ratio
```

## 10. 常见错误避免

- **避免使用 KEYS 命令**：生产环境使用 SCAN 替代
- **避免存储明文密码**：敏感数据加密或使用专用配置中心
- **避免单点故障**：务必配置主从或集群
- **避免大Value**：拆分或压缩
- **避免没有超时机制的操作**：如 BLPOP 无限等待

---

> 参考版本：Redis 7.x
> 适用场景：Web应用后端、实时计算、消息队列、分布式锁等
