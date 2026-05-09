---
name: "framework-logging-mdc"
description: "提供Slf4j MDC结构化日志规范，用于请求链路追踪。"
---

# MDC 结构化日志规范

## 使用时机
- 需要追踪请求链路时
- 需要在日志中添加上下文信息时
- 需要输出JSON格式日志时

## 一、MDC（Mapped Diagnostic Context）

### 基本使用

```java
// 在过滤器或拦截器中设置
MDC.put("requestId", UUID.randomUUID().toString());
MDC.put("userId", String.valueOf(userId));
MDC.put("traceId", traceId);

try {
    // 业务逻辑
    log.info("[Service.doSomething] 处理请求");
} finally {
    // 清理MDC
    MDC.clear();
}
```

### Logback配置MDC输出

```xml
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - traceId=%X{traceId} userId=%X{userId} - %msg%n</pattern>
```

## 二、JSON结构化日志

### 添加依赖

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.2</version>
</dependency>
```

### 配置JSON输出

```xml
<appender name="JSON_FILE" class="ch.qos.logback.core.FileAppender">
    <file>logs/app.json</file>
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <customFields>{"appName":"my-app","environment":"prod"}</customFields>
    </encoder>
</appender>
```

## 三、完整配置示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - traceId=%X{traceId} userId=%X{userId} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

## 四、过滤器示例

```java
@Component
public class MdcFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        try {
            MDC.put("requestId", UUID.randomUUID().toString());
            MDC.put("ip", getClientIp(request));
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

## 最佳实践

1. **始终清理MDC**：在finally块中调用MDC.clear()
2. **统一key命名**：定义常量避免拼写错误
3. **敏感信息不放入MDC**：避免泄露用户敏感数据

---

**参考文档**：
- [基础日志规范](../basic/skill.md)
- [敏感信息脱敏](../sensitive/skill.md)
- [日志配置最佳实践](../config/skill.md)
- [日志排查技巧](../troubleshooting/skill.md)
