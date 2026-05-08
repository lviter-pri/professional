# Java Core

Java核心技术是Java开发的基石，涵盖了Java语言本身及其运行时环境的核心知识。掌握这些知识是成为优秀Java工程师的必备条件。

## 一、Java语言概述

### 1.1 Java的特点

- **跨平台性**：Write Once, Run Anywhere（一次编写，到处运行）
- **面向对象**：封装、继承、多态
- **自动内存管理**：垃圾回收机制
- **健壮性**：强类型检查、异常处理
- **安全性**：沙箱机制、代码签名

### 1.2 Java版本演进

| 版本 | 发布时间 | 重要特性 |
|------|----------|----------|
| JDK 1.0 | 1996 | 初始版本 |
| JDK 1.5 | 2004 | 泛型、注解、枚举、自动装箱 |
| JDK 1.8 | 2014 | Lambda表达式、Stream API、接口默认方法 |
| JDK 11 | 2018 | LTS版本、模块化、HttpClient |
| JDK 17 | 2021 | LTS版本、密封类、模式匹配 |

## 二、核心模块

本目录包含以下核心技术模块：

### 2.1 JVM（Java虚拟机）
深入理解JVM的运行时数据区、垃圾收集、类加载机制等核心概念。

- [JVM概述](JVM/README.md)
- [自动内存管理机制](JVM/深入理解JVM-jdk1.7/自动内存管理机制.md)
- [垃圾收集器与内存分配策略](JVM/深入理解JVM-jdk1.7/垃圾收集器与内存分配策略.md)
- [JVM调优](JVM/深入理解JVM-jdk1.7/JVM调优-jdk1.8.md)
- [HotSpot虚拟机对象探秘](JVM/深入理解JVM-jdk1.7/HotSpot虚拟机对象探秘.md)
- [OutOfMemoryError异常](JVM/深入理解JVM-jdk1.7/OutOfMemoryError异常.md)

### 2.2 多线程与并发
掌握Java并发编程的核心概念和工具类。

- [并发理论基础](多线程/一部分并发理论基础.md)
- [并发工具类详解](多线程/二部分并发工具类详解.md)
- [AQS](多线程/AQS.md)
- [CAS自旋锁](多线程/CAS自旋锁.md)
- [Java锁](多线程/Java锁.md)
- [Synchronized锁详解](多线程/Synchronized锁详解.md)
- [ThreadLocal详解](多线程/ThreadLocal详解.md)
- [JMM内存模型与Volatile](多线程/JMM内存模型Volatile关键字.md)
- [多线程面试题总结](多线程/多线程面试题总结.md)

### 2.3 集合框架
掌握Java集合框架的设计与实现。

- [集合框架概述](集合/README.md)
- [ArrayList详解](集合/Collection/ArrayList详解.md)
- [HashMap详解](集合/Map/Map与HashMap详解.md)
- [ConcurrentHashMap详解](集合/Map/ConcurrentHashMap详解.md)
- [CopyOnWriteArrayList详解](集合/Collection/CopyOnWriteArrayList详解.md)
- [HashSet详解](集合/Collection/HashSet详解.md)
- [Fail-Fast机制](集合/Fail-Fast.md)

### 2.4 设计模式
学习经典的设计模式及其应用场景。

- [设计模式概述](设计模式/README.md)
- [单例模式](设计模式/创建型模型/单例模式.md)
- [简单工厂模式](设计模式/创建型模型/简单工厂模式.md)
- [策略模式](设计模式/行为模型/策略模式.md)
- [模板方法模式](设计模式/行为模型/模板方法设计模式.md)
- [命令模式](设计模式/行为模型/命令模式.md)
- [责任链模式](设计模式/行为模型/责任链模式.md)

### 2.5 网络编程
理解网络通信的基本原理。

- [计算机网络基础](网络/计算机网络.md)
- [TCP协议](网络/TCP协议.md)
- [HTTP常见面试题](网络/http/HTTP常见面试.md)
- [HTTP常见错误码](网络/http/HTTP常见错误码.md)

### 2.6 IO模型
掌握Java的IO操作和NIO编程。

- [IO模型概述](IO模型/README.md)

### 2.7 数据处理
学习JSON等数据处理技术。

- [JSON处理](数据处理/json.md)

## 三、学习建议

### 3.1 学习路径
1. **基础语法**：掌握Java基本语法、面向对象编程
2. **核心API**：深入学习集合框架、并发包
3. **JVM原理**：理解内存模型、垃圾回收、类加载
4. **设计模式**：学习并实践常用设计模式
5. **实战项目**：通过项目巩固所学知识

### 3.2 关键技能
- 熟练使用Java 8+特性（Lambda、Stream API）
- 理解并发编程原理和实践
- 掌握JVM调优技巧
- 具备良好的代码设计能力

---
