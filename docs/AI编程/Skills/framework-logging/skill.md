---
name: "framework-logging"
description: "提供Slf4j日志最佳实践和规范。当用户需要日志格式、级别选择、放置位置或性能优化指导时调用。"
---

# Slf4j 日志规范总览

本文档为日志相关技能的入口页面，包含以下子文档：

## 日志规范目录

### 1. 基础日志规范
[![基础日志规范](https://img.shields.io/badge/文档-基础规范-blue)](basic/skill.md)

- 日志格式规范
- 日志级别选择
- 异常日志处理
- 性能优化建议

### 2. MDC结构化日志
[![MDC结构化日志](https://img.shields.io/badge/文档-MDC结构化-orange)](mdc/skill.md)

- MDC上下文传递
- 请求链路追踪
- JSON结构化输出
- Filter集成示例

### 3. 敏感信息脱敏
[![敏感信息脱敏](https://img.shields.io/badge/文档-脱敏规范-green)](sensitive/skill.md)

- 敏感信息识别
- 脱敏工具类
- 日志审计检查
- 合规要求

### 4. 日志配置最佳实践
[![日志配置](https://img.shields.io/badge/文档-配置指南-purple)](config/skill.md)

- 生产环境配置
- 异步日志配置
- 文件滚动策略
- 按环境配置

### 5. 日志排查技巧
[![排查技巧](https://img.shields.io/badge/文档-排查指南-red)](troubleshooting/skill.md)

- 问题定位方法
- 常用命令
- 日志分析工具
- 排查流程

## 使用流程

```
1. 学习基础规范 → 掌握日志格式和级别选择
2. 配置MDC → 实现请求链路追踪
3. 添加脱敏处理 → 确保数据安全
4. 配置生产环境 → 优化性能
5. 使用排查技巧 → 快速定位问题
```

## 快速参考

| 场景 | 推荐文档 |
|------|---------|
| 添加日志 | [基础日志规范](basic/skill.md) |
| 链路追踪 | [MDC结构化日志](mdc/skill.md) |
| 数据安全 | [敏感信息脱敏](sensitive/skill.md) |
| 环境配置 | [日志配置最佳实践](config/skill.md) |
| 问题排查 | [日志排查技巧](troubleshooting/skill.md) |
