---
description: 公司服务端开发规范
alwaysApply: true
---

## 技术选型
- JDK 采用 JDK 1.8
- Spring Boot 采用 Spring Boot 2.5.X 或 1.5.X
- MyBatis 采用 MyBatis 3.4.X
- MySQL 采用 MySQL 5.7
- 代码风格采用阿里巴巴 Java 代码规范
- 配置中心使用apollo或SpringCloud Config
- 内部微服务调用优先使用Feign调用

## 测试要求
- 新增功能单元测试覆盖率大于80%，性能达标，安全无漏洞

## 禁止项（违反则必须纠正）
- 禁止直接使用开源API访问kafka, RocketMQ, 优先使用company-framework相关能力
- 禁止在Controller层编写业务逻辑，必须下沉到 Service
- 禁止在Service层直接操作数据库，必须通过Mapper
- 禁止使用System.out.println输出日志,必须使用SLF4J日志框架
- 禁止跨服务直接访问对方数据库,必须通过API调用
- 禁止在循环中操作数据库