---
创建时间: 2026-06-08
来源: javabetter.cn
tags:
  - 数据库/Redis
  - 面试
  - 缓存
---

# Redis

> 来源：[Redis学习路线](https://javabetter.cn/xuexiluxian/redis.html) | [面渣逆袭-Redis](https://javabetter.cn/sidebar/sanfene/redis.html)

---

## 一、Redis 特性

| 特性 | 说明 |
|:----|:-----|
| 纯内存 | 读写极快（10万+ QPS） |
| 单线程 | 避免并发竞争（6.x后支持多线程IO） |
| 五种数据结构 | String、List、Set、ZSet、Hash |
| 持久化 | RDB（快照）+ AOF（日志） |
| 高可用 | 主从 + 哨兵 + 集群 |

---

## 二、数据结构应用场景

| 数据类型 | 典型应用 |
|:---------|:---------|
| **String** | 缓存、计数器（INCR）、分布式锁（SETNX） |
| **Hash** | 存储对象（用户信息、商品详情） |
| **List** | 消息队列（LPUSH + BRPOP）、最新列表 |
| **Set** | 去重、共同好友（SINTER）、随机抽奖 |
| **ZSet** | 排行榜（ZREVRANGE）、延迟队列 |

---

## 三、持久化

| 对比 | RDB | AOF |
|:----|:----|:----|
| 方式 | 快照（全量数据） | 追加日志（操作记录） |
| 文件大小 | 小 | 大 |
| 恢复速度 | 快 | 慢 |
| 数据安全 | 可能丢数据 | 可配置每秒同步 |
| 推荐 | **两者都开启** | |

---

## 四、缓存问题与解决

### 1. 缓存穿透
**现象**：查询一个不存在的数据，每次都打到数据库

**解决**：布隆过滤器（Bloom Filter）/ 缓存空值

### 2. 缓存击穿
**现象**：热点 key 过期，大量并发请求同时打到数据库

**解决**：互斥锁（SETNX）/ 热点数据永不过期

### 3. 缓存雪崩
**现象**：大量 key 同时过期，或 Redis 宕机，请求打到数据库

**解决**：过期时间加随机值 / 多级缓存（本地+Redis）/ 限流降级

---

## 五、分布式锁

```bash
# 加锁（SETNX + EXPIRE 原子操作）
SET lock_key unique_id NX EX 30

# 解锁（Lua 脚本保证原子性）
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```

> **Redisson** 提供了更完善的分布式锁实现（可重入、看门狗自动续期）。
