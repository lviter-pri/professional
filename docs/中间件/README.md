# 中间件

中间件是构建分布式系统的基础设施，提供缓存、消息队列、搜索等核心能力。本目录涵盖主流中间件的使用和原理。

## 一、中间件分类

### 1.1 缓存中间件

| 中间件 | 类型 | 特点 |
|--------|------|------|
| **Redis** | 内存键值存储 | 高性能、支持多种数据结构 |
| **Memcached** | 内存键值存储 | 简单、轻量 |

### 1.2 消息队列

| 中间件 | 特点 | 适用场景 |
|--------|------|----------|
| **Kafka** | 高吞吐、分布式 | 日志收集、实时流处理 |
| **RabbitMQ** | 可靠、灵活路由 | 异步解耦、任务分发 |
| **RocketMQ** | 高性能、事务消息 | 金融级消息处理 |

### 1.3 搜索引擎

| 中间件 | 特点 | 适用场景 |
|--------|------|----------|
| **Elasticsearch** | 全文搜索、实时分析 | 搜索服务、日志分析 |
| **Solr** | 企业级搜索 | 电商搜索 |

### 1.4 数据库

| 中间件 | 类型 | 特点 |
|--------|------|------|
| **PostgreSQL** | 关系型数据库 | 功能丰富、支持JSON |
| **MySQL** | 关系型数据库 | 性能优异、生态成熟 |

## 二、中间件选型指南

### 2.1 缓存选型

| 场景 | 推荐 | 理由 |
|------|------|------|
| 复杂数据结构 | Redis | 支持List、Set、Hash等 |
| 纯缓存场景 | Memcached | 简单高效 |
| 分布式锁 | Redis | 支持原子操作 |
| 延迟队列 | Redis | ZSet实现 |

### 2.2 消息队列选型

| 场景 | 推荐 | 理由 |
|------|------|------|
| 高吞吐日志 | Kafka | 百万级TPS |
| 可靠消息 | RabbitMQ | 确认机制完善 |
| 事务消息 | RocketMQ | 金融级 |
| 实时计算 | Kafka | 流处理集成 |

### 2.3 搜索引擎选型

| 场景 | 推荐 | 理由 |
|------|------|------|
| 全文搜索 | Elasticsearch | 开箱即用 |
| 日志分析 | Elasticsearch | ELK生态 |
| 企业搜索 | Solr | 成熟稳定 |

## 三、核心中间件详解

### 3.1 Redis

Redis是一个开源的高性能键值存储系统，支持多种数据结构：

- **String**：字符串，支持原子操作
- **List**：链表，支持两端操作
- **Set**：集合，支持交集、并集
- **Hash**：哈希表，适合存储对象
- **ZSet**：有序集合，支持排序

**典型应用：**
- 缓存层
- 分布式锁
- 会话存储
- 实时排行榜

### 3.2 Kafka

Kafka是分布式流处理平台：

**核心概念：**
- **Topic**：消息主题
- **Partition**：分区，并行处理
- **Consumer Group**：消费者组
- **Offset**：消费位置

**典型应用：**
- 日志收集
- 实时流处理
- 事件驱动架构

### 3.3 Elasticsearch

Elasticsearch是分布式搜索和分析引擎：

**核心特性：**
- 全文搜索
- 实时分析
- 分布式架构
- RESTful API

**典型应用：**
- 网站搜索
- 日志分析
- 指标监控

---

## 本目录包含的中间件文档

### Redis
- [Redis概述](redis/README.md)
- [Redis数据结构](redis/Redis数据结构.md)
- [Redis核心1-11基础篇](redis/Redis核心1-11基础篇.md)
- [Redis核心12-21高级篇](redis/Redis核心12-21高级篇.md)
- [Redis实现延迟队列](redis/Redis实现延迟队列.md)

### Elasticsearch
- [搜索引擎基础](ElasticSearch/搜索引擎.md)
- [ES基础语法](ElasticSearch/Es基础语法.md)
- [ES基础查询](ElasticSearch/Es基础查询.md)
- [Es7.4.2增删改查](ElasticSearch/Es7.4.2-RestHighLevelClient增删改查.md)
- [Es-Nested嵌套查询](ElasticSearch/Es-Nested嵌套查询.md)
- [es通用查询](ElasticSearch/es通用查询.md)
- [es复杂统计-绝对值](ElasticSearch/es复杂统计-绝对值.md)
- [安装ELK](ElasticSearch/安装ELK.md)

### 消息队列
- [消息队列概述](消息队列/消息队列.md)
- [Kafka入门](消息队列/Kafka/Kafka入门.md)
- [Kafka消费超时导致重复消费](消息队列/Kafka/Kafka消费超时导致重复消费.md)
- [RabbitMQ安装及使用](消息队列/RabbitMq/RabbitMQ安装及使用.md)

### PostgreSQL
- [PostgreSQL概述](Postgressql/ReadME.md)
- [初始Psql](Postgressql/初始Psql.md)
