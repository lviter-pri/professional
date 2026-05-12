---
name: "framework-logging-config"
description: "提供Logback日志配置最佳实践，包括异步日志和文件滚动配置。"
---

# Logback 日志配置最佳实践

## 使用时机
- 需要配置生产环境日志时
- 需要优化日志性能时
- 需要配置日志文件滚动策略时

## 一、生产环境配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 文件输出 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 异步输出 -->
    <appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
        <queueSize>1024</queueSize>
        <appender-ref ref="FILE"/>
    </appender>

    <!-- 根日志级别 -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ASYNC_FILE"/>
    </root>

    <!-- 第三方库日志级别 -->
    <logger name="com.zaxxer.hikari" level="WARN"/>
    <logger name="org.springframework" level="WARN"/>
</configuration>
```

## 二、异步日志配置

```xml
<appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
    <queueSize>1024</queueSize>           <!-- 队列大小 -->
    <discardingThreshold>0</discardingThreshold>  <!-- 不丢弃日志 -->
    <includeCallerData>false</includeCallerData>  <!-- 不包含调用者信息 -->
    <appender-ref ref="FILE"/>
</appender>
```

## 三、文件滚动配置

```xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logs/app.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>logs/app.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <maxFileSize>100MB</maxFileSize>      <!-- 单个文件大小 -->
        <maxHistory>30</maxHistory>         <!-- 保留30天 -->
        <totalSizeCap>1GB</totalSizeCap>       <!-- 总大小限制 -->
        <cleanHistoryOnStart>true</cleanHistoryOnStart>
    </rollingPolicy>
</appender>
```

## 四、按环境配置

```xml
<springProfile name="dev">
    <root level="DEBUG">
        <appender-ref ref="CONSOLE"/>
    </root>
</springProfile>

<springProfile name="prod">
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ASYNC_FILE"/>
    </root>
</springProfile>
```

## 五、日志级别设置建议

| 环境 | 根级别 | 应用包级别 | 第三方库级别 |
|------|--------|-----------|-------------|
| **开发** | DEBUG | DEBUG | WARN |
| **测试** | INFO | DEBUG | WARN |
| **生产** | INFO | INFO | WARN |

## 六、性能优化要点

1. **使用异步日志**：减少主线程阻塞
2. **合理设置队列大小**：根据日志量调整
3. **限制日志文件大小**：避免磁盘空间耗尽
4. **定期清理日志**：设置maxHistory

---

**参考文档**：
- [基础日志规范](../basic/skill.md)
- [MDC结构化日志](../mdc/skill.md)
- [敏感信息脱敏](../sensitive/skill.md)
- [日志排查技巧](../troubleshooting/skill.md)
