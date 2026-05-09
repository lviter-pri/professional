---
name: "framework-logging-basic"
description: "提供Slf4j日志基础规范，包括日志格式、级别选择和异常处理。"
---

# Slf4j 基础日志规范

## 使用时机
- 需要在代码中添加日志时
- 不确定日志级别选择时
- 需要正确记录异常日志时

## 快速开始

### 添加 @Slf4j 注解

```java
@Slf4j
@Service
public class XxxService {
    public void doSomething() {
        log.info("处理开始");
    }
}
```

### 手动声明 Logger

```java
public class XxxService {
    private static final Logger log = LoggerFactory.getLogger(XxxService.class);
}
```

## 一、日志格式规范

### 基本格式
推荐格式：`[类名.方法名] 操作描述: 关键参数`

```java
log.info("[AuthService.loginUser] 用户登录: userId={}", userId);
log.error("[KafkaConsumer.processMessage] 消息处理失败: topic={}, offset={}", topic, offset);
```

### 占位符使用
使用占位符避免字符串拼接：

```java
// 正确 ✅
log.info("处理订单: orderId={}, status={}", orderId, status);

// 错误 ❌
log.info("处理订单: orderId=" + orderId + ", status=" + status);
```

### 异常日志
异常作为最后一个参数：

```java
try {
    processOrder(order);
} catch (Exception e) {
    log.error("[OrderService.processOrder] 订单处理失败: orderId={}", orderId, e);
}
```

## 二、日志级别规范

| 级别 | 使用场景 | 生产环境建议 |
|------|---------|-------------|
| **TRACE** | 方法进入/退出、详细调试 | 关闭 |
| **DEBUG** | SQL参数、中间结果 | 关闭 |
| **INFO** | 关键业务流程、状态变更 | 开启 |
| **WARN** | 缓存未命中、配置默认值 | 开启 |
| **ERROR** | 数据库连接失败、业务异常 | 开启 |

### 使用禁忌

| 级别 | 禁止行为 |
|------|---------|
| ERROR | 记录可恢复的异常（应使用WARN） |
| DEBUG | 生产环境开启 |
| INFO | 循环内频繁记录 |

## 三、性能优化

### 条件日志判断
```java
if (log.isDebugEnabled()) {
    log.debug("复杂对象详情: {}", buildComplexDebugInfo());
}
```

### 批量操作日志
```java
log.info("[BatchProcessor] 开始处理: count={}", orders.size());
// 处理逻辑
log.info("[BatchProcessor] 处理完成: total={}, success={}, failed={}", total, success, failed);
```

## 最佳实践总结

1. **日志级别**：INFO用于生产，DEBUG用于开发
2. **日志内容**：包含定位信息、操作描述、关键参数
3. **异常处理**：异常作为最后一个参数，保留完整堆栈
4. **性能优化**：条件判断避免不必要计算

---

**参考文档**：
- [MDC结构化日志](../mdc/skill.md)
- [敏感信息脱敏](../sensitive/skill.md)
- [日志配置最佳实践](../config/skill.md)
- [日志排查技巧](../troubleshooting/skill.md)
