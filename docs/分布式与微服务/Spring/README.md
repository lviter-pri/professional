# Spring Framework 深度解析

Spring Framework是Java企业级开发的核心框架，它的核心思想是**控制反转（IoC）**和**面向切面编程（AOP）**。本目录深入探讨Spring的核心机制和高级特性。

## 一、Spring核心模块

### 1.1 Core Container（核心容器）

| 模块 | 功能 |
|------|------|
| **Core** | IoC容器基础，BeanFactory接口 |
| **Beans** | Bean的定义、创建和管理 |
| **Context** | 应用上下文，提供框架级服务 |
| **Expression** | Spring表达式语言（SpEL） |

### 1.2 AOP模块
- 面向切面编程支持
- 方法拦截和增强
- 事务管理、日志等横切关注点

### 1.3 Data Access模块
- JDBC抽象层
- ORM框架集成（Hibernate、JPA）
- 事务管理抽象

### 1.4 Web模块
- Spring MVC框架
- RESTful服务支持
- WebSocket支持

## 二、Spring中的设计模式

Spring框架大量使用了设计模式，以下是一些核心模式：

### 2.1 工厂模式（Factory Pattern）
- **BeanFactory**：IoC容器的核心接口
- **ApplicationContext**：扩展的Bean工厂

### 2.2 单例模式（Singleton Pattern）
- 默认Bean作用域为单例
- 通过`@Scope("singleton")`显式声明

### 2.3 代理模式（Proxy Pattern）
- JDK动态代理：基于接口
- CGLIB代理：基于类继承
- AOP的核心实现机制

### 2.4 策略模式（Strategy Pattern）
- Resource接口的多种实现
- TransactionDefinition接口

### 2.5 模板方法模式（Template Method）
- JdbcTemplate、RestTemplate等
- 定义算法骨架，子类实现具体步骤

### 2.6 观察者模式（Observer Pattern）
- ApplicationEvent和ApplicationListener
- 事件发布/订阅机制

## 三、IoC与DI

### 3.1 控制反转（Inversion of Control）

**传统方式：**
```java
// 手动创建依赖
Service service = new Service();
Repository repo = new Repository();
service.setRepository(repo);
```

**IoC方式：**
```java
// Spring容器负责创建和注入
@Autowired
private Repository repository;
```

### 3.2 依赖注入（Dependency Injection）

**注入方式：**

| 方式 | 说明 | 示例 |
|------|------|------|
| **构造器注入** | 通过构造方法注入 | `@Autowired` + 构造函数 |
| **Setter注入** | 通过setter方法注入 | `@Autowired` + setter方法 |
| **字段注入** | 直接注入字段 | `@Autowired` + 字段 |
| **接口注入** | 通过接口回调注入 | `ApplicationContextAware` |

### 3.3 Bean生命周期

```
实例化 → 属性注入 → BeanNameAware → BeanFactoryAware → 
ApplicationContextAware → BeanPostProcessor.postProcessBeforeInitialization →
InitializingBean.afterPropertiesSet → @PostConstruct → 
BeanPostProcessor.postProcessAfterInitialization → 使用中 → 
DisposableBean.destroy → @PreDestroy
```

## 四、AOP（面向切面编程）

### 4.1 AOP核心概念

| 概念 | 说明 |
|------|------|
| **Aspect** | 切面，横切关注点的模块化 |
| **JoinPoint** | 连接点，程序执行的某个位置 |
| **Advice** | 通知，在连接点执行的代码 |
| **Pointcut** | 切点，匹配连接点的表达式 |
| **Weaving** | 织入，将切面应用到目标对象 |

### 4.2 通知类型

- **Before**：方法执行前
- **After**：方法执行后
- **AfterReturning**：方法返回后
- **AfterThrowing**：方法抛出异常后
- **Around**：环绕通知

### 4.3 AOP实现方式

**JDK动态代理：**
```java
Proxy.newProxyInstance(classLoader, interfaces, invocationHandler)
```

**CGLIB代理：**
```java
Enhancer.create(targetClass, methodInterceptor)
```

## 五、Spring线程并发处理

### 5.1 Bean的线程安全性

| 作用域 | 线程安全 | 说明 |
|--------|----------|------|
| **singleton** | 不安全 | 默认作用域，需注意共享状态 |
| **prototype** | 安全 | 每次请求创建新实例 |
| **request** | 安全 | 每个请求一个实例 |
| **session** | 安全 | 每个会话一个实例 |

### 5.2 线程安全实践

- **无状态Bean**：避免在单例Bean中存储可变状态
- **线程安全集合**：使用ConcurrentHashMap等
- **ThreadLocal**：线程本地变量
- **同步控制**：使用synchronized或Lock

## 六、Spring循环依赖

### 6.1 什么是循环依赖

当Bean A依赖Bean B，而Bean B又依赖Bean A时，就形成了循环依赖。

```java
// 循环依赖示例
@Component
public class A {
    @Autowired
    private B b;
}

@Component  
public class B {
    @Autowired
    private A a;
}
```

### 6.2 Spring如何解决循环依赖

**三级缓存机制：**

1. **singletonObjects**：单例对象缓存（成品）
2. **earlySingletonObjects**：早期单例对象缓存（半成品）
3. **singletonFactories**：单例工厂缓存（创建中的Bean）

**解决流程：**
```
创建A → 放入singletonFactories → 
创建B → 放入singletonFactories → 
B依赖A → 从singletonFactories获取A的早期引用 → 
B初始化完成 → 放入singletonObjects → 
A依赖B → 从singletonObjects获取B → 
A初始化完成 → 放入singletonObjects
```

### 6.3 无法解决的循环依赖

- **构造器注入的循环依赖**：无法解决
- **prototype作用域的循环依赖**：无法解决

## 七、Spring事务管理

### 7.1 事务传播行为

| 传播行为 | 说明 |
|----------|------|
| **REQUIRED** | 如果当前没有事务，创建新事务；否则加入当前事务 |
| **SUPPORTS** | 如果当前有事务，加入；否则以非事务方式执行 |
| **MANDATORY** | 必须在事务中执行，否则抛出异常 |
| **REQUIRES_NEW** | 创建新事务，挂起当前事务 |
| **NOT_SUPPORTED** | 以非事务方式执行，挂起当前事务 |
| **NEVER** | 必须在非事务中执行，否则抛出异常 |
| **NESTED** | 嵌套事务，底层使用Savepoint |

### 7.2 事务隔离级别

| 隔离级别 | 说明 |
|----------|------|
| **DEFAULT** | 使用数据库默认隔离级别 |
| **READ_UNCOMMITTED** | 允许读取未提交的数据 |
| **READ_COMMITTED** | 只能读取已提交的数据 |
| **REPEATABLE_READ** | 保证同一事务中多次读取相同数据结果一致 |
| **SERIALIZABLE** | 最高隔离级别，完全串行化 |

---

## 本目录包含的文档

- [InitillizingBean接口](InitillizingBean.md)
- [Spring AOP详解](SpringAOP.md)
- [Spring事务管理](Spring事务.md)
- [TransactionSynchronizationManager](TransactionSynchronizationManager.md)
- [大事务处理优化](大事务处理优化.md)
- [循环依赖解决方案](循环依赖.md)
