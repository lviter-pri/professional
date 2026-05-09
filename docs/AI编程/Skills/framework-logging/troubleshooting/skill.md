---
name: "framework-logging-troubleshooting"
description: "提供日志排查和分析技巧，帮助快速定位问题。"
---

# 日志排查技巧

## 使用时机
- 需要定位生产环境问题时
- 需要分析日志文件时
- 需要排查日志相关问题时

## 一、快速定位问题

### 常用命令

```bash
# 查找包含错误的日志
grep -E "ERROR|Exception" app.log

# 按时间范围过滤
grep "2024-01-15 10:30" app.log

# 按请求ID追踪
grep "traceId=abc123" app.log

# 统计错误类型
grep "ERROR" app.log | awk '{print $NF}' | sort | uniq -c

# 查看最近100行日志
tail -100f app.log

# 查找关键字并显示上下文
grep -A 5 -B 5 "ERROR" app.log
```

## 二、常见问题排查

| 问题现象 | 排查方向 |
|---------|---------|
| **日志不输出** | 检查日志级别配置、appender是否正确引用 |
| **日志重复** | 检查是否有重复的appender引用 |
| **性能问题** | 检查是否开启了DEBUG/TRACE、是否使用异步日志 |
| **敏感信息泄露** | 检查日志内容，添加脱敏处理 |
| **日志文件过大** | 配置滚动策略和保留时间 |

## 三、日志分析工具

| 工具 | 用途 |
|------|------|
| **grep/awk/sed** | 命令行日志过滤和统计 |
| **ELK Stack** | 日志收集、存储、分析 |
| **Grafana Loki** | 轻量级日志聚合 |
| **Splunk** | 企业级日志分析 |
| **VisualVM** | JVM日志分析 |

## 四、日志分析流程

```
1. 定位问题时间范围
   grep "2024-01-15 10:30" app.log

2. 查找错误信息
   grep "ERROR" app.log

3. 根据traceId追踪完整链路
   grep "traceId=abc123" app.log

4. 分析异常堆栈
   grep -A 20 "Exception" app.log

5. 统计错误频率
   grep "ERROR" app.log | wc -l
```

## 五、日志输出示例

### 良好的日志示例

```
2024-01-15 10:30:15.123 [http-nio-8080-exec-1] INFO  c.e.s.AuthService - traceId=abc123 userId=12345 - [AuthService.loginUser] 用户登录成功
2024-01-15 10:30:15.456 [http-nio-8080-exec-1] WARN  c.e.s.CacheService - traceId=abc123 userId=12345 - [CacheService.get] 缓存未命中
2024-01-15 10:30:15.789 [http-nio-8080-exec-1] ERROR c.e.s.OrderService - traceId=abc123 userId=12345 - [OrderService.createOrder] 订单创建失败
java.lang.IllegalArgumentException: 库存不足
    at com.example.service.OrderService.createOrder(OrderService.java:45)
```

## 六、排查技巧总结

1. **时间定位**：先确定问题发生的时间范围
2. **级别过滤**：优先查看ERROR级别日志
3. **链路追踪**：使用traceId追踪完整请求链路
4. **堆栈分析**：查看异常堆栈定位代码位置
5. **统计分析**：统计错误频率判断问题严重程度

---

**参考文档**：
- [基础日志规范](../basic/skill.md)
- [MDC结构化日志](../mdc/skill.md)
- [敏感信息脱敏](../sensitive/skill.md)
- [日志配置最佳实践](../config/skill.md)
