# 消息队列选型：Kafka vs RabbitMQ

## 基本信息

- **类型**: case
- **版块**: 
- **标签**: 消息队列, 分布式系统, 系统架构, Kafka, RabbitMQ

## 核心对比

| 维度 | Kafka | RabbitMQ |
|------|-------|----------|
| **设计理念** | 分布式日志系统，以日志为中心 | 传统消息代理，以队列为中心 |
| **吞吐量** | 极高（百万级/秒） | 中等（万级/秒） |
| **消息持久化** | 顺序写磁盘，O(1)读取 | 内存或磁盘，索引读取 |
| **消息顺序** | 分区内有序 | 队列内有序 |
| **消息路由** | 基于Topic/Partition | 基于Exchange+RoutingKey |
| **消费模式** | 拉取（Pull） | 推送（Push） |
| **ACK机制** | 基于Offset | 手动ACK |
| **消息回溯** | 支持（重新消费任意位置） | 不支持 |
| **多消费者** | 消费组隔离，独立消费 | 同一队列消息被单一消费者消费 |
| **集群部署** | 原生分布式，Leader-Follower | 主从或联邦模式 |

## 架构差异

### Kafka

```
Producer → Topic (N Partition) → Consumer Group (N Consumer)
                     ↓
              Leader-Follower Replica
```

- **Partition**：物理存储单位，每个Partition有序
- **Consumer Group**：组内消费者竞争消费，组间隔离
- **Offset**：消费者提交消费位置，支持回溯

### RabbitMQ

```
Producer → Exchange → Binding → Queue → Consumer
                               ↓
                        Dead Letter Exchange
```

- **Exchange类型**：Direct、Fanout、Topic、Headers
- **Queue**：消息存储实体，遵循FIFO
- **Binding**：Exchange到Queue的路由规则

## 选型决策树

```
                    ↓
         吞吐量需求如何？
           ↓               ↓
        百万级            十万级以下
           ↓               ↓
        Kafka ←—— 是否需要灵活路由？——→ RabbitMQ
                       ↓              ↓
                   需要           不需要
                   复杂路由        简单队列
```

## 典型场景

### 选择Kafka的场景

1. **日志收集与分析**：高吞吐，允许多消费者组独立消费
2. **事件溯源**：需要消息回溯能力
3. **大数据管道**：与Spark、Flink等生态无缝集成
4. **ClickHouse/数据仓库**：写入密集型场景
5. **订单后置处理**：高并发，消息量大

### 选择RabbitMQ的场景

1. **任务队列**：一对多、多对多的任务分发
2. **复杂路由**：根据消息属性灵活路由到不同队列
3. **请求/响应模式**：RPC、异步回调
4. **优先级队列**：消息优先级处理
5. **事务消息**：严格的消息可靠性要求

## 关键差异解析

### 消息持久化

Kafka：顺序写磁盘，OS Page Cache优化，O(1)读取
RabbitMQ：内存+磁盘混合，消息可选持久化

### 消息消费语义

Kafka：
- At Least Once（默认）
- At Most Once（手动控制Offset）
- Exactly Once（事务支持）

RabbitMQ：
- 手动ACK确保可靠消费
- 消息确认后可重新投递

### 分区与扩展性

Kafka：
- 分区独立，消费并发度=分区数
- 分区分配在Consumer Group内动态 Rebalance

RabbitMQ：
- 队列本身不支持分区
- 水平扩展通过队列镜像或联邦

## 总结

| 场景 | 推荐 |
|------|------|
| 日志系统、大数据分析 | Kafka |
| 任务队列、异步任务 | RabbitMQ |
| 高并发、低延迟 | Kafka |
| 复杂业务路由 | RabbitMQ |
| 消息回溯需求 | Kafka |
| 生态集成需求 | Kafka |

**核心原则**：性能优先选Kafka，路由灵活性优先选RabbitMQ。
