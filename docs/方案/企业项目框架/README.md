# 企业级项目框架

企业级项目框架是构建大型分布式系统的基础，涉及多种技术栈和架构模式。本目录涵盖了主流的企业级框架和架构设计方案。

## 一、企业级架构概述

### 1.1 架构演进历程

| 阶段 | 特点 | 技术栈 |
|------|------|--------|
| **单体应用** | 单一代码库，部署简单 | Spring MVC + MySQL |
| **垂直拆分** | 按业务模块拆分 | 多个独立单体应用 |
| **分布式服务** | 服务化拆分 | Dubbo + Zookeeper |
| **微服务架构** | 细粒度服务治理 | Spring Cloud / Dubbo |
| **云原生架构** | 容器化、自动化运维 | Kubernetes + Docker |

### 1.2 核心关注点

- **高可用**：服务容错、故障转移、负载均衡
- **高性能**：缓存策略、异步处理、优化数据库
- **可扩展**：水平扩展、弹性伸缩、服务网格
- **安全性**：认证授权、数据加密、访问控制
- **可观测性**：日志、监控、链路追踪

## 二、Spring 全家桶

### 2.1 Spring Framework

Spring Framework是Java企业级开发的基础框架，核心是IoC（控制反转）和AOP（面向切面编程）。

**核心模块：**
- **Core Container**：IoC容器、Bean管理
- **AOP**：面向切面编程
- **Data Access**：JDBC、ORM、事务管理
- **Web**：MVC框架、REST支持
- **Integration**：消息队列、远程调用

**设计理念：**
- 依赖注入（DI）
- 面向接口编程
- 关注点分离

### 2.2 Spring Boot

Spring Boot简化了Spring应用的初始化和部署，通过自动配置大大降低了开发门槛。

**核心特性：**
- **自动配置**：根据依赖自动配置Bean
- **起步依赖**：预定义的依赖组合
- **嵌入式服务器**：内置Tomcat、Jetty
- **Actuator**：应用监控和管理
- **约定大于配置**：减少样板代码

**典型应用场景：**
- 快速构建RESTful服务
- 微服务架构的服务实现
- 命令行应用

### 2.3 Spring Cloud

Spring Cloud提供了微服务架构所需的一系列工具，基于Spring Boot构建。

**核心组件：**

| 组件 | 功能 |
|------|------|
| **Eureka** | 服务注册与发现 |
| **Ribbon** | 客户端负载均衡 |
| **Feign** | 声明式HTTP客户端 |
| **Hystrix** | 服务容错与熔断 |
| **Zuul/Gateway** | API网关 |
| **Config** | 配置中心 |
| **Sleuth** | 分布式链路追踪 |

**架构优势：**
- 服务自动注册与发现
- 负载均衡和故障转移
- 统一配置管理
- 服务熔断和降级

## 三、Dubbo 分布式框架

Dubbo是阿里巴巴开源的高性能RPC框架，专注于服务治理。

### 3.1 Dubbo架构

```
服务提供者 → 注册中心 ← 服务消费者
    ↑              ↑              ↑
  配置中心      监控中心        配置中心
```

### 3.2 核心特性

- **高性能RPC**：基于Netty的高性能通信
- **服务治理**：注册发现、负载均衡、容错
- **可扩展协议**：支持多种协议（Dubbo、HTTP、gRPC）
- **多语言支持**：Java、Go、Python等

### 3.3 与Spring Cloud对比

| 维度 | Dubbo | Spring Cloud |
|------|-------|--------------|
| **通信方式** | RPC（同步） | HTTP（REST） |
| **性能** | 高性能 | 相对较低 |
| **生态** | 专注RPC | 全栈微服务 |
| **适用场景** | 内部服务调用 | 对外API、跨语言 |

## 四、ORM框架

### 4.1 MyBatis

MyBatis是一款优秀的持久层框架，支持定制化SQL、存储过程以及高级映射。

**核心优势：**
- SQL与代码分离
- 灵活的结果映射
- 动态SQL支持
- 缓存机制

## 五、项目研发规范

### 5.1 代码结构规范
合理的代码组织是项目可维护性的基础。

### 5.2 Git分支管理
规范的分支策略确保团队协作顺畅。

### 5.3 MapStruct
对象映射工具，简化DTO转换。

## 六、DDD领域驱动设计

领域驱动设计是一种软件开发方法，强调将业务领域模型作为设计的核心。

**核心概念：**
- **领域**：问题空间
- **实体**：具有唯一标识的对象
- **值对象**：无标识的对象
- **聚合**：一组相关对象的集合
- **领域服务**：跨实体的业务逻辑

---

## 本目录包含的文档

### Spring Framework
- [Spring概述](Spring/README.md)
- [Spring AOP](Spring/SpringAOP.md)
- [Spring事务](Spring/Spring事务.md)
- [循环依赖](Spring/循环依赖.md)
- [大事务处理优化](Spring/大事务处理优化.md)
- [InitillizingBean](Spring/InitillizingBean.md)
- [TransactionSynchronizationManager](Spring/TransactionSynchronizationManager.md)

### Spring Boot
- [SpringApplication解析](SpringBoot/SpringApplication解析.md)
- [SpringBoot-Arthas](SpringBoot/SpringBoot-Arthas.md)

### Spring Cloud
- [微服务SpringCloud](SpringCloud/微服务SpringCloud.md)
- [Eureka](SpringCloud/Eureka.md)
- [Feign](SpringCloud/Feign.md)
- [Hystrix](SpringCloud/Hystrix.md)

### SpringFramework源码
- [源码解析之ApplicationContext](SpringFramework/源码解析之ApplicationContext.md)

### Dubbo
- [Dubbo概述](dubbo/README.md)
- [Dubbo详解](dubbo/Dubbo.md)
- [分布式服务框架](dubbo/分布式服务框架.md)

### ORM框架
- [MyBatis详解](ORM框架/Mybatis详解.md)

### DDD领域
- [传统分包](DDD领域/传统分包.md)

### 项目研发基础规范
- [Git分支管理](项目研发基础规范/Git分支管理.md)
- [MapStruct](项目研发基础规范/MapStruct.md)
- [项目代码结构规范](项目研发基础规范/项目代码结构规范.md)
